## DirectPay

این فرآیند برای پرداخت مستقیم از سمت پذیرنده (merchant) بدون دخالت کاربر در مواردی همچون تمدید خودکار اشتراک استفاده
می‌شود.

#### Initiate contract:

برای ساخت قرارداد دایرکت پی  پذیرنده (merchant) باید محدودیت حدکثر میزان تراکنش و دوره‌ی زمانی این محدودیت را مشخص کند

به طور مثل اگر پذیرنده قراردادی با محدودیت ۱ میلیون تومان به صورت هفتگی برای کاربر ایجاد کن. بعد از امضای این قرارداد توسط کاربر پذیرنده حداکثر هفته ای ۱ میلیون تومان میتواند پرداخت مستقیم از حساب کاربر انجام دهد.

```yaml
paths:
  { base_url }/direct-pay/contract/init/
    post:
      summary: initiate-contract
      requestBody:
        content:
          application/json:
            schema:
              type:
                type: string
                enum: [ direct_debit, wallet ]
                example: "wallet"
              period:
                type: string
                enum: [ weekly, monthly, yearly ]
                example: "monthly"          
                description: دوره‌ی زمانی از ابتدا ماه، هفته ویا سال شروع می‌شود. به این معنی که اگر قرارداد کاربر ۱۳ خرداد ماه امضا شود دوره‌ی زمانی آن تا یکم تیر ماه است و از یکم تیر ماه محدودیت میزان تراکنش ریست می‌شود.
              amount_limit:
                type: int
                min_value: 100000
                max_value: 1000000000
                example: 1000000
      responses:
        '200':
          content:
            application/json:
              schema:
                contract_token:
                  type: string
                  example: "9bb790a3-44fd-486f-8ce8-38aa02cab069"
```

example:

```bash
curl --location --request POST 'https://pardakht.cafebazaar.ir/pardakht/badje/v1/direct-pay/contract/init' \
--header 'Content-Type: application/json' \
--header 'Authorization: Token cxvndf40824nfpw98he899jb440f66bt6ac8c30a' \
--data-raw '{
    "type": "wallet"
    "period": "monthly"
    "amount_limit": 1000000
}'

{"contract_token":"9bb790a3-44fd-486f-8ce8-38aa02cab069"}
```

#### Finalize Contract Without SDK:

برای امضای قرارداد توسط کاربر، می‌بایست کاربر را به صفحه‌ی امضای قرارداد ریدایرکت کنید. برای این کار توکن قرارداد
دریافتی از `init contract` را به همراه `redirect_url` به این آدرس پاس می‌دهید و کاربر پس از امضای قرارداد به کلاینت
مرچنت
ریدایرکت می‌شود.

```yaml
servers:
  - url: https://cafebazaar.ir/bazaar-pay/

paths:
  { servers/url }/contract/direct-pay
    get:
      summary: finalize-contract-without-sdk
      parameters:
        - name: token
          in: query
          description: contract_token received from init-contract / توکن قرارداد گرفته شده از ای‌پی‌آی Init Contract
          schema:
            type: string
            example: "9bb790a3-44fd-486f-8ce8-38aa02cab069"
        - name: redirect_url
          in: query
          description: user will be redirected to this address after the process
          schema:
            type: string
            example: "https://example.com/bazaar-pay-return/direct-pay-contract"
        - name: phone_number
          in: query
          description: phone number suggested to user for login in bazaar-pay / شماره موبایلی که پس از ریدایرکت به صفحه‌ی بازارپی برای لاگین به کاربر پیشنهاد می‌شود.
          schema:
            type: string
            example: "09999999999"
```

#### Open Finalize Contract flow:

برای آغاز فلوی نهایی‌سازی قرارداد می‌بایست از sdk وب بازارپی استفاده نمایید.

```typescript
import {startFinalizeContractProccess} from 'bazaar-payment-sdk';

startFinalizeContractProccess(contractToken, callBackUrl, phoneNumber)
    .then(() => {
        // user will be redirected to provided callBackUrl.
    });
```

با استفاده از تابع `startFinalizeContractProccess` پاپ‌آپ نهایی‌سازی قرارداد دایرکت‌پی باز می‌شود و پس از اتمام فرایند
نهایی‌سازی کلاینت به `callBackUrl` ریدایرکت می‌شود.
پس از ریدایرکت کاربر می‌توانید با استفاده از ای‌پی‌آی `Trace Contract` از وضعیت نهایی قرارداد مطلع شوید.

###### startFinalizeContractProcess Arguments:

```yaml
arguments:
  - name: contractToken
    type: string
  - name: callbackUrl
    type: CallBackUrl
  - name: phoneNumber
    type: string
```

CallbackUrl type:

```typescript
interface CallbackUrl {
    url?: string;
    method: "post" | "get";
    data: {
        [propName: string]: string;
    }
}
```

#### Trace contract:

```yaml
paths:
  { base_url }/direct-pay/contract/trace/
    get:
      summary: trace-contract
      parameters:
        - in: query
          name: contract_token
          schema:
            type: string
          required: true
          example: "9bb790a3-44fd-486f-8ce8-38aa02cab069"
          description: contract_token received from init-contract / توکن قرارداد گرفته شده از ای‌پی‌آی Init Contract
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                type: object
                properties:
                  state:
                    type: string
                    enum:
                      - new                  # قرارداد به تازگی ساخته شده است. ممکن است کاربر هنوز در پروسه‌ی دادن رضایت باشد.
                      - active               # راداد فعال است و می‌توان پرداخت‌مستقیم انجام داد.
                      - declined             # قراداد توسط کاربر رد شده است.
                      - cancelled            # قرارداد به دلیل اینکه کاربر اکشنی انجام نداده است منقضی شده است و اعتبار ندارد.
                    example: "new"
                  expiration_time:
                    type: string
                    description: تاریخ و زمان منقضی شدن قرارداد در فرمت ISO
                    example: 2024-06-25T09:28:34.668933Z
                  limit_amount:
                    type: integer
                    description: ماکزیمم مبلغ در هر دوره
                    example: 100000
                  limit_remaining_amount:
                    type: integer
                    description: مبلغ باقی‌مانده مجاز در دوره جاری
                    example: 98730
                  limit_expiration_time:
                    type: string
                    description: تاریخ و زمان پایان دوره فعلی (زمان ریست شدن محدودیت مبلغ) ISO
                    example: 2024-03-20T00:00:00Z
```

مثال:

```bash
curl --location 'https://pardakht.cafebazaar.ir/pardakht/badje/v1/direct-pay/contract/trace?contract_token=9bb790a3-44fd-486f-8ce8-38aa02cab069' \
--header 'Content-Type: application/json' \--header 'Authorization: Token cxvndf40824nfpw98he899jb440f66bt6ac8c30a' 

{"state":"active","expiration_time":"2024-06-25T09:28:34.668933Z","amount_limit":100000,"limit_remaining_amount":98730,"limit_expiration_time":"2024-03-20T00:00:00Z"}
```

#### Direct Pay:

نیاز است که قبل از فراخوانی این ای‌پی‌آی، قرارداد رضایت کاربر تایید شده باشد. قبل از صدا زدن این ای‌پی‌آی باید توسط
ای‌پی‌آی init checkout یه توکن چک‌اوت ساخته شده باشد. از این توکن مانند فرآیند عادی پرداخت برای Trace و Refund می‌توان
استفاده کرد.

```yaml
paths:
  { base_url }/direct-pay/
    post:
      summary: trace-contract
      requestBody:
        content:
          application/json:
            schema:
              contract_token:
                type: string
                example: "9bb790a3-44fd-486f-8ce8-38aa02cab069"
                description: contract_token received from init-contract / توکن قرارداد گرفته شده از ای‌پی‌آی Init Contract
              checkout_token:
                type: string
                example: "0123456789"
                description: contract_token received from init-checkout / توکن چک‌اوت گرفته شده از ای‌پی‌آی init checkout
      responses:
        '204':
          description: payment made successfully
```

مثال:

```bash
curl --location --request POST 'https://pardakht.cafebazaar.ir/pardakht/badje/v1/direct-pay/' \
  --header 'Content-Type: application/json' \
  --data-raw '{
    "contract_token": "9bb790a3-44fd-486f-8ce8-38aa02cab069",
    "checkout_token": "0123456789"
}'
```

در صورت موفقیت، یک Body خالی را برمی‌گرداند.
نسبت به توکن چک‌اوت، idempotent است.
تفاوت این روش با پرداخت عادی که از طریق sdk توسط کاربر انجام می‌شود این است که نیازی به فراخوانی ای‌پی‌آی commit نیست و
اتوماتیک کامیت می‌شود. بنابراین بهتر است این ای‌پی‌آی بعد از ارائه‌ی محصول به کاربر و در یک تراکنش اتمیک فراخوانی گردد.
