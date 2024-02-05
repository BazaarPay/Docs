# دایرکت‌پی

این فرآیند برای پرداخت مستقیم از سمت پذیرنده (merchant) بدون دخالت کاربر در مواردی همچون تمدید خودکار اشتراک استفاده
می‌شود.

## ایجاد توکن قرارداد دایرکت‌پی

برای ساخت قرارداد دایرکت پی پذیرنده (merchant) باید محدودیت حداکثر میزان تراکنش و دوره‌ی زمانی این محدودیت را مشخص کند.

به طور مثل اگر پذیرنده، قراردادی با محدودیت ۱ میلیون تومان به صورت هفتگی برای کاربر ایجاد کند، بعد از امضای این قرارداد
توسط کاربر پذیرنده حداکثر هفته‌ای ۱ میلیون تومان می‌تواند پرداخت مستقیم را از حساب کاربر انجام دهد.

### نمونه

```yaml
openapi: 3.1.0
info:
  title: Initialize Directpay Contract API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
paths:
  /direct-pay/contract/init/:
    post:
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                type:
                  required: true
                  type: string
                  enum:
                    - direct_debit
                    - wallet
                  example: "wallet"
                  description: |
                    - direct_debit: دایرکت دبیت (پرداخت مستقیم)
                    - wallet: کیف پول
                amount_limit:
                  required: true
                  type: integer
                  minimum: 100000
                  maximum: 1000000000
                  example: 1000000
                period:
                  required: true
                  type: string
                  enum:
                    - daily
                    - weekly
                    - monthly
                    - quarterly
                    - semiannual
                    - yearly
                  example: "yearly"
                  description: |
                    دوره‌ی زمانی از ابتدا هفته، ماه یا سال شروع می‌شود. به این معنی که اگر قرارداد کاربر ۱۳ خرداد ماه امضا شود، دوره‌ی زمانی آن تا یکم تیر ماه است و از یکم تیر ماه محدودیت میزان تراکنش ریست می‌شود، مقادیر مجاز شامل:
                      - daily: روزانه، در صورتی که قرارداد اعتبار داشته باشد، محدودیت میزان تراکنش روزانه ریست می‌شود
                      - weekly: هفتگی، در صورتی که قرارداد اعتبار داشته باشد، محدودیت میزان تراکنش از اول هفته (شنبه) ریست می‌شود
                      - monthly: ماهانه، در صورتی که قرارداد اعتبار داشته باشد، محدودیت میزان تراکنش از اول ماه ریست می‌شود
                      - quarterly: سه‌ماهه، در صورتی که قرارداد اعتبار داشته باشد، محدودیت میزان تراکنش از اول سه‌ماه ریست می‌شود
                      - semiannual: شش‌ماهه، در صورتی که قرارداد اعتبار داشته باشد، محدودیت میزان تراکنش از اول شش‌ماه ریست می‌شود
                      - yearly: سالانه، در صورتی که قرارداد اعتبار داشته باشد، محدودیت میزان تراکنش از اول سال ریست می‌شود
                changeable_limits:
                  required: false
                  type: bool
                  default: false
                  description: |
                    مقادیر amount_limit و period در زمان تایید قرارداد توسط کاربر قابل تغییر باشند.
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  contract_token:
                    type: string
                    example: '7f9bf78c-a5e2-4126-9482-37484b3706be'
        '401':
          $ref: './fa/shared-components/error-responses.md#/responses/401'
        '403':
          $ref: './fa/shared-components/error-responses.md#/responses/403'
        '400':
          $ref: './fa/shared-components/error-responses.md#/responses/400'
        '503':
          $ref: './fa/shared-components/error-responses.md#/responses/503'
components:
  securitySchemes:
    ApiKeyAuth:
      $ref: './fa/shared-components/security.md#/securitySchemes/ApiKeyAuth'
```

در صورت استفاده از changeable_limits،‌ جهت تعیین/تغییر مقادیر قابل انتخاب توسط کاربر برای amount_limit و period به
تیم‌بازارپی اطلاع دهید.

### نمونه cURL

```curl
curl --location --request POST 'https://api.bazaar-pay.ir/badje/v1/direct-pay/contract/init/' \
--header 'Authorization: Token {merchant_token}' \
--header 'Content-Type: application/json' \
--data-raw '{
    "type": "wallet",
    "period": "monthly",
    "amount_limit": 1000000
}'
```

### نمونه موفق پاسخ درخواست

```json
{
	"contract_token": "9bb790a3-44fd-486f-8ce8-38aa02cab069"
}
```

## فعال‌سازی/رد قرارداد دایرکت‌پی بدون Web SDK

برای امضای قرارداد توسط کاربر، می‌بایست کاربر را به صفحه‌ی امضای قرارداد ریدایرکت کنید. برای این کار توکن قرارداد
دریافتی از `init contract` را به همراه `redirect_url` به این آدرس پاس می‌دهید و کاربر پس از امضای قرارداد به کلاینت
مرچنت
ریدایرکت می‌شود.
در فرآیند بستن قرارداد دایرکت‌پی، در صورتی که نوع قرارداد از جنس دایرکت‌دبیت باشد و کاربر قرارداد فعال دایرکت‌دبیت
نداشته باشد، به صورت خودکار وارد فرآیند بستن قرارداد دایرکت‌دبیت هم می‌شود.

### نمونه

```yaml
openapi: 3.1.0
info:
  title: Open Directpay Webpage
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path}'
    description: BazaarPay Web
paths:
  /contract/direct-pay:
    summary: finalize-contract-without-sdk
    description: بعد از تایید/رد قرارداد، کاربر به آدرس بازگشت ارسال شده توسط مرچنت منتقل می‌شود
    parameters:
      - name: contract_token
        in: query
        required: true
        schema:
          type: string
          example: 9bb790a3-44fd-486f-8ce8-38aa02cab069
        description: توکن قرارداد گرفته شده از اندپوینت Init Contract
      - name: redirect_url
        in: query
        required: true
        description: پس از عملیات فعال‌سازی، کاربر به این آدرس بازگشت داده می‌شود
        schema:
          type: string
          example: https://example.com/bazaar-pay-return/direct-pay-contract
      - name: phone
        in: query
        required: false
        schema:
          type: string
          example: 09999999999
        description: در صورت نیاز به لاگین،‌شماره کاربر توسط این پارامتر در صفحه آن پر می‌شود. در صورتی که از قبل لاگین باشد از همان یوزر برای ایجاد قرارداد استفاده می‌شود.
      - name: message
        in: query
        required: false
        schema:
          type: string
          example: این یک پیام تست است
        description: مرچنت توسط این فیلد می‌تواند یک پیام اختصاصی به کاربر نمایش دهد.
```

### نمونه آدرس

```
https://cafebazaar.ir/bazaar-pay/contract/direct-pay?contract_token={contract_token}&redirect_url={encoded_url}&phone={user_phone_number}&message={encoded_message}
```

### فعال‌سازی/رد قرارداد دایرکت‌پی توسط Web SDK

برای آغاز فلوی نهایی‌سازی قرارداد می‌بایست از sdk وب بازارپی استفاده نمایید.

```typescript
import {startFinalizeContractProccess} from 'bazaar-payment-sdk';

startFinalizeContractProccess(contractToken, callBackUrl, phoneNumber)
    .then(() => {
        // The user will be redirected to the provided callBackUrl.
    });
```

* با استفاده از تابع `startFinalizeContractProccess` پاپ‌آپ نهایی‌سازی قرارداد دایرکت‌پی باز می‌شود و پس از اتمام فرآیند
  نهایی‌سازی، کلاینت به آدرس بازگشت (`callBackUrl`) ریدایرکت می‌شود.
* پس از ریدایرکت کاربر می‌توانید با استفاده از اندپوینت `Trace Contract` از وضعیت نهایی قرارداد مطلع شوید.
* امکان رد/تایید قراردادی که از قبل رد/تایید شده باشد مجددا وجود ندارد و در صورتی که کاربر اقدام به این عمل کند ارور
  مناسب
  آن مانند `این قرارداد قبلا فعال شده‌است و امکان تغییر وضعیت وجود ندارد.` نمایش داده می‌شود.

#### startFinalizeContractProcess Arguments:

```yaml
arguments:
  - name: contractToken
    type: string
  - name: callBackUrl
    type: CallbackUrl
  - name: phoneNumber
    type: string
```

##### CallbackUrl Type

```yaml
CallbackUrl:
  type: object
  properties:
    url:
      type: string
    method:
      type: string
      enum: [ post, get ]
    data:
      type: object
      additionalProperties:
        type: string
  required:
    - method
    - data
```

##### CallbackUrl Interface

```typescript
interface CallbackUrl {
    url?: string;
    method: "post" | "get";
    data: {
        [propName: string]: string;
    }
}
```

## پیگیری قرارداد دایرکت‌پی

توسط این اندپوینت می‌توانید از وضعیت قرارداد مطلع شوید.

### نمونه

```yaml
openapi: 3.1.0
info:
  title: Trace Directpay Contract API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
paths:
  /direct-pay/contract/trace/:
    get:
      summary: trace-contract
      parameters:
        - name: contract_token
          in: query
          required: true
          schema:
            type: string
            example: af72319b-9baf-4c2b-9dbf-76cf119a4582
          description: توکن قرارداد گرفته شده از اندپوینت Init Contract
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  state:
                    type: string
                    enum:
                      - new
                      - active
                      - declined
                      - cancelled
                      - expired
                    example: new
                    description: |
                      وضعیت قرار داد کاربر، مقدیر مجاز برابر است با:
                      - new: قرارداد به تازگی ساخته شده است. ممکن است کاربر هنوز در پروسه‌ی دادن رضایت باشد
                      - active: قراداد فعال است و می‌توان پرداخت‌مستقیم انجام داد
                      - declined: قراداد توسط کاربر رد شده است
                      - cancelled: مرچنت قرارداد کاربر را لغو کرده است
                      - expired: قرارداد منقضی شده است
                  expiration_time:
                    type: string
                    example: 2024-06-25T09:28:34.668933Z
                    format: iso-8601
                    description: تاریخ و زمان منقضی شدن قرارداد
                  amount_limit:
                    type: integer
                    example: 100000
                    description: ماکزیمم مبلغ در هر دوره
                  limit_remaining_amount:
                    type: integer
                    example: 98730
                    description: مبلغ باقی‌مانده مجاز در دوره جاری
                  limit_expiration_time:
                    type: string
                    format: iso-8601
                    example: 2024-06-25T09:28:34.668933Z
                    description: تاریخ و زمان پایان دوره فعلی (زمان ریست شدن محدودیت میزان تراکنش)
        '401':
          $ref: './fa/shared-components/error-responses.md#/responses/401'
        '403':
          $ref: './fa/shared-components/error-responses.md#/responses/403'
        '400':
          $ref: './fa/shared-components/error-responses.md#/responses/400'
        '503':
          $ref: './fa/shared-components/error-responses.md#/responses/503'
components:
  securitySchemes:
    ApiKeyAuth:
      $ref: './fa/shared-components/security.md#/securitySchemes/ApiKeyAuth'
```

### نمونه cURL

```curl
curl --location 'https://api.bazaar-pay.ir/badje/v1/direct-pay/contract/trace/?contract_token=af72319b-9bae-4c2b-9cbf-76cs119a4582' \
--header 'Authorization: Token {merchant_token}' 
```

### نمونه موفق پاسخ درخواست

```json
{
	"state": "declined",
	"expiration_time": "2024-07-02T06:43:44.843212Z",
	"amount_limit": 100000,
	"limit_remaining_amount": 100000,
	"limit_expiration_time": "2025-07-02T06:43:44.843212Z"
}
```

## لغو قرارداد دایرکت‌پی

توسط این اندپوینت مرچنت می‌تواند قراردادهای فعال کاربر را لغو کند.
درصورت مواجه با خطاهای دسته‌ي ۴۰۰ می‌توانید با فراخوانی اندپوینت trace از وضعیت فعلی قرارداد مطلع شوید.

### نمونه

```yaml
openapi: 3.1.0
info:
  title: Cancel Directpay Contract API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
paths:
  /direct-pay/contract/cancel/:
    post:
      summary: cancel-contract
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                contract_token:
                  type: string
                  required: true
                  example: 9bb790a3-44fd-486f-8ce8-38aa0scab069
                  description: توکن قرارداد گرفته شده از اندپوینت Init Contract
      responses:
        '204':
          description: Success
        '401':
          $ref: './fa/shared-components/error-responses.md#/responses/401'
        '403':
          $ref: './fa/shared-components/error-responses.md#/responses/403'
        '400':
          description: Bad Request
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: './fa/shared-components/error-responses.md#/responses/400/content/application/json/schema'
                  - $ref: '#/components/schemas/CancelResponse'
        '503':
          $ref: './fa/shared-components/error-responses.md#/responses/503'
components:
  securitySchemes:
    ApiKeyAuth:
      $ref: './fa/shared-components/security.md#/securitySchemes/ApiKeyAuth'
```

### نمونه cURL

```curl
curl --location 'https://api.bazaar-pay.ir/badje/v1/direct-pay/contract/cancel/?contract_token=af72319b-9bae-4c2b-9cbf-76cs119a4582' \
--header 'Authorization: Token {merchant_token}' 
```

## پرداخت با دایرکت‌پی

نیاز است که قبل از فراخوانی این اندپوینت، قرارداد کاربر تایید (فعال) شده باشد. قبل از صدا زدن این اندپوینت باید توسط
اندپوینت init checkout یک توکن چک‌اوت ساخته شده باشد تا بتوان عملیات پرداخت را با آن انجام داد. می‌توان از این توکن در
اندپوینت‌های Trace و Refund نیز
استفاده کرد.

شما می‌توانید از ورژن دوم پرداخت نیز استفاده کنید، در این ورژن ساختار نمایش خطا در رسپانس این ریکوئست بهبود یافته تا کار
را برای استفاده مرچنت‌ها راحت‌تر نماید. علاوه بر آن مرچنت‌ها می‌توانند توسط مقداری که در جیسن آبجکت `code` که در رسپانس این
ریکوئست و در صورت به خطا خوردن برگرداننده می‌شود، لاجیک مورد نظر خود را نوشته و از آن استفاده نمایند.

### نمونه درخواست

```yaml
openapi: 3.1.0
info:
  title: Pay With Directpay Contract API
  version: 1.0.0 and 2.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
  - url: 'https://{base_url}{base_path_v2}'
    description: BazaarPay API v2
paths:
  /direct-pay/:
    post:
      summary: pay-with-contract
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                contract_token:
                  type: string
                  required: true
                  example: 9bb790a3-44fd-486f-8ce8-38aa0scab069
                  description: توکن قرارداد گرفته شده از اندپوینت Init Contract
                checkout_token:
                  type: string
                  required: true
                  example: "0123456789"
                  description: توکن چک‌اوت گرفته شده از اندپوینت init checkout
      responses:
        '204':
          description: Success
        '401':
          $ref: './fa/shared-components/error-responses.md#/responses/401'
        '403':
          $ref: './fa/shared-components/error-responses.md#/responses/403'
        '400':
          description: Bad Request
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: './fa/shared-components/error-responses.md#/responses/400/content/application/json/schema'
                  - $ref: '#/components/schemas/PayResponse'
                  - $ref: '#/components/schemas/PayResponseV2'
                    description: "فقط در نسخه v2"
        '500':
          description: Internal Server Error
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: './fa/shared-components/error-responses.md#/responses/500/content/application/json/schema'
                  - $ref: '#/components/schemas/PayResponseV2'
                    description: "فقط در نسخه v2"
        '503':
          $ref: './fa/shared-components/error-responses.md#/responses/503'
components:
  securitySchemes:
    ApiKeyAuth:
      $ref: './fa/shared-components/security.md#/securitySchemes/ApiKeyAuth'
```

### نمونه cURL

```curl
curl --location 'https://api.bazaar-pay.ir/badje/v1/direct-pay/' \
--header 'Authorization: Token {merchant_token}' \
--data '{
    "contract_token": "7f9bf78c-a5e2-4126-9482-3s484b3706be",
    "checkout_token": "1443280167"
}'
```

* در صورت موفقیت، یک رسپانس خالی را برمی‌گرداند.
* نسبت به توکن چک‌اوت، idempotent است.
* تفاوت این روش با پرداخت عادی که از طریق SDK توسط کاربر انجام می‌شود این است که نیازی به فراخوانی اندپوینت commit نیست
  و
  اتوماتیک کامیت (تایید خرید) انجام می‌شود. بنابراین بهتر است این اندپوینت بعد از ارائه‌ی محصول به کاربر و در یک تراکنش
  اتمیک فراخوانی گردد.

## نمونه خطاها

```yaml
components:
  schemas:
    PayResponse:
      type: object
      properties:
        detail:
          oneOf:
            - type: array
              items:
                type: string
            - type: string
        examples:
          direct_pay_is_not_active:
            summary: قرارداد دایرک‌ت‌پی کاربر به هر علتی فعال نشده باشد
            value:
              detail: [ "قرارداد کاربر فعال نیست." ]
          contract_pay_over_max:
            summary: حداکثر سقف مجاز در بازه‌ی ثبت شده برای یک قرارداد (مانند یک قرارداد هفتگی با سقف ۱۰ هزارتومان)، رعایت نشده باشد
            value:
              detail: "با توجه به قرارداد شما، در بازه هفتگی نمی‌توانید از ویژگی پرداخت مستقیم بیش از ۱۰,۰۰۰ تومان استفاده کنید."
          insufficient_wallet_balance:
            summary: موجودی کیف پول کاربر هنگام پرداخت با دایرکت‌پی کافی نباشد
            value:
              detail: "موجودی حساب کافی نیست."
          insufficient_direct_debit_balance:
            summary: موجودی کارت کاربر که دایرکت‌دبیت را با آن فعال کرده است هنگام پرداخت با دایرکت‌پی کافی نباشد
            value:
              detail: "موجودی کافی در حساب بانکی خود ندارید"
          direct_debit_less_than_allowed:
            summary: حداقل مقدار مجاز برای پرداخت با دایرکت دبیت رعایت نشود
            value:
              detail: "حداقل مقدار تراکنش ۱۰۰۰۰ ریال است."
          direct_debit_is_not_active:
            summary: امکان پرداخت با دایرکت‌دبیت وجود نداشته باشد، مانند زمانی که قرارداد کاربر منقضی، لغو یا در حال پردازش باشد
            value:
              detail: "شما سرویس پرداخت سریع خود را فعال نکرده‌اید."
          direct_debit_not_allowed:
            summary: امکان پرداخت با دایرکت‌دبیت وجود نداشته باشد، مانند زمانی که درگاه غیرفعال باشد یا به یک علت امکان استفاده از آن وجود نداشته باشد
            value:
              detail: "کاربر مجاز به انتخاب این درگاه نمی‌باشد."
          checkout_token_expired:
            summary: توکن پرداخت که از اندپوینت init checkout گرفته شده است، منقضی شده باشد
            value:
              detail: "توکن پرداخت منقضی شده است، لطفاً فرآیند پرداخت را از اول آغاز کنید."
          direct_debit_card_expired:
            summary: کارت کاربر به هر علتی (منقضی شده باشد یا توسط بانک) غیرفعال شده باشد
            value:
              detail: "کارت غیرفعال می‌باشد."
          direct_debit_service_down:
            summary: سرویس دایرکت‌دبیت به هر علتی غیرفعال شده باشد
            value:
              detail: "امکان برقراری ارتباط با بانک وجود ندارد، لطفا از سایر روش‌های پرداخت استفاده کنید یا دقایقی دیگر مجددا تلاش کنید."
          direct_debit_disabled_by_user:
            summary: قرارداد دایرکت‌دیبت توسط کاربر از سمت بانک لغو شده باشد
            value:
              detail: "شما قرارداد خود را از سمت بانک لغو کرده‌اید، لطفا قرارداد فعلی را لغو و مجددا قرارداد جدیدی ایجاد کنید."
          direct_debit_over_max_per_transaction:
            summary: اقدام به پرداخت بیشتر از مبلغ مجاز ثبت شده در قرارداد دایرکت‌دبیت شود (این مبلغ ممکن هست وابسته به قرارداد دایرکت‌دبیت متفاوت باشد)
            value:
              detail: "مبلغ تراکنش شما بیش از مقدار مجاز در سرویس پرداخت مستقیم است."
          direct_debit_over_max_amount_of_transaction_per_day:
            summary: اقدام به پرداخت بیشتر از مبلغ مجاز روزانه ثبت شده در قرارداد دایرکت‌دبیت شود (این مبلغ ممکن هست وابسته به قرارداد دایرکت‌دبیت متفاوت باشد)
            value:
              detail: "شما به سقف مقدار تراکنش روزانه پرداخت مستقیم رسیده‌اید."
          direct_debit_over_max_count_of_transactions_per_day:
            summary: اقدام به پرداخت بیشتر از تعداد مجاز روزانه ثبت شده در قرارداد دایرکت‌دبیت شود (این تعداد برابر با ۱۰ عدد تراکنش می‌باشد)
            value:
              detail: "شما به سقف تعداد تراکنش روزانه پرداخت مستقیم رسیده‌اید."
          user_account_disabled:
            summary: حساب کاربر غیرفعال باشد.
            value:
              detail: "حساب غیرفعال است."
    CancelResponse:
      type: object
      properties:
        oneOf:
          detail:
            - type: string
          contract_token:
            - type: list
              items:
                - type: string
        examples:
          invalid_contract_token:
            summary: توکن قرارداد ارسال شده توکن معتبری نیست
            value:
              contract_token: [ "توکن قرارداد صحیح نیست." ]
          bad_contract_token:
            summary: توکن قرارداد ارسال شده قابلیت کنسل شدن را ندارد.(قرارداد فعال نشده باشد یا معتبر نباشد)
            value:
              detail: "توکن قرارداد صحیح نیست."
    PayResponseV2:
      type: object
      properties:
        detail:
          - type: string
        code:
          - type: string
        examples:
          user_account_disabled:
            summary: حساب کاربر غیرفعال باشد.
            value:
              detail: "حساب غیرفعال است."
              code: "account.disabled"
          insufficient_wallet_balance:
            summary: موجودی کیف پول کاربر هنگام پرداخت با دایرکت‌پی کافی نباشد
            value:
              detail: "موجودی حساب کافی نیست."
              code: "account.insufficient_balance"
          direct_debit_less_than_allowed:
            summary: حداقل مقدار مجاز برای پرداخت با دایرکت دبیت رعایت نشود
            value:
              detail: "حداقل مقدار تراکنش ۱۰۰۰۰ ریال است."
              code: "purchase.minimum_amount_violation"
          checkout_token_expired:
            summary: توکن پرداخت که از اندپوینت init checkout گرفته شده است، منقضی شده باشد
            value:
              detail: "توکن پرداخت منقضی شده است، لطفاً فرآیند پرداخت را از اول آغاز کنید."
              code: "purchase.timeout"
          direct_pay_is_not_active:
            summary: قرارداد دایرک‌ت‌پی کاربر به هر علتی فعال نشده باشد
            value:
              detail: "قرارداد کاربر فعال نیست."
              code: "direct_pay.inactive_contract"
          contract_pay_over_max:
            summary: حداکثر سقف مجاز در بازه‌ی ثبت شده برای یک قرارداد (مانند یک قرارداد هفتگی با سقف ۱۰ هزارتومان)، رعایت نشده باشد
            value:
              detail: "با توجه به قرارداد شما، در بازه هفتگی نمی‌توانید از ویژگی پرداخت مستقیم بیش از ۱۰,۰۰۰ تومان استفاده کنید."
              code: "direct_pay.period_amount_limit_violation"
          insufficient_direct_debit_balance:
            summary: موجودی کارت کاربر که دایرکت‌دبیت را با آن فعال کرده است هنگام پرداخت با دایرکت‌پی کافی نباشد
            value:
              detail: "موجودی کافی در حساب بانکی خود ندارید"
              code: "direct_debit.insufficient_bank_account_balance"
          direct_debit_is_not_active:
            summary: امکان پرداخت با دایرکت‌دبیت وجود نداشته باشد، مانند زمانی که قرارداد کاربر منقضی، لغو یا در حال پردازش باشد
            value:
              detail: "شما سرویس پرداخت سریع خود را فعال نکرده‌اید."
              code: "direct_debit.inactive_contract"
          direct_debit_card_expired:
            summary: کارت کاربر به هر علتی (منقضی شده باشد یا توسط بانک) غیرفعال شده باشد
            value:
              detail: "کارت غیرفعال می‌باشد."
              code: "direct_debit.inactive_card"
          direct_debit_disabled_by_user:
            summary: قرارداد دایرکت‌دیبت توسط کاربر از سمت بانک لغو شده باشد
            value:
              detail: "شما قرارداد خود را از سمت بانک لغو کرده‌اید، لطفا قرارداد فعلی را لغو و مجددا قرارداد جدیدی ایجاد کنید."
              code: "direct_debit.inactive_contract"
          direct_debit_over_max_per_transaction:
            summary: اقدام به پرداخت بیشتر از مبلغ مجاز ثبت شده در قرارداد دایرکت‌دبیت شود (این مبلغ ممکن هست وابسته به قرارداد دایرکت‌دبیت متفاوت باشد)
            value:
              detail: "مبلغ تراکنش شما بیش از مقدار مجاز در سرویس پرداخت مستقیم است."
              code: "direct_debit.transaction_amount_limit_violation"
          direct_debit_over_max_amount_of_transaction_per_day:
            summary: اقدام به پرداخت بیشتر از مبلغ مجاز روزانه ثبت شده در قرارداد دایرکت‌دبیت شود (این مبلغ ممکن هست وابسته به قرارداد دایرکت‌دبیت متفاوت باشد)
            value:
              detail: "شما به سقف مقدار تراکنش روزانه پرداخت مستقیم رسیده‌اید."
              code: "direct_debit.daily_transaction_amount_limit_violation"
          direct_debit_over_max_count_of_transactions_per_day:
            summary: اقدام به پرداخت بیشتر از تعداد مجاز روزانه ثبت شده در قرارداد دایرکت‌دبیت شود
            value:
              detail: "شما به سقف تعداد تراکنش روزانه پرداخت مستقیم رسیده‌اید."
              code: "direct_debit.daily_transaction_count_limit_violation"
          direct_debit_over_max_amount_of_transactions_per_month:
            summary: اقدام به پرداخت بیشتر از مبلغ مجاز ماهانه ثبت شده در قرارداد دایرکت‌دبیت شود (این مبلغ ممکن هست وابسته به قرارداد دایرکت‌دبیت متفاوت باشد)
            value:
              detail: "شما به سقف مقدار تراکنش ماهانه پرداخت مستقیم رسیده‌اید."
              code: "direct_debit.monthly_transaction_amount_limit_violation"
          direct_debit_over_max_count_of_transactions_per_month:
            summary: اقدام به پرداخت بیشتر از تعداد مجاز ماهانه ثبت شده در قرارداد دایرکت‌دبیت شود
            value:
              detail: "شما به سقف تعداد تراکنش ماهانه پرداخت مستقیم رسیده‌اید."
              code: "direct_debit.monthly_transaction_count_limit_violation"
          direct_debit_withdrawal_not_allowed:
            summary: برداشت از حساب بانکی مجاز نیست
            value:
              detail: "برداشت از سپرده مجاز نیست."
              code: "direct_debit.withdrawal_not_allowed"
          invalid_national_id:
            summary: کد ملی مربوط به قرارداد، معتبر نیست
            value:
              detail: "کد ملی نامعتبر است."
              code: "direct_debit.invalid_national_id"
          blocked_bank_account:
            summary: حساب بانکی مسدود است
            value:
              detail: "حساب بانکی شما مسدود شده است، لطفا برای رفع مشکل به بانک مراجعه کنید."
              code: "direct_debit.blocked_bank_account"
          inactive_card:
            summary: کارت بانکی غیر فعال است
            value:
              detail: "کارت غیرفعال است."
              code: "direct_debit.inactive_card"
          expired_card:
            summary: کارت بانکی منقضی شده‌است
            value:
              detail: "کارت منقضی شده‌است"
              code: "direct_debit.inactive_card"
          direct_debit_service_down:
            summary: سرویس دایرکت‌دبیت به هر علتی غیرفعال شده باشد
            value:
              detail: "امکان برقراری ارتباط با بانک وجود ندارد، لطفا از سایر روش‌های پرداخت استفاده کنید یا دقایقی دیگر مجددا تلاش کنید."
              code: "internal_server_error"
          direct_debit_not_allowed:
            summary: امکان پرداخت با دایرکت‌دبیت وجود نداشته باشد، مانند زمانی که درگاه غیرفعال باشد یا به یک علت امکان استفاده از آن وجود نداشته باشد
            value:
              detail: "کاربر مجاز به انتخاب این درگاه نمی‌باشد."
              code: "error"
```
