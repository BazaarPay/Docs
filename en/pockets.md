<h1 id="charge-pockets">Charge Pockets</h1>

This API allows merchants to charge user pockets.

<h2 id="charge">Charge</h2>

This endpoint charges one or more user pockets within a specified pocket group.

To access this endpoint and find the slug for the pocket group you can charge, please contact our colleagues on the BazarPay product team.

```yaml
openapi: 3.1.0
info:
  title: Charge Pockets API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
paths:
  /pockets/{pocket_group_slug}/charge/:
    post:
      summary: Charge Pockets
      security:
        - ApiKeyAuth: []
      parameters:
        - name: pocket_group_slug
          in: path
          required: true
          schema:
            type: string
            example: "my-pocket-group"
          description: The slug of the pocket group to charge.
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
                        description: A unique token for idempotency. The token must be UUID7.
                      phone_number:
                        type: string
                        description: The user's phone number.
                      amount:
                        type: integer
                        description: The amount to charge.
      responses:
        '200':
          description: Success
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
                    message: "Charge was successful."
                  - token: "01988f7a-d22b-7c3d-977f-27af99c2a5a5"
                    phone_number: "989120000001"
                    amount: 20000
                    status: "error"
                    message: "Duplicate token."
        '401':
          $ref: './en/shared-components/error-responses.md#/responses/401'
        '403':
          $ref: './en/shared-components/error-responses.md#/responses/403'
        '404':
          $ref: './en/shared-components/error-responses.md#/responses/404'

components:
  securitySchemes:
    ApiKeyAuth:
      $ref: "./en/shared-components/security.md#/securitySchemes/ApiKeyAuth"
```

example:

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

<h2 id="trace-charge">Trace Charge</h2>

This endpoint traces the status of a specific charge using the idempotency token.

```yaml
openapi: 3.1.0
info:
  title: Trace Charge API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
paths:
  /pockets/{pocket_group_slug}/charge/{token}/:
    get:
      summary: Trace Charge
      security:
        - ApiKeyAuth: []
      parameters:
        - name: pocket_group_slug
          in: path
          required: true
          schema:
            type: string
            example: "my-pocket-group"
          description: The slug of the pocket group.
        - name: token
          in: path
          required: true
          schema:
            type: string
            format: uuid
          description: The idempotency token of the charge.
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
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
                token: "01988f7a-d22b-7c3d-977f-27af99c2a5a5"
                phone_number: "989120000000"
                amount: 10000
        '401':
          $ref: './en/shared-components/error-responses.md#/responses/401'
        '403':
          $ref: './en/shared-components/error-responses.md#/responses/403'
        '404':
          $ref: './en/shared-components/error-responses.md#/responses/404'

components:
  securitySchemes:
    ApiKeyAuth:
      $ref: "./en/shared-components/security.md#/securitySchemes/ApiKeyAuth"
```

example:

```bash
curl --location 'https://api.bazaar-pay.ir/badje/v1/pockets/my-pocket-group/charge/a8c99f2c-3f4a-4b56-896b-2b1e7b3f4e5a/' \
--header 'Authorization: Token merchant_token'
