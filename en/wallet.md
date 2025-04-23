<h1 id="wallet">Wallet</h1>

To integrate BazaarPay wallet and merchant application

<h2 id="get-balance">Get user balance</h2>

This feature requires special access. If needed, contact BazaarPay support to obtain it.

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
          description: User mobile number
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
                    example: "7,971 TMN"
        '401':
          $ref: './en/shared-components/error-responses.md#/responses/401'
        '403':
          $ref: './en/shared-components/error-responses.md#/responses/403'
        '400':
          $ref: './en/shared-components/error-responses.md#/responses/400'
        '503':
          $ref: './en/shared-components/error-responses.md#/responses/503'
components:
  securitySchemes:
    ApiKeyAuth:
      $ref: "./en/shared-components/security.md#/securitySchemes/ApiKeyAuth";
```

example:

```bash
curl --location 'api.bazaar-pay.ir/badje/v1/get-balance/?user_phone_number=09120000000' \
  --header 'Authorization: Token merchant_token'
```

<h2 id="increase-balance">Increase user balance</h2>

To increase the balance of the BazarPay wallet, direct the user to the following address.

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
    description: The user is taken to the inventory increase page.
    parameters:
      - name: redirect_url
        in: query
        required: true
        schema:
          type: string
          example: "https://cafebazaar.ir/bazaar-pay/increase-balance?redirect_url=https://bazaar-pay.ir"
        description: The address must be encoded as encodeURIComponent
```

:bulb: After moving to the balance increase page, one of the options available to the user is the automatic wallet recharge contract.
If the user does not have an active automatic recharge contract at this moment, he will be able to activate it on this page and
if he has previously activated the automatic recharge contract, he will have access to its settings.

:bulb: **Automatic wallet recharge** allows the user to always keep his BazaarPay wallet charged. In this way,
after the wallet balance decreases to **Minimum balance**, the user's wallet will be automatically recharged using **Direct payment**.
