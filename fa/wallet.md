# کیف پول

برای یکپارچه کردن کیف پول بازارپی و اپلیکیشن مرچنت

## دریافت موجودی کاربر

این قابلیت نیازمند دارا بودن یک دسترسی ویژه می‌باشد، در صورت نیاز، برای دریافت آن با پشتیبانی بازارپی در ارتباط باشید.

```yaml
openapi: 3.1.0
info:
  title: Get User Balance API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
paths:
  /get-balance/:
    get:
      summary: user-balance
      security:
        - ApiKeyAuth: [ ]
      parameters:
        - name: user_phone_number
          in: query
          schema:
            type: string
            example: "09120000000"
          required: true
          description: شماره موبایل کاربر
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  balance:
                    type: number
                    example: 79719
                  balance_string:
                    type: number
                    example: "۷,۹۷۱ تومان"
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
      $ref: "./fa/shared-components/security.md#/securitySchemes/ApiKeyAuth";
```

example:

```bash
curl --location 'pardakht.cafebazaar.ir/pardakht/badje/v1/get-balance/?user_phone_number=09120000000' \
  --header 'Authorization: Token merchant_token'
```

## افزایش موجودی کاربر

برای افزایش موجودی کیف پول بازارپی، کاربر را به آدرس زیر هدایت کنید.

```yaml
openapi: 3.1.0
info:
  title: Open Increase Balance Webpage
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path}'
    description: BazaarPay Web
paths:
  /increase-balance:
    summary: increase-balance
    description: کاربر به صفحه‌ی افزایش موجودی منتقل می‌شود
    parameters:
      - name: redirect_url
        in: query
        required: true
        schema:
          type: string
          example: "https://cafebazaar.ir/bazaar-pay/increase-balance?redirect_url=https://bazaar-pay.ir"
        description: آدرس باید به صورت encodeURIComponent انکد شود
```
