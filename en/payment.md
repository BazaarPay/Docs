<h1 id="payment">Payment</h1>

<h2 id="init-checkout">Create checkout token</h2>

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
                  description: Checkout token amount
                destination:
                  type: string
                  required: true
                  example: "developers"
                  description: Merchant unique name
                service_name:
                  type: string
                  required: true
                  minimum: 1
                  maximum: 512
                  example: "product 1"
                  description: Merchant provided service name
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
      $ref: './en/shared-components/security.md#/securitySchemes/ApiKeyAuth'
```

If you need to enable authentication in this API for your merchant, please inform the BazaarPay team.

<h3 id="init-checkout-sample-curl">Sample cURL</h3>

```curl
curl --location --request POST 'https://api.bazaar-pay.ir/badje/v1/checkout/init/' \
  --header 'Content-Type: application/json' \
  --data-raw '{
    "amount": 50000,
    "destination": "developers",
    "service_name": "product 1"
}'
```

<h3 id="init-checkout-sample-success-response">Sample successful response</h3>

```json
{
	"checkout_token": "0123456789",
	"payment_url": "https://cafebazaar.ir/bazaar-pay/payment?token=0123456789"
}
```

<h2 id="payment-flow">Payment flow</h2>

<h3 id="payment-flow-android">Android SDK</h3>

To implement this type of payment method, visit the following address:

[Android SDK Guide](https://github.com/cafebazaar/BazaarPay)

<h3 id="payment-flow-web">Web</h3>

If the acceptor is unable to implement the SDK for any reason or there is no SDK for the desired platform, the user can be redirected directly to the BazaarPay portal and after the user completes the payment flow, the user will be redirected to the requested page with the payment status.

1.After receiving the payment_url in the response to the init-checkout call, you must redirect the user to that URL. You can also add the query params described in the table below to the payment_url:

```yaml
QueryParams:
  - name: token
    type: string
    required: true
    description: The `payment_url` that is used in the output of this endpoint will be returned, you can put this value in the query param or for convenience just use `/checkout/init/`. This is the token used for the payment process.
  - name: redirect_url
    type: string
    format: encodedURL    # must be encoded by encodeURIComponent
    required: true
    description: The return address to the merchant where the user will be returned if successful/unsuccessful. Be sure to include your return address.
  - name: phone
    type: string
    required: false
    description: The user's contact number, which is automatically filled in to speed up the login process.
```

<h4 id="payment-flow-web-sample">Example of using a payment url</h4>

This address will be returned in `/checkout/init/`.

```
https://{base_url}{base_path}/payment?token=checkout_token
```

If you use the provided query params, the sample address will look like this:

```
https://{base_url}{base_path}/payment?token=checkout_token&phone=user_phone_number&redirect_url=merchant_redirect_url
```

And the final example would look like this:

```
https://cafebazaar.ir/bazaar-pay/payment?token=3258455376&phone=09123456789&redirect_url=https://bazaar-pay.ir
```

<h2 id="commit">Confirm payment</h2>

Calling this endpoint is a declaration of the authenticity of the product or service provided by the acceptor to the user. By calling this endpoint, the transaction is finalized and if this endpoint is not called, the entire transaction amount will be automatically returned to the user's wallet after a few minutes.

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
                  description: Checkout token you got from init checkout
      responses:
        '204':
          description: checkout committed successfully
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
      $ref: './en/shared-components/security.md#/securitySchemes/ApiKeyAuth'
```

If you need to enable authentication in this API for your merchant, please inform the BazaarPay team.

<h3 id="commit-sample-curl">Sample cURL</h3>

```curl
curl --location --request POST 'https://api.bazaar-pay.ir/badje/v1/commit/' \
--header 'Content-Type: application/json' \
--data-raw '{
    "checkout_token": "some_token"
}'

```

<h2 id="refund">Full/Partial Refund</h2>

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
                  description: Checkout token you got from init checkout
                amount:
                  type: integer
                  description: The amount that needs to be returned from the purchase, if not specified, it will be returned in full.
                  required: false
      responses:
        '204':
          description: refunded successfully
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
      $ref: './en/shared-components/security.md#/securitySchemes/ApiKeyAuth'
```

* Idempotent only with respect to the checkout token.
* Only one refund can be issued per checkout, and the amount of the refund will be equal to the first successful execution of this endpoint.

<h3 id="refund-sample-curl">Sample cURL</h3>

```curl
curl --location --request POST 'https://api.bazaar-pay.ir/badje/v1/refund/' \
--header 'Content-Type: application/json' \
--header 'Authorization: Token some_auth_token \
--data-raw '{
    "checkout_token": "some_token"
}'
```

<h2 id="trace">Trace payment</h2>

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
                  description: Checkout token you got from init checkout
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
                      - invalid_token                 # Checkout token is invalid.
                      - unpaid                        # The user has not paid and it has not been long since the checkout was created. In this case, the user may still be paying.
                      - paid_not_committed            # The user has paid the money but it has not been committed. The money may have been returned to the user's wallet.
                      - paid_not_committed_refunded   # The user has paid the money but it has not been committed and the time allowed for committing has passed, the money has been returned to the user's wallet.
                      - paid_committed                # The user has paid and it has been committed by the merchant.
                      - refunded                      # The user's payment has been returned to their wallet after being committed. The refund must have been made by the merchant.
                      - timed_out                     # The token has expired and the token has expired.
                    example: "paid_committed"
        '400':
          $ref: './en/shared-components/error-responses.md#/responses/400'
        '503':
          $ref: './en/shared-components/error-responses.md#/responses/503'
```

<h3 id="trace-sample-curl">Sample cURL</h3>

```curl
curl --location --request POST 'https://api.bazaar-pay.ir/badje/v1/trace/' \
  --header 'Content-Type: application/json' \
  --data-raw '{
    "checkout_token": "some_token"
  }'
```

<h3 id="trace-sample-success-response">Sample successful response</h3>

```json
{
	"status": "paid_committed"
}
```

<h2 id="get-checkouts-status">Checkout status</h2>

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
                  description: Creation time
                  example: "2022-11-15T00:00"
                end_time:
                  type: string
                  format: iso-8601
                  description: Refund time
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
      $ref: './en/shared-components/security.md#/securitySchemes/ApiKeyAuth'
```
* The interval between these two days should not be more than 31 days.
* If the value of filter_date_by is equal to refund_date, only checkouts that have a value for the refund_datetime field will be displayed in the output. So as a result, only checkouts that have been refunded will be displayed in the output.
* If the value of filter_date_by is equal to payment_date, only checkouts that have a value for the payment_datetime field will be displayed in the output. So as a result, only checkouts that have been paid will be displayed in the output.

<h3 id="get-checkouts-status-sample-curl">Sample cURL</h3>

```curl
curl --location --request POST 'https://api.bazaar-pay.ir/badje/v1/get-checkouts-status/' \
--header 'Authorization: Token {token}' --header 'Content-Type: application/json' \
--data-raw '{
    "start_datetime": "2022-11-15T00:00", "end_datetime": "2022-11-15T14:00", "filter_date_by": "creation_date"
}'
```

<h3 id="get-checkouts-status-sample-success-response">Sample successful response</h3>

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

<h2 id="bank-refund">Full/Partial Refund (Wallet Refund + Bank Refund)</h2>

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

* It is idempotent only with respect to the checkout token.
* Each checkout allows only a single refund. The refund amount will correspond to the amount specified in the first successful invocation of this endpoint.
