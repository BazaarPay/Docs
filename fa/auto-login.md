# احراز هویت کاربر به واسطه‌ی پذیرنده

در این فرایند کاربر به جای احراز هویت (لاگین) در بازارپی، در سایت یا اپلیکیشن پذیرنده (merchant) احراز هویت شده
و سپس شماره‌ی همراه وی توسط پذیرنده به بازارپی ارسال می‌شود تا برای این شماره‌ی همراه،
یک توکن دسترسی (access token) ساخته شود و از آن برای دسترسی کاربر به سرویس‌های بازارپی
استفاده گردد. لطفاً توجه کنید که این توکن دسترسی باید به ازای هر فرایند پرداخت مجدداً ساخته شود.

در این حالت، کاربر نیازی به لاگین مجزا در بازارپی ندارد. sdkهای کلاینت
که از این قابلیت احراز هویت اتوماتیک پشتیبانی می‌کنند (sdk اندروید از [نسخه‌ی 5.2.0 به بعد](https://github.com/cafebazaar/BazaarPay/releases/tag/5.2.0))،
در صورتی که کاربر از قبل در بازارپی لاگین نباشد، از توکن دسترسی که
توسط مرچنت فراهم شده برای ادامه‌ی عملیات کاربر استفاده می‌کنند.

نمودار دنباله‌ی این فرایند را در زیر مشاهده می‌کنید:
```mermaid
sequenceDiagram
    participant کاربر
    participant مرچنت
    participant بازارپی
    participant web / android sdk

    مرچنت ->> بازارپی: ارسال شماره همراه کاربر
    بازارپی -->> مرچنت: دریافت توکن دسترسی
    Note over بازارپی, مرچنت: Create User Access Token

    مرچنت ->> بازارپی: درخواست ساخت توکن چک‌اوت
    بازارپی -->> مرچنت: ارسال توکن چک‌اوت (و آدرس پرداخت)
    Note over بازارپی, مرچنت: Initiate Checkout

    مرچنت ->> web / android sdk: انتقال توکن دسترسی و توکن چک‌اوت به اس‌دی‌کی و شروع فلوی پرداخت

    web / android sdk ->> کاربر: بازکردن پاپ‌آپ/انتقال کاربر به آدرس پرداخت
    کاربر ->> بازارپی: انجام عملیات موفق پرداخت
    بازارپی ->> مرچنت: انتقال کاربر به مرچنت

    مرچنت ->> کاربر: ارایه محصول به کاربر

    مرچنت ->> بازارپی: تایید عملیات خرید
    Note over مرچنت, بازارپی: Commit
```

## دریافت توکن دسترسی برای کاربر از سمت پذیرنده

پذیرنده بعد از احراز هویت کاربر در سایت خود، می‌تواند از طریق
اندپوینت `create-user-access-token`، برای شماره همراه کاربر توکن دسترسی دریافت نماید.
این توکن دسترسی فقط می‌تواند برای یکی از سه محدوده (scope) زیر مورد استفاده قرار گیرد:
1. خرید (purchase): برای خرید از طریق کیف پول، درگاه بانکی، و یا دایرکت‌دبیت
1. افزایش موجودی (increase_balance): برای فرآیند افزایش موجودی مستقیم کیف پول
1. پرداخت خودکار (direct_pay): برای فرآیند پرداخت خودکار

در صورتی که از توکن ایجادشده در محدوده‌ی دیگری استفاده کنید با خطای ۴۰۳ مواجه خواهید شد.

### نمونه

```yaml
openapi: 3.1.0
info:
  title: BazaarPay API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path}'
paths:
  /merchant/create-user-access-token/:
    post:
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                user_phone_number:
                  type: string
                  required: true
                  example: "09123456789 or 989123456789"
                  description: شماره همراه کاربر (مانند فرمت نمونه)
                scope:
                  required: true
                  type: string
                  enum:
                    - purchase
                    - increase_balance
                    - direct_pay
                  example: "purchase"
                  description: |
                    - purchase: دریافت توکن برای فرآیند خرید (خرید از طریق کیف پول، درگاه بانکی، و یا دایرکت‌دبیت)
                    - increase_balance: دریافت توکن برای فرآیند افزایش موجودی به‌طور مستقیم
                    - direct_pay: دریافت توکن برای فرآیند پرداخت خودکار (دایرکت‌پی)
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  access_token:
                    type: string
                    example: "A1023e71738b4c1045529003314f429f0.81f023z="
        '401':
          $ref: './fa/shared_components/error-responses.md#/responses/401'
        '403':
          description: Permission Denied
          content:
            application/json:
              schema:
              - $ref: '#/components/schemas/PermissionDenied'
        '400':
          description: Bad Request
          content:
            application/json:
              schema:
                oneOf:
                - $ref: './fa/shared_components/error-responses.md#/responses/400/content/application/json/schema'
                - $ref: '#/components/schemas/BadRequest'
        '503':
          $ref: './fa/shared_components/error-responses.md#/responses/503'
components:
  securitySchemes:
    ApiKeyAuth:
      $ref: './fa/shared_components/security.md#/securitySchemes/ApiKeyAuth'
```

### نمونه cURL

```curl
curl --request POST 'https://pardakht.cafebazaar.ir/pardakht/badje/v1/merchant/create-user-access-token/' \
--header 'Content-Type: application/json; charset=utf-8' \
--header 'Authorization: Token {merchant_token}' \
--data-raw '{
    "scope": "purchase",
    "user_phone_number": "09123456789"
}'
```

### نمونه موفق پاسخ درخواست

```json
{
	"access_token": "A1y.023e717-8b4c1045529003314f429f0.81f023z="
}
```

در صورتی که پذیرنده پاسخی با کد وضعیت 403 دریافت کند،
باید فرایند پرداخت را بدون توکن دسترسی اتوماتیک ادامه دهد.
این اتفاق ممکن است به خاطر غیرفعال‌شدن موقتی قابلیت احراز هویت اتوماتیک به دلایل فنی رخ بدهد.

## ارسال توکن دسترسی به sdk اندروید

برای دیدن نحوه‌ی احراز هویت اتوماتیک در sdk اندروید،
به [مستندات مربوط به sdk](https://github.com/cafebazaar/BazaarPay#2-launch-payment) مراجعه کنید.

## ارسال توکن دسترسی به sdk وب

مستندات مربوط به احراز هویت اتوماتیک در sdk وب در آینده نزدیک اعلام خواهد شد.

## دریافت شماره همراه کاربر از روی توکن دسترسی

کلاینت و sdkها می‌توانند برای به‌دست‌آوردن شماره‌ی همراه کاربر از روی توکن دسترسی
و نمایش شماره‌ی همراه در UI، از اندپوینت `user/info/` استفاده کنند.

### نمونه

```yaml
openapi: 3.1.0
info:
  title: BazaarPay API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path}'
paths:
  /user/info/:
    get:
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  phone_number:
                    type: string
                    example: "989123456789"
        '401':
          $ref: './fa/shared_components/error-responses.md#/responses/401'
        '400':
          $ref: './fa/shared_components/error-responses.md#/responses/400'
        '503':
          $ref: './fa/shared_components/error-responses.md#/responses/503'
components:
  securitySchemes:
    ApiKeyAuth:
      $ref: './fa/shared_components/security.md#/securitySchemes/ApiKeyAuth'
```

### نمونه cURL

```curl
curl 'https://pardakht.cafebazaar.ir/pardakht/badje/v1/user/info/' \
     --header 'Authorization: Bearer {user_access_token}' \
     --header 'Accept: application/json'
```

### نمونه موفق پاسخ درخواست

```json
{
    "phone_number": "989123456789"
}
```

## نمونه خطاها

```yaml
components:
  schemas:
    PermissionDenied:
      type: object
      properties:
        detail:
        - type: string
        examples:
          permission_error:
            value:
              description: وقتی پذیرنده دسترسی لازم برای ساختن توکن (برای خرید یا افزایش موجودی یا پرداخت خودکار) را ندارد.
              detail: "شما دسترسی لاگین اتوماتیک برای خرید را ندارید."
    BadRequest:
      type: object
      properties:
        oneOf:
          detail:
            - type: string
          scope:
            - type: list
              items:
                - type: string
          user_phone_number:
            - type: list
              items:
                - type: string
        examples:
          invalid_scope:
            value:
              description: مقدار scope ارسال‌شده معتبر نیست
              scope: ["این اسکوپ انتخاب معتبری نیست."]
          invalid_user_phone_number:
            value:
              description: شماره تماس معتبر نیست.
              user_phone_number: ["شماره تماس معتبر نیست"]
```
## اتولاگین در وب سرویس بازارپی

در حال حاضر سرویس اتولاگین در وب سرویس بازار‌پی قابل دسترس است. این سرویس با استفاده از کوئری پارام توکن احراز هویت کاربر را دریافت کرده و فرایند مورد نظر را با استفاده از آن توکن احراز هویت انجام می‌دهد.

### استفاده از web-sdk بازار‌پی
این روش در حال پیاده سازی می‌باشد و بزودی در دسترس قرار می‌گیرد.

### فراخوانی مستقیم وب سرویس
در هر اینتری پوینت از وب سرویس با پاس دادن کوئری پارامز `auto_login_token` وب سرویس روش احراز هویت را اتولاگین تشخیص داده و تا انتهای فرآیند مورد نظر کاربر با آن توکن احراز خواهد شد.

به طور مثال برای استفاده از سرویس پرداخت بازارپی با استفاده از اتولاگین می‌بایست به آدرس `cafebazaar.ir/bazaar-pay/payment‍` کاربر را ریدایرکت کرده تا فرآیند پرداخت آغاز گردد
در حالت عادی هر اینتری پوینت تعدادی کوئری پارامز دارد که در هر قسمت به تفصیل توضیح داده شدن، برای استفاده از سرویس اتولاگین دراین سرویس‌ها می‌بایست علاوه بر کوئری پارامز سرویس، کوئری `auro_login_token` را هم به سرویس پاس دهید و باقی فرآیند را وب سرویس انجام خواهد داد.

### نمونه پیاده‌سازی

#### سرویس پرداخت بدون اتولاگین
```
  https://cafebazaar.ir/bazaar-pay/payment?checkout_token={checkout_token}&redirect_url={merchant_redirect_url}
```
#### سرویس پرداخت با اتولاگین
```
   https://cafebazaar.ir/bazaar-pay/payment?checkout_token={checkout_token}&redirect_url={merchant_redirect_url}&auto_login_token={auto_login_v2_token}
```

## فورس لاگین در وب سرویس بازارپی
در فرآیند‌های به‌خصوصی این دسترسی به پذیرنده داده می‌شود تا فرآیند مورد انتظار کاربر را محدود به یک اکانت(شماره‌ی همراه) کتد. برای این کار می‌بایست از سرویس فورس‌لاگین استفاده شود. این قابلیت برای سرویس های دایرکت‌پی (تخصیص سهمیه) و افزایش موجودی قابل دسترس می‌باشد.

### استفاده از web-sdk بازار‌پی
این روش در حال پیاده سازی می‌باشد و بزودی در دسترس قرار می‌گیرد.

###  فورس‌لاگین در ورود مستقیم به وب سرویس 
برای استفاده ازین سرویس در فرآیند مورد نظر کاربر را به همراه دو کوئری پارامز به وب‌سرویس هدایت می‌کنید. این دو پارامز مشخص‌کننده درخواست فورس‌لاگین و ارسال شماره‌ی همراه کاربر برای احراز در فلو است. این سرویس بر بستر لاگین بازارپی و همچنین اکسس توکن اتولاگین پیاده شده است.

کوئری پارام `force_login` که نشان دهنده‌ی درخواست فورس لاگین است می‌بایست با مقدار `true` و کوئری پارام `phone` با شماره‌ی مورد نظر برای احراز ارسال گردند.

در صورتی که سرویس اتولاگین را پیاده‌سازی نموده‌اید به همراه این کوئری پارامز، مطابق با توضیحات فوق در بخش اتولاگین، کوئری پارام `auto_login_token` را با اکسس توکن شماره‌ی همراه مورد نظر، به سرویس پاس دهید.

وب سرویس بازارپی با استفاده ازین پارامز درخواست پذیرنده را تشخیص داده و فرآیند را به صورت فورس لاگین آغاز می‌کند.
 ### Query Params
 ```yaml
  queryParams:
    - force_login:
        key: force_login
        value: true
        description: force login request flag 
        required
    - phone:
        key: phone
        sample-value: 09123456789
        description: user's forced phone number
        required
 ```


### نمونه پیاده‌سازی

#### (لاگین بازارپی) فراخوانی سرویس پرداخت با فورس لاگین 
```
  https://cafebazaar.ir/bazaar-pay/increase-balance?redirect_url={merchant_redirect_url}&force_login=true&phone=989123456789
```
#### سرویس پرداخت با فورس لاگین (اتولاگین) 
```
   https://cafebazaar.ir/bazaar-pay/increase-balance?redirect_url={merchant_redirect_url}&auto_login_token={auto_login_v2_token}&force_login=true&phone=989123456789
```
