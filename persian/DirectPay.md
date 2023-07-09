## DirectPay

این فرآیند برای پرداخت مستقیم از سمت پذیرنده (merchant) بدون دخالت کاربر در مواردی همچون تمدید خودکار اشتراک استفاده
می‌شود.

#### Initiate contract:

برای ساخت قرارداد دایرکت پی پذیرنده (merchant) باید محدودیت حداکثر میزان تراکنش و دوره‌ی زمانی این محدودیت را مشخص کند.

به طور مثل اگر پذیرنده، قراردادی با محدودیت ۱ میلیون تومان به صورت هفتگی برای کاربر ایجاد کند، بعد از امضای این قرارداد
توسط کاربر پذیرنده حداکثر هفته‌ای ۱ میلیون تومان می‌تواند پرداخت مستقیم را از حساب کاربر انجام دهد.

```yaml
openapi: 3.1.0
info:
  title: BazaarPay API
  version: 1.0.0
servers:
  - url: 'https://pardakht.cafebazaar.ir'
paths:
  /pardakht/badje/v1/direct-pay/contract/init:
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
                    - weekly
                    - monthly
                    - yearly
                  example: "yearly"
                  description: |
                    دوره‌ی زمانی از ابتدا هفته، ماه یا سال شروع می‌شود. به این معنی که اگر قرارداد کاربر ۱۳ خرداد ماه امضا شود، دوره‌ی زمانی آن تا یکم تیر ماه است و از یکم تیر ماه محدودیت میزان تراکنش ریست می‌شود، مقادیر مجاز شامل:
                      - weekly: هفتگی، در صورتی که قرارداد اعتبار داشته باشد، محدودیت میزان تراکنش از اول هفته (شنبه) ریست می‌شود
                      - monthly: ماهانه، در صورتی که قرارداد اعتبار داشته باشد، محدودیت میزان تراکنش از اول ماه ریست می‌شود
                      - yearly: سالانه، در صورتی که قرارداد اعتبار داشته باشد، محدودیت میزان تراکنش از اول سال ریست می‌شود
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
          $ref: './shared/unsuccessful-response.yml#/responses/401'
        '400':
          $ref: './shared/unsuccessful-response.yml#/responses/400'
components:
  securitySchemes:
    ApiKeyAuth:
      $ref: './shared/security.yml#/securitySchemes/ApiKeyAuth'
```

cURL Example:

```curl
curl --location --request POST 'https://pardakht.cafebazaar.ir/pardakht/badje/v1/direct-pay/contract/init' \
--header 'Authorization: Token {merchant_token}' \
--data-raw '{
    "type": "wallet"
    "period": "monthly"
    "amount_limit": 1000000
}'
```

Success Response Example:

```json
{
	"contract_token": "9bb790a3-44fd-486f-8ce8-38aa02cab069"
}
```

#### Finalize Contract Without SDK:

برای امضای قرارداد توسط کاربر، می‌بایست کاربر را به صفحه‌ی امضای قرارداد ریدایرکت کنید. برای این کار توکن قرارداد
دریافتی از `init contract` را به همراه `redirect_url` به این آدرس پاس می‌دهید و کاربر پس از امضای قرارداد به کلاینت
مرچنت
ریدایرکت می‌شود.

```yaml
openapi: 3.1.0
info:
  title: BazaarPay Web
  version: 1.0.0
servers:
  - url: 'https://cafebazaar.ir'
paths:
  /bazaar-pay/contract/direct-pay:
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
        - name: phone_number
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

#### Open Finalize Contract flow:

برای آغاز فلوی نهایی‌سازی قرارداد می‌بایست از sdk وب بازارپی استفاده نمایید.

```typescript
import {startFinalizeContractProccess} from 'bazaar-payment-sdk';

startFinalizeContractProccess(contractToken, callBackUrl, phoneNumber)
    .then(() => {
        // The user will be redirected to the provided callBackUrl.
    });
```

با استفاده از تابع `startFinalizeContractProccess` پاپ‌آپ نهایی‌سازی قرارداد دایرکت‌پی باز می‌شود و پس از اتمام فرآیند
نهایی‌سازی، کلاینت به آدرس بازگشت (`callBackUrl`) ریدایرکت می‌شود.
پس از ریدایرکت کاربر می‌توانید با استفاده از اندپوینت `Trace Contract` از وضعیت نهایی قرارداد مطلع شوید.

###### startFinalizeContractProcess Arguments:

```yaml
arguments:
  - name: contractToken
    type: string
  - name: callBackUrl
    type: CallbackUrl
  - name: phoneNumber
    type: string
```

CallbackUrl Type:

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

```typescript
interface CallbackUrl {
    url?: string;
    method: "post" | "get";
    data: {
        [propName: string]: string;
    }
}
```

#### Trace Contract:

```yaml
openapi: 3.1.0
info:
  title: BazaarPay API
  version: 1.0.0
servers:
  - url: https://pardakht.cafebazaar.ir
paths:
  /pardakht/badje/v1/direct-pay/contract/trace/:
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
                    example: new
                    description: |
                      وضعیت قرار داد کاربر، مقدیر مجاز برابر است با:
                      - new: قرارداد به تازگی ساخته شده است. ممکن است کاربر هنوز در پروسه‌ی دادن رضایت باشد
                      - active: قراداد فعال است و می‌توان پرداخت‌مستقیم انجام داد
                      - declined: قراداد توسط کاربر رد شده است
                      - cancelled: قرارداد به دلیل اینکه کاربر اکشنی انجام نداده است منقضی شده است و اعتبار ندارد
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
          $ref: './shared/unsuccessful-response.yml#/responses/401'
        '400':
          $ref: './shared/unsuccessful-response.yml#/responses/400'
components:
  securitySchemes:
    ApiKeyAuth:
      $ref: './shared/security.yml#/securitySchemes/ApiKeyAuth'
```

cURL Example:

```curl
curl --location 'https://pardakht.cafebazaar.ir/pardakht/badje/v1/direct-pay/contract/trace?contract_token=af72319b-9bae-4c2b-9cbf-76cs119a4582' \
--header 'Authorization: Token {merchant_token}' 
```

Success Response Example:

```json
{
	"state": "declined",
	"expiration_time": "2024-07-02T06:43:44.843212Z",
	"amount_limit": 100000,
	"limit_remaining_amount": 100000,
	"limit_expiration_time": "2025-07-02T06:43:44.843212Z"
}
```

#### Direct Pay:

نیاز است که قبل از فراخوانی این اندپوینت، قرارداد کاربر تایید (فعال) شده باشد. قبل از صدا زدن این اندپوینت باید توسط
اندپوینت init checkout یک توکن چک‌اوت ساخته شده باشد تا بتوان عملیات پرداخت را با آن انجام داد. می‌توان از این توکن در
اندپوینت‌های Trace و Refund نیز
استفاده کرد.

```yaml
openapi: 3.1.0
info:
  title: BazaarPay API
  version: 1.0.0
servers:
  - url: https://pardakht.cafebazaar.ir
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
          $ref: './shared/unsuccessful-response.yml#/responses/401'
        '400':
          $ref: './shared/unsuccessful-response.yml#/responses/400'
components:
  securitySchemes:
    ApiKeyAuth:
      $ref: './shared/security.yml#/securitySchemes/ApiKeyAuth'
```

cURL Example:

```curl
curl --location 'https://pardakht.cafebazaar.ir/pardakht/badje/v1/direct-pay?lang=fa' \
--header 'Authorization: Token {merchant_token}' \
--data '{
    "contract_token": "7f9bf78c-a5e2-4126-9482-3s484b3706be",
    "checkout_token": "1443280167"
}'
```

در صورت موفقیت، یک Body خالی را برمی‌گرداند.
نسبت به توکن چک‌اوت، idempotent است.
تفاوت این روش با پرداخت عادی که از طریق SDK توسط کاربر انجام می‌شود این است که نیازی به فراخوانی اندپوینت commit نیست و
اتوماتیک کامیت (تایید خرید) انجام می‌شود. بنابراین بهتر است این اندپوینت بعد از ارائه‌ی محصول به کاربر و در یک تراکنش
اتمیک فراخوانی گردد.
