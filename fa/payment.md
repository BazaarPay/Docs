# پرداخت

## ایجاد چک‌اوت توکن

```yaml
openapi: 3.1.0
info:
  title: BazaarPay API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path}'
paths:
  /checkout/init/:
    post:
      summary: init-checkout
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                amount:
                  type: integer
                  format: int64
                  required: true
                  example: 50000
                  minimum: 0
                  maximum: 1000000000
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
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  checkout_token:
                    type: string
                    example: "0123456789"
                  payment_url:
                    type: string
                    format: url
                    example: "https://cafebazaar.ir/user/payment?token=0123456789"
components:
  securitySchemes:
    ApiKeyAuth:
      $ref: './fa/shared_components/security.yml#/securitySchemes/ApiKeyAuth'
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

### نمونه پاسخ موفق درخواست

```json
{
	"checkout_token": "0123456789",
	"payment_url": "https://cafebazaar.ir/user/payment?token=0123456789"
}
```

## فلو پرداخت

### Web SDK

در ابتدا این پکیج را با توضیحاتی که در لینک زیر قرار دارد نصب کنید:

[پکیج SDK وب](https://www.npmjs.com/package/@cafebazaar/payment-sdk)

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

### Android SDK

برای پیاده‌سازی این نوع روش پراخت به آدرس زیر مراجعه کنید:

[Android SDK Guide](https://github.com/cafebazaar/BazaarPay)

### Without SDK

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

صدا زدن این اندپوینت به منزله اعلام صحت ارائه‌ی محصول یا خدمت توسط پذیرنده به کاربر است. با صدا زدن این اندپوینت تراکنش
ثبت شده نهایی شده و در صورت عدم صدا زدن این اندپوینت کل مبلغ تراکنش به صورت خودکار به کیف پول کاربر بعد از چند دقیقه باز
خواهد گشت.

```yaml
openapi: 3.1.0
info:
  title: BazaarPay API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path}'
paths:
  /commit/:
    post:
      summary: commit
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                checkout_token:
                  type: string
                  description: توکن چک‌اوت گرفته شده از اندپوینت init checkout
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
openapi: 3.1.0
info:
  title: BazaarPay API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path}'
paths:
  /refund/:
    post:
      summary: refund
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                checkout_token:
                  type: string
                  description: توکن چک‌اوت گرفته شده از اندپوینت init checkout
                amount:
                  type: integer
                  description: مقداری که نیاز است از خرید بازگشت داده شود، در صورتی که مقداردهی نشود به صورت کلی بازگشت داده می‌شود
                  required: false
      responses:
        '204':
          description: refunded successfully
components:
  securitySchemes:
    ApiKeyAuth:
      $ref: './fa/shared_components/security.yml#/securitySchemes/ApiKeyAuth'
```

* تنها نسبت به توکن چک‌اوت، idempotent است.
* به ازای هر چک‌اوت، تنها یک بار می‌توان پول را بازگرداند و مقدار این بازگشت پول، برابر با اولین اجرای موفق این اندپوینت
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
openapi: 3.1.0
info:
  title: BazaarPay API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path}'
paths:
  /trace/:
    post:
      summary: trace
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                checkout_token:
                  type: string
                  description: توکن چک‌اوت گرفته شده از اندپوینت init checkout
                  example: "0123456789"
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
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
components:
  securitySchemes:
    ApiKeyAuth:
      $ref: './fa/shared_components/security.yml#/securitySchemes/ApiKeyAuth'
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
openapi: 3.1.0
info:
  title: BazaarPay API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path}'
paths:
  /get-checkouts-status/:
    post:
      summary: checkouts-status
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
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
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  checkouts:
                    type: array
                    items:
                      oneOf:
                        - type: object
                          properties:
                            token:
                              type: string
                            amount:
                              type: integer
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
components:
  securitySchemes:
    ApiKeyAuth:
      $ref: './fa/shared_components/security.yml#/securitySchemes/ApiKeyAuth'
```

* فاصله‌ی بین این دو روز نباید بیشتر از ۳۱ روز باشه.
* اگر مقدار filter_date_by برابر refund_date باشد، فقط checkout هایی در خروجی نمایش داده میشوند که دارای مقدار برای فیلد
  refund_datetime باشند. پس در نتیجه فقط checkout هایی که ریفاند شده‌اند در خروجی نمایش داده می‌شوند.

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

