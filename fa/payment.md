<h1 id="payment">پرداخت</h1>

<h2 id="init-checkout">ایجاد چک‌اوت توکن</h2>

```yaml
openapi: 3.1.0
info:
  title: Initialize Checkout Token API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
paths:
  /checkout/init/:
    post:
      summary: init-checkout
      security:
        - ApiKeyAuth: [ ]
        - { }
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
                    example: "https://cafebazaar.ir/bazaar-pay/payment?token=0123456789"
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

در صورت نیاز به فعال شدن احراز‌هویت در این‌ ای‌پی‌آی برای مرچنت شما،‌به تیم بازارپی اطلاع دهید.

<h3 id="init-checkout-sample-curl">نمونه cURL</h3>

```curl
curl --location --request POST 'https://api.bazaar-pay.ir/badje/v1/checkout/init/' \
  --header 'Content-Type: application/json' \
  --data-raw '{
    "amount": 50000,
    "destination": "developers",
    "service_name": "product 1"
}'
```

<h3 id="init-checkout-sample-success-response">نمونه پاسخ موفق درخواست</h3>

```json
{
  "checkout_token": "0123456789",
  "payment_url": "https://cafebazaar.ir/bazaar-pay/payment?token=0123456789"
}
```

<h2 id="payment-flow">فلو پرداخت</h2>

<h3 id="payment-flow-android">Android SDK</h3>

برای پیاده‌سازی این نوع روش پراخت به آدرس زیر مراجعه کنید:

[Android SDK Guide](https://github.com/cafebazaar/BazaarPay)

<h3 id="payment-flow-web">Web</h3>

در صورتی که پذیرنده به هر دلیلی امکان پیاده‌سازی SDK را نداشته باشد یا برای پلتفرم مورد نظر SDK وجود نداشته باشد،
به‌صورت مستقیم می‌توان کاربر را به درگاه بازارپی هدایت کرد و پس از آنکه کاربر فلوی پرداخت را طی نمود، کاربر با وضعیت
پرداختش به صفحه‌ی درخواست شده ریدایرکت می‌شود.

۱. پس از دریافت payment_url در پاسخ فراخوانی init-checkout، باید کاربر را به آن یوآرال هدایت کنید. همچنین می‌توانید
کوئری پارام‌های توضیح داده شده در جدول زیر را به payment_url اضافه کنید:

```yaml
QueryParams:
  - name: token
    type: string
    required: true
    description: که در خروجی این اندپوینت قرار دارد استفاده کنید `payment_url` برگرداننده می‌شود، شما می‌توانید این مقدار را در کوئری پارام قرار دهید یا برای سهولت کار تنها از  `/checkout/init/`  توکن مورد استفاده برای فرآیند پرداخت است، این مقدار در
  - name: redirect_url
    type: string
    format: encodedURL    # must be encoded by encodeURIComponent
    required: true
    description: آدرس بازگشتی به مرچنت که کاربر در صورت موفق/ناموفق بودن به این آدرس بازگشت داده می‌شود، حتما آدرس بازگشتی خود را اندکد کنید
  - name: phone
    type: string
    required: false
    description: شماره تماس کاربر که برای سرعت بخشیدن به فرآیند لاگین به صورت خودکار پر می‌شود
```

<h4 id="payment-flow-web-sample">نمونه استفاده از آدرس پرداخت</h4>

این آدرس توسط درخواست `/checkout/init/` برگرداننده می‌شود.

```
https://{base_url}{base_path}/payment?token=checkout_token
```

در صورتی که از کوئری‌پارام‌های فراهم شده استفاده کنید، نمونه آدرس مانند زیر می‌شود:

```
https://{base_url}{base_path}/payment?token=checkout_token&phone=user_phone_number&redirect_url=merchant_redirect_url
```

مثال نمونه‌ی نهایی آن مانند زیر می‌شود:

```
https://cafebazaar.ir/bazaar-pay/payment?token=3258455376&phone=09123456789&redirect_url=https://bazaar-pay.ir
```

<h2 id="commit">تایید خرید</h2>

صدا زدن این اندپوینت به منزله اعلام صحت ارائه‌ی محصول یا خدمت توسط پذیرنده به کاربر است. با صدا زدن این اندپوینت تراکنش
ثبت شده نهایی شده و در صورت عدم صدا زدن این اندپوینت کل مبلغ تراکنش به صورت خودکار به کیف پول کاربر بعد از چند دقیقه باز
خواهد گشت.

```yaml
openapi: 3.1.0
info:
  title: Commit checkout Token API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
paths:
  /commit/:
    post:
      summary: commit
      security:
        - ApiKeyAuth: [ ]
        - { }
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

در صورت نیاز به فعال شدن احراز‌هویت در این‌ ای‌پی‌آی برای مرچنت شما،‌به تیم بازارپی اطلاع دهید.

<h3 id="commit-sample-curl">نمونه cURL</h3>

```curl
curl --location --request POST 'https://api.bazaar-pay.ir/badje/v1/commit/' \
--header 'Content-Type: application/json' \
--data-raw '{
    "checkout_token": "some_token"
}'

```

<h2 id="refund">بازگشت کامل/جزیی خرید</h2>

```yaml
openapi: 3.1.0
info:
  title: Refund Checkout Token API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
paths:
  /refund/:
    post:
      summary: refund
      security:
        - ApiKeyAuth: [ ]
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

* تنها نسبت به توکن چک‌اوت، idempotent است.
* به ازای هر چک‌اوت، تنها یک بار می‌توان پول را بازگرداند و مقدار این بازگشت پول، برابر با اولین اجرای موفق این اندپوینت
  خواهد بود.

<h3 id="refund-sample-curl">نمونه cURL</h3>

```curl
curl --location --request POST 'https://api.bazaar-pay.ir/badje/v1/refund/' \
--header 'Content-Type: application/json' \
--header 'Authorization: Token some_auth_token \
--data-raw '{
    "checkout_token": "some_token"
}'
```

<h2 id="trace">پیگیری خرید</h2>

```yaml
openapi: 3.1.0
info:
  title: Trace Checkout Token API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
paths:
  /trace/:
    post:
      summary: trace
      security:
        - { }
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
                      - invalid_token                 # چک‌اوت توکن ورودی درست نیست.
                      - unpaid                        # کاربر پول را پرداخت نکرده است و زمان زیادی از ساخت چک اوت نگذشته است. در این حالت ممکن است کاربر هنوز در حال پرداخت باشد.
                      - paid_not_committed            # کاربر پول را پرداخت کرده است ولی کامیت نشده است. ممکن است که پول به کیف پول کاربر بازگردانده شده باشد.
                      - paid_not_committed_refunded   # کاربر پول را پرداخت کرده است ولی کامیت نشده است و بازه‌ی زمانی مجاز برای کامیت کردن سپری شده، پول به کیف پول کاربر بازگردانده شده است.
                      - paid_committed                # کاربر پول را پرداخته کرده و توسط مرچنت کامیت هم شده است.
                      - refunded                      # پرداخت کاربر پس از کامیت شدن به کیف پولش بازگشته است. ریفاند حتما توسط مرچنت انجام شده است.
                      - timed_out                     # زمان استفاده از توکن به اتمام رسیده و توکن منقضی شده است. 
                    example: "paid_committed"
        '400':
          $ref: './fa/shared-components/error-responses.md#/responses/400'
        '503':
          $ref: './fa/shared-components/error-responses.md#/responses/503'
```

<h3 id="trace-sample-curl">نمونه cURL</h3>

```curl
curl --location --request POST 'https://api.bazaar-pay.ir/badje/v1/trace/' \
  --header 'Content-Type: application/json' \
  --data-raw '{
    "checkout_token": "some_token"
  }'
```

<h3 id="trace-sample-success-response">نمونه پاسخ موفق درخواست</h3>

```json
{
  "status": "paid_committed"
}
```

<h2 id="get-checkouts-status">گزارش وضعیت چک‌اوت‌ها</h2>

```yaml
openapi: 3.1.0
info:
  title: Get Checkout Statuses API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
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
                    - payment_date
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
                              enum:
                                - invalid_token
                                - unpaid,paid_not_committed
                                - paid_not_committed_refunded
                                - paid_committed
                                - refunded
                                - timed_out
                            is_committed:
                              type: boolean
                            payment_datetime:
                              type: string
                              format: iso-8601
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

* فاصله‌ی بین این دو روز نباید بیشتر از ۳۱ روز باشه.
* اگر مقدار filter_date_by برابر refund_date باشد، فقط checkout هایی در خروجی نمایش داده میشوند که دارای مقدار برای فیلد
  refund_datetime باشند. پس در نتیجه فقط checkout هایی که ریفاند شده‌اند در خروجی نمایش داده می‌شوند.
* اگر مقدار filter_date_by برابر payment_date باشد، فقط checkout هایی در خروجی نمایش داده میشوند که دارای مقدار برای
  فیلد
  payment_datetime باشند. پس در نتیجه فقط checkout هایی که پرداخت شده‌اند در خروجی نمایش داده می‌شوند.

<h3 id="get-checkouts-status-sample-curl">نمونه cURL</h3>

```curl
curl --location --request POST 'https://api.bazaar-pay.ir/badje/v1/get-checkouts-status/' \
--header 'Authorization: Token {token}' --header 'Content-Type: application/json' \
--data-raw '{
    "start_datetime": "2022-11-15T00:00", "end_datetime": "2022-11-15T14:00", "filter_date_by": "creation_date"
}'
```

<h3 id="get-checkouts-status-sample-success-response">نمونه پاسخ موفق درخواست</h3>

```json
{
  "checkouts": [
    {
      "token": "checkout_token1",
      "amount": 90000,
      "service_name": "service name 1",
      "created_datetime": "2022-11-14T21:13:12.030613Z",
      "status": "timed_out",
      "payment_datetime": null,
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
      "payment_datetime": "2022-11-14T21:13:12.030613Z",
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
      "payment_datetime": "2022-11-14T21:13:12.030613Z",
      "commit_datetime": "2022-11-14T21:13:12.030613Z",
      "refund_datetime": "2022-11-14T21:13:12.030613Z",
      "is_committed": true
    }
  ],
  "next": "https://pardakht-secure.cafebazaar.org/pardakht/badje/v1/get-checkouts-status/?cursor=qweqweqweqwe",
  "previous": null
}
```

<h2 id="bank-refund">بازگشت کامل/جزیی خرید (به کیف پول یا بانک)</h2>

```yaml
openapi: 3.1.0
info:
  title: Refund Checkout Token API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v3}'
    description: BazaarPay API v3
paths:
  /bank-refund/:
    post:
      summary: Request for Wallet Refund + Bank Refund
      security:
        - ApiKeyAuth: [ ]
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                checkout_token:
                  type: string
                  description: The checkout token obtained from the `init-checkout` endpoint.
                amount:
                  type: integer
                  description: The amount that needs to be refunded from the purchase. If not specified, the full amount will be refunded.
                  required: false
      responses:
        '202':
          description: refunded successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  refunded_to:
                    type: string
                    enum: [ wallet, bank ]
                    description: Explains that the balance has been refunded to wallet and bank or only refunded to bank.
        '204':
          description: the request has been applied before
        '401':
          $ref: './fa/shared-components/error-responses.md#/responses/401'
        '403':
          $ref: './fa/shared-components/error-responses.md#/responses/403'
        '400':
          $ref: './fa/shared-components/error-responses.md#/responses/400'
        '503':
          $ref: './fa/shared-components/error-responses.md#/responses/503'
    get:
      summary: Trace for Wallet Refund + Bank Refund
      security:
        - ApiKeyAuth: [ ]
      parameters:
        - in: path
          required: true
          name: checkout_token
          schema:
            type: string
          description: The checkout token obtained from the `init-checkout` endpoint.
      responses:
        '200':
          description: refunded successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  bank_state:
                    type: string
                    enum: [ in_progress, successful, failed, not_started ]
                    description: The `not_started` state means that the balance has been refunded to wallet only.
        '401':
          $ref: './fa/shared-components/error-responses.md#/'
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

* تنها نسبت به توکن چک‌اوت، idempotent است.
* به ازای هر چک‌اوت، تنها یک بار می‌توان پول را بازگرداند و مقدار این بازگشت پول، برابر با اولین اجرای موفق این اندپوینت
  خواهد بود.
