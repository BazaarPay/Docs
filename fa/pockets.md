<h1 id="charge-pockets-fa">شارژ جیب</h1>

این API به پذیرندگان امکان شارژ جیب کاربران را می‌دهد.

<h2 id="charge-fa">شارژ</h2>

این اندپوینت یک یا چند جیب را در یک گروه از جیب‌های مشخص شارژ می‌کند.

برای دسترسی به این اندپوینت و یافتن شناسه (slug) گروه جیب‌هایی که می‌توانید آن‌ها رو شارژ کنید، با همکاران ما در تیم محصول بازارپی تماس بگیرید.
```yaml
openapi: 3.1.0
info:
  title: API شارژ کیف پول
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
paths:
  /pockets/{pocket_group_slug}/charge/:
    post:
      summary: شارژ کیف پول‌ها
      security:
        - ApiKeyAuth: []
      parameters:
        - name: pocket_group_slug
          in: path
          required: true
          schema:
            type: string
            example: "my-pocket-group"
          description: اسلاگ گروه کیف پولی که باید شارژ شود.
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                charges:
                  type: array
                  items:
                    type: object
                    properties:
                      token:
                        type: string
                        format: uuid
                        description: یک توکن یکتا برای جلوگیری از درخواست‌های تکراری. این توکن باید uuid7 باشد.
                      phone_number:
                        type: string
                        description: شماره تلفن کاربر.
                      amount:
                        type: integer
                        description: مبلغ شارژ.
      responses:
        '200':
          description: موفق
          content:
            application/json:
              schema:
                type: object
                properties:
                  results:
                    type: array
                    items:
                      type: object
                      properties:
                        token:
                          type: string
                          format: uuid
                        phone_number:
                          type: string
                        amount:
                          type: integer
                        status:
                          type: string
                          enum:
                            - success
                            - error
                        message:
                          type: string
            example:
                results:
                  - token: "01988f7a-d22b-7c3d-977f-27af99c2a5a5"
                    phone_number: "989120000000"
                    amount: 10000
                    status: "success"
                    message: "شارژ با موفقیت انجام شد."
                  - token: "01988f7a-d22b-7c3d-977f-27af99c2a5a5"
                    phone_number: "989120000001"
                    amount: 20000
                    status: "error"
                    message: "توکن تکراری است"
        '401':
          $ref: './fa/shared-components/error-responses.md#/responses/401'
        '403':
          $ref: './fa/shared-components/error-responses.md#/responses/403'
        '404':
          $ref: './fa/shared-components/error-responses.md#/responses/404'

components:
  securitySchemes:
    ApiKeyAuth:
      $ref: "./fa/shared-components/security.md#/securitySchemes/ApiKeyAuth"
```

مثال:

```bash
curl --location --request POST 'https://api.bazaar-pay.ir/badje/v1/pockets/my-pocket-group/charge/' \
--header 'Authorization: Token merchant_token' \
--header 'Content-Type: application/json' \
--data-raw '{
    "charges": [
        {
            "token": "a8c99f2c-3f4a-4b56-896b-2b1e7b3f4e5a",
            "phone_number": "989120000000",
            "amount": 10000
        }
    ]
}'
```

<h2 id="trace-charge-fa">پیگیری شارژ</h2>

این اندپوینت وضعیت یک شارژ خاص را با استفاده از توکن یکتا پیگیری می‌کند.

```yaml
openapi: 3.1.0
info:
  title: API پیگیری شارژ
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: API v1 بازارپی
paths:
  /pockets/{pocket_group_slug}/charge/{token}/:
    get:
      summary: پیگیری شارژ
      security:
        - ApiKeyAuth: []
      parameters:
        - name: pocket_group_slug
          in: path
          required: true
          schema:
            type: string
            example: "my-pocket-group"
          description: اسلاگ گروه کیف پول.
        - name: token
          in: path
          required: true
          schema:
            type: string
            format: uuid
          description: توکن یکتای شارژ.
      responses:
        '200':
          description: موفق
          content:
            application/json:
              schema:
                type: object
                properties:
                  results:
                    type: array
                    items:
                      type: object
                      properties:
                        token:
                          type: string
                          format: uuid
                        phone_number:
                          type: string
                        amount:
                          type: integer
              example:
                results:
                  - token: "01988f7a-d22b-7c3d-977f-27af99c2a5a5"
                    phone_number: "989120000000"
                    amount: 10000
        '401':
          $ref: './fa/shared-components/error-responses.md#/responses/401'
        '403':
          $ref: './fa/shared-components/error-responses.md#/responses/403'
        '404':
          $ref: './fa/shared-components/error-responses.md#/responses/404'

components:
  securitySchemes:
    ApiKeyAuth:
      $ref: "./fa/shared-components/security.md#/securitySchemes/ApiKeyAuth"
```

مثال:

```bash
curl --location 'https://api.bazaar-pay.ir/badje/v1/pockets/my-pocket-group/charge/a8c99f2c-3f4a-4b56-896b-2b1e7b3f4e5a/' \
--header 'Authorization: Token merchant_token'
```
