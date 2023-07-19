# پرداخت

## ایجاد چک‌اوت توکن

```yaml
paths:
  { base_url }/checkout/init/
    post:
      summary: init-checkout
      requestBody:
        content:
          application/json:
            schema:
              amount:
                type: int
                format: int64
                required: true
                example: 50000
                minimum: 0
                maximum: 1,000,000,000
                description: مقدار توکن چگ‌اوت
              destination:
                type: string
                required: true
                example: "developers"
                description: نام مرچنت که به ازای هر مرچنت یکتا است
              service_name:
                type: string
                required: true
                minimum: 1
                maximum: 512
                example: "product 1"
                description: نام سرویس/خدمت ارایه شده
      responses:
        '200':
          content:
            application/json:
              schema:
                checkout_token:
                  type: string
                  example: "0123456789"
                payment_url:
                  type: string
                  format: url
                  example: "https://cafebazaar.ir/user/payment?token=0123456789"
```

### نمونه cURL

```curl
curl --location --request POST 'https://pardakht.cafebazaar.ir/pardakht/badje/v1/checkout/init/' \
  --header 'Content-Type: application/json' \
  --data-raw '{
    "amount": 50000,
    "destination": "developers",
    "service_name": "product 1"
}'
```

### نمونه پاسخ درخواست

```json
{
	"checkout_token": "0123456789",
	"payment_url": "https://cafebazaar.ir/user/payment?token=0123456789"
}
```

## فلو پرداخت

### Web SDK

پس از گرفتن توکن چک‌اوت می‌بایست با استفاده از اسکریپت زیر پاپ‌آپ بازارپی برای کاربر باز شود.

```typescript
import {startPaymentProccess} from 'bazaar-payment-sdk';

startPaymentProccess(checkoutToken, callBackUrl, phoneNumber)
    .then(() => {
        // code to execute when payment flow is finished.
    });

interface CallbackUrl {
    url?: string;
    method: "post" | "get";
    data: {
        [propName: string]: string;
    }
}
```

#### startPaymentProcess Arguments:

```yaml
arguments:
  - name: checkoutToken
    type: string
  - name: callbackUrl
    type: CallBackUrl
  - name: phoneNumber
    type: string
```

#### [Android SDK](https://github.com/cafebazaar/BazaarPay)

استفاده از کیت توسعه اندروید بازارپی بسیار ساده است. شما فقط باید یک نمونه از کلاس BazaarPayLauncher ایجاد کنید. سپس از
این نمونه برای شروع فرآیند پرداخت با فراخوانی تابع launchBazaarPay استفاده کنید.

سه نکته وجود دارد که باید در نظر بگیرید:

۱. شما باید یک نمونه از BazaarPayLauncher را به عنوان ویژگی کلاس ایجاد کنید. (این مورد اجباری است زیرا از Activity
Result API استفاده می کند. برای کسب اطلاعات بیشتر به این لینک مراجعه کنید.)

۲. هنگام ایجاد یک نمونه جدید از کلاس BazaarPayLauncher باید سه پارامتر را پاس کنید:

- context: باید پیاده‌سازی رابط ActivityResultCaller باشد.
- onSuccess: این تابع پس از یک فرآیند پرداخت موفق فراخوانی می شود.
- onCancel: اگر فرآیند پرداخت با موفقیت به پایان نرسد (لغو توسط کاربر) این تابع فراخوانی می‌شود.

۳. هنگام فراخوانی تابع launchBazaarPay باید checkout توکن دریافتی پرداخت را به عنوان پارامتر ارسال کنید. ضمن اینکه
می‌توانید شماره تلفن کاربر هم به عنوان پارامتر اختیاری phoneNumber ارسال نمایید تا به صورت پیش‌فرض در فیلد شماره تلفن
مربوط به لاگین قرار گیرد. دو پارامتر اختیاری دیگر با مقادیر پیش فرض وجود دارد (isEnglish = false / isDarkMode = false).
با استفاده از این پارامترها می توانید زبان (پیش فرض فارسی است) و حالت تاریک (پیش فرض روشن است) را به صورت دستی تنظیم
کنید.

نکته‌ی مهم: در پایان خرید موفق کاربر، شما باید در سمت سرور خرید کاربر را توسط API Commit تایید کنید. صدا زدن این
ای‌پی‌آی به منزله اعلام صحت ارائه‌ی محصول یا خدمت توسط پذیرنده به کاربر است. با صدا زدن این ای‌پی‌آی تراکنش ثبت شده
نهایی شده و در صورت عدم صدا زدن این ای‌پی‌آی کل مبلغ تراکنش به صورت خودکار به کیف پول کاربر بعد از چند دقیقه باز خواهد
گشت. راه توصیه شده برای انجام این کار فراخوانی توسط سرور است، در صورتی که برنامه دارای سرور نیست این امکان در SDK فراهم
شده است. این کار توسط فراخوانی متد commit با پارامتر ورودی checkout توکن انجام می‌شود، در صورتی که موفقیت آمیز باشد متد
onSuccess اجرا شده و در غیر این صورت خطای مورد نظر به تابع onFailure پاس داده می‌شود. نکته‌ی دیگر این که این متد suspend
بوده و برای فراخوانی آن باید از کوروتین استفاده نمایید، در غیر این صورت طبق راهنمای ای‌پی‌آی commit فراخوانی را بر اساس
تکنولوژی مورد استفاده‌ی خود انجام دهید.

#### Without SDK:

در صورتی که پذیرنده به هر دلیلی امکان پیاده‌سازی SDK را نداشته باشد یا برای پلتفرم مورد نظر SDK وجود نداشته باشد،
به‌صورت مستقیم می‌توان کاربر را به درگاه بازارپی هدایت کرد و پس از آنکه کاربر فلوی پرداخت را طی نمود، کاربر با وضعیت
پرداختش به صفحه‌ی درخواست شده ریدایرکت می‌شود.

۱. پس از دریافت payment_url در پاسخ فراخوانی init-checkout، باید کاربر را به آن یوآرال هدایت کنید. همچنین می‌توانید
کوئری پارامترهای توضیح داده شده در جدول زیر را به payment_url اضافه کنید:

```yaml
arguments:
  - name: redirect_url
    type: string
    format: url    # encoded by encodeURIComponent
    description: user will be redirected to this address after process
  - name: phone
    type: string
    description: phone number suggested to user for login in bazaar-pay
```

example:

Init-checkout’s payment_url:
https://cafebazaar.ir/user/payment?token=my_checkout_token

Redirect user to:
https://cafebazaar.ir/user/payment?token=my_checkout_token&redirect_url=https://your-web-site.omg/status&phone=09194950906

## تایید خرید

صدا زدن این ای‌پی‌آی به منزله اعلام صحت ارائه‌ی محصول یا خدمت توسط پذیرنده به کاربر است. با صدا زدن این ای‌پی‌آی تراکنش
ثبت شده نهایی شده و در صورت عدم صدا زدن این ای‌پی‌آی کل مبلغ تراکنش به صورت خودکار به کیف پول کاربر بعد از چند دقیقه باز
خواهد گشت.

```yaml
paths:
  { base_url }/commit/
    post:
      summary: commit
      requestBody:
        content:
          application/json:
            schema:
              checkout_token:
                type: string
                description: contract_token received from init-checkout / توکن چک‌اوت گرفته شده از ای‌پی‌آی init checkout
      responses:
        '204':
          description: checkout committed successfully
```

### نمونه cURL

```curl
curl --location --request POST 'https://pardakht.cafebazaar.ir/pardakht/badje/v1/commit/' \
--header 'Content-Type: application/json' \
--data-raw '{
    "checkout_token": "some_token"
}'

```

## بازگشت کامل/جزیی خرید

```yaml
paths:
  { base_url }/refund/
    post:
      security: ApiKeyAuth
      summary: refund
      requestBody:
        content:
          application/json:
            schema:
              checkout_token:
                type: string
                description: contract_token received from init-checkout / توکن چک‌اوت گرفته شده از ای‌پی‌آی init checkout
              amount:
                type: int
                description: refund amount
                required: false
      responses:
        '204':
          description: refunded successfully
```

* تنها نسبت به توکن چک‌اوت، idempotent است.
* به ازای هر چک‌اوت، تنها یک بار می‌توان پول را بازگرداند و مقدار این بازگشت پول، برابر با اولین اجرای موفق این ای‌پی‌آی
  خواهد بود.

### نمونه cURL

```curl
curl --location --request POST 'https://pardakht.cafebazaar.ir/pardakht/badje/v1/refund/' \
--header 'Content-Type: application/json' \
--header 'Authorization: Token some_auth_token \
--data-raw '{
    "checkout_token": "some_token"
}'
```

## پیگیری خرید

```yaml
paths:
  { base_url }/trace/
    post:
      summary: trace
      requestBody:
        content:
          application/json:
            schema:
              checkout_token:
                type: string
                description: contract_token received from init-checkout / توکن چک‌اوت گرفته شده از ای‌پی‌آی init checkout
                example: "0123456789"
      responses:
        '200':
          content:
            application/json:
              schema:
                status:
                  type: string
                  enum:
                    - invalid_token        # چک‌اوت توکن ورودی درست نیست.
                    - unpaid               # کاربر پول را پرداخت نکرده است و زمان زیادی از ساخت چک اوت نگذشته است. در این حالت ممکن است کاربر هنوز در حال پرداخت باشد.
                    - paid_not_committed   # کاربر پول را پرداخت کرده است ولی کامیت نشده است. ممکن است که پول به کیف پول کاربر بازگردانده شده باشد.
                    - paid_committed       # کاربر پول را پرداخته کرده و توسط مرچنت کامیت هم شده است.
                    - refunded             # پرداخت کاربر پس از کامیت شدن به کیف پولش بازگشته است. ریفاند حتما توسط مرچنت انجام شده است.
                    - timed_out            # کاربر پول را پرداخت نکرده است. (فرآیند کنسل شده است) 
                  example: "paid_committed"
```

### نمونه cURL

```curl
curl --location --request POST 'https://pardakht.cafebazaar.ir/pardakht/badje/v1/trace/' \
  --header 'Content-Type: application/json' \
  --data-raw '{
    "checkout_token": "some_token"
  }'
```

### نمونه پاسخ موفق درخواست

```json
{
	"status": "paid_committed"
}
```

## گزارش وضعیت چک‌اوت‌ها

```yaml
paths:
  { base_url }/get-checkouts-status/
    post:
      summary: checkouts-status
      requestBody:
        content:
          application/json:
            schema:
              start_datetime:
                type: string
                format: iso-8601
                description: زمان ساخت
                example: "2022-11-15T00:00"
              end_time:
                type: string
                format: iso-8601
                description: زمان ریفاند
                example: "2022-11-15T14:00"
              filter_date_by:
                type: string
                enum:
                  - creation_date
                  - refund_date
      responses:
        '200':
          content:
            application/json:
              schema:
                checkouts:
                  type: array
                  items:
                    oneOf:
                      - type: object
                        properties:
                          token:
                            type: string
                          amount:
                            type: int
                          service_name:
                            type: string
                          created_datetime:
                            type: string
                            format: iso-8601
                          status:
                            type: string
                            enum: [ invalid_token, unpaid,paid_not_committed, paid_committed, refunded, timed_out ]
                          is_committed:
                            type: boolean
                          commit_datetime:
                            type: string
                            format: iso-8601
                          refund_datetime:
                            type: string
                            format: iso-8601
                next:
                  type: string
                  format: url
                  description: url to request for next page of checkouts
                previous:
                  type: string
                  format: url
                  description: url to request for previous page of checkouts
```

* فاصله‌ی بین این دو روز نباید بیشتر از ۳۱ روز باشه.
* اگر مقدار filter_date_by برابر refund_date باشد، فقط checkout هایی در خروجی نمایش داده میشوند که دارای مقدار برای فیلد
  refund_datetime باشند. پس در نتیجه فقط checkout هایی که ریفاند شده اند در خروجی نمایش داده می‌شوند.

### نمونه cURL

```curl
curl --location --request POST 'https://pardakht.cafebazaar.ir/pardakht/badje/v1/get-checkouts-status/' \
--header 'Authorization: Token {token}' --header 'Content-Type: application/json' \
--data-raw '{
    "start_datetime": "2022-11-15T00:00", "end_datetime": "2022-11-15T14:00", "filter_date_by": "creation_date"
}'
```

### نمونه پاسخ موفق درخواست

```json
{
	"checkouts": [
		{
			"token": "checkout_token1",
			"amount": 90000,
			"service_name": "service name 1",
			"created_datetime": "2022-11-14T21:13:12.030613Z",
			"status": "timed_out",
			"commit_datetime": null,
			"refund_datetime": null,
			"is_committed": true
		},
		{
			"token": "checkout_token2",
			"amount": 50000,
			"service_name": "service name 2",
			"created_datetime": "2022-11-14T21:13:12.030613Z",
			"status": "paid_committed",
			"commit_datetime": "2022-11-14T21:13:12.030613Z",
			"refund_datetime": null,
			"is_committed": false
		},
		{
			"token": "checkout_token2",
			"amount": 50000,
			"service_name": "service name 2",
			"created_datetime": "2022-11-14T21:13:12.030613Z",
			"status": "refunded",
			"commit_datetime": "2022-11-14T21:13:12.030613Z",
			"refund_datetime": "2022-11-14T21:13:12.030613Z",
			"is_committed": true
		}
	],
	"next": "https://pardakht-secure.cafebazaar.org/pardakht/badje/v1/get-checkouts-status/?cursor=qweqweqweqwe",
	"previous": null
}
```

