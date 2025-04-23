<h1 id="direct-pay">DirectPay</h1>

This process is used for automatic payment from the merchant side without user intervention and consists of the following steps:

1. [Create DirectPay contract](#init-contract)
2. [Activate/Cancel DirectPay contract](#verify-contract)
3. [Trace DirectPay contract](#trace-contract)
4. [Cancel DirectPay contract](#cancel-contract)
5. [Payment with DirectPay](#payment)
6. [Get user DirectPay balance](#get-balance)

<h2 id="init-contract">Create direct pay contract token</h2>

At this stage, the merchant creates a DirectPay contract. This contract has not yet been approved by the user and cannot be used for a payment.
To create a DirectPay contract, the merchant must specify the maximum transaction amount limit and the time period for this limit.
For example, if the merchant creates a contract with a limit of 1 million Tomans per week for the user, after the user signs this contract, the merchant can make a maximum of 1 million Tomans per week of direct payments from the user's account.

<h3 id="init-contract-sample">Sample</h3>

```yaml
openapi: 3.1.0
info:
  title: Initialize Directpay Contract API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
paths:
  /direct-pay/contract/init/:
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
                    - direct_debit: Direct debit from user bank account
                    - wallet: Wallet
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
                    - daily
                    - weekly
                    - monthly
                    - quarterly
                    - semiannual
                    - yearly
                  example: "yearly"
                  description: |
                    The time period starts from the beginning of the week, month or year. This means that if the user contract is signed on June 13, the time period is until July 1, and from July 1, the transaction limit is reset. The allowed values ​​include:
                      - daily: daily, if the contract is valid, the daily transaction limit is reset
                      - weekly: weekly, if the contract is valid, the transaction limit is reset from the beginning of the week (Saturday)
                      - monthly: monthly, if the contract is valid, the transaction limit is reset from the beginning of the month
                      - quarterly: quarterly, if the contract is valid, the transaction limit is reset from the beginning of the quarter
                      - semiannual: semiannual, if the contract is valid, the transaction limit is reset from the beginning of the six months
                      - yearly: annual, if the contract is valid, the transaction limit is reset from the beginning of the year
                redirect_url:
                  required: false
                  type: url
                  default: null
                  description: |
                    The address to which the user will be redirected after completing the process.
                owner_phone_number:
                  required: false
                  type: str
                  default: null
                  example: "989123456789" | "09123456789" | "9123456789"
                  description: |
                    This field is optional.
                    In this field, we get from the acceptor the mobile number of the user that the acceptor expects to finalize this contract (confirm or reject the contract).
                    If this field is filled in by the acceptor, we check on the contract finalization page
                    whether the mobile number of the user who is finalizing this contract matches the mobile number that the acceptor sent.
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
                  redirect_url:
                    type: string
                    format: url
                    example: "https://app.bazaar-pay.ir/contract/direct-pay?contract_token=7f9bf78c-a5e2-4126-9482-37484b3706be"
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

<h3 id="init-contract-sample-curl">Sample cURL</h3>

```curl
curl --location --request POST 'https://api.bazaar-pay.ir/badje/v1/direct-pay/contract/init/' \
--header 'Authorization: Token {merchant_token}' \
--header 'Content-Type: application/json' \
--data-raw '{
    "type": "wallet",
    "period": "monthly",
    "amount_limit": 1000000,
    "redirect_url": "https://merchant.com"
}'
```

<h3 id="init-contract-sample-success-response">Sample successful response</h3>

```json
{
	"contract_token": "9bb790a3-44fd-486f-8ce8-38aa02cab069"
}
```

<h2 id="verify-contract">Activate/Cancel DirectPay Contract on the Web</h2>

For the user to sign the contract, you need to redirect the user to the `redirect_url` received from `init-contract`. After signing the contract, the user will be redirected to the merchant client.
In the DirectPay contract closing process, if the contract type is Direct Debit and the user does not have an active Direct Debit contract, the user will automatically enter the Direct Debit contract closing process.
You can also add the query params described below to the redirect_url:

(If you have sent the redirect_url in the init-contract API, there is no need to send it again.)

<h3 id="verify-contract-sample">Sample</h3>

```yaml
QueryParams:
  - name: redirect_url
    in: query
    required: true
    description: After activation, the user will be redirected to this address. Be sure to encode your redirect url.
    schema:
      type: string
      format: encodedURL    # must be encoded by encodeURIComponent
      example: https://example.com/bazaar-pay-return/direct-pay-contract
  - name: phone
    in: query
    required: false
    schema:
      type: string
      example: 09999999999
    description: If login is required, the user number is filled in on the page by this parameter. If the user is already logged in, the same user is used to create the contract.
  - name: message
    in: query
    required: false
    schema:
      type: string
      example: An example message
    description: The merchant can display a custom message to the user using this field.
```

<h3 id="verify-contract-sample-address">Sample address</h3>

```
https://cafebazaar.ir/bazaar-pay/contract/direct-pay?contract_token={contract_token}&redirect_url={encoded_url}&phone={user_phone_number}&message={encoded_message}
```

<h2 id="trace-contract">Tracing direct pay contract</h2>

You can check the status of the contract through this endpoint.

<h3 id="trace-contract-sample">Sample</h3>

```yaml
openapi: 3.1.0
info:
  title: Trace Directpay Contract API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
paths:
  /direct-pay/contract/trace/:
    get:
      summary: trace-contract
      parameters:
        - name: contract_token
          in: query
          required: true
          schema:
            type: string
            example: af72319b-9baf-4c2b-9dbf-76cf119a4582
          description: Contract token you got from Init Contract
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
                      - expired
                    example: new
                    description: |
                      The user's contract status, the allowed amount, is:
                        - new: The contract has just been created. The user may still be in the process of giving consent
                        - active: The contract is active and direct payment can be made
                        - declined: The contract has been declined by the user
                        - cancelled: The merchant has cancelled the user's contract
                        - expired: The contract has expired
                  expiration_time:
                    type: string
                    example: 2024-06-25T09:28:34.668933Z
                    format: iso-8601
                    description: Contract Expiry
                  amount_limit:
                    type: integer
                    example: 100000
                    description: Max amount of withdraw in each period
                  limit_remaining_amount:
                    type: integer
                    example: 98730
                    description: Remaining amount in current period
                  limit_expiration_time:
                    type: string
                    format: iso-8601
                    example: 2024-06-25T09:28:34.668933Z
                    description: Current period expiry
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

<h3 id="trace-contract-sample">Example cURL</h3>

```curl
curl --location 'https://api.bazaar-pay.ir/badje/v1/direct-pay/contract/trace/?contract_token=af72319b-9bae-4c2b-9cbf-76cs119a4582' \
--header 'Authorization: Token {merchant_token}' 
```

<h3 id="trace-contract-sample-success-response">Sample successful response</h3>

```json
{
	"state": "declined",
	"expiration_time": "2024-07-02T06:43:44.843212Z",
	"amount_limit": 100000,
	"limit_remaining_amount": 100000,
	"limit_expiration_time": "2025-07-02T06:43:44.843212Z"
}
```

<h2 id="cancel-contract">Cancel direct pay contract</h2>

With this endpoint, the merchant can cancel the user's active contracts.
If you encounter 400 category errors, you can find out the current status of the contract by calling the trace endpoint.

<h3 id="cancel-contract-sample">Sample</h3>

```yaml
openapi: 3.1.0
info:
  title: Cancel Directpay Contract API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
paths:
  /direct-pay/contract/cancel/:
    post:
      summary: cancel-contract
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
                  description: Contract token you got from Init Contract
      responses:
        '204':
          description: Success
        '401':
          $ref: './en/shared-components/error-responses.md#/responses/401'
        '403':
          $ref: './en/shared-components/error-responses.md#/responses/403'
        '400':
          description: Bad Request
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: './en/shared-components/error-responses.md#/responses/400/content/application/json/schema'
                  - $ref: '#/components/schemas/CancelResponse'
        '503':
          $ref: './en/shared-components/error-responses.md#/responses/503'
components:
  securitySchemes:
    ApiKeyAuth:
      $ref: './en/shared-components/security.md#/securitySchemes/ApiKeyAuth'
```

<h3 id="cancel-contract-sample-curl">Sample cURL</h3>

```curl
curl --location 'https://api.bazaar-pay.ir/badje/v1/direct-pay/contract/cancel/?contract_token=af72319b-9bae-4c2b-9cbf-76cs119a4582' \
--header 'Authorization: Token {merchant_token}' 
```

<h2 id="payment">Payment with direct pay</h2>

The user agreement needs to be confirmed before calling this endpoint. Before calling this endpoint, a checkout token must be created by [init checkout endpoint](./payment.md#init-checkout) so that the payment operation can be performed with it. This token can also be used in [Trace](./payment.md#trace) and [Refund](./payment.md#refund) endpoints.

You can also use the second version of the payment, in this version the structure of displaying errors in the response of this request has been improved to make it easier for merchants to use. In addition, merchants can write and use their desired logic by using the value in the `code` json object that is returned in the response of this request in case of an error.

<h3 id="payment-sample">Sample request</h3>

```yaml
openapi: 3.1.0
info:
  title: Pay With Directpay Contract API
  version: 1.0.0 and 2.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
  - url: 'https://{base_url}{base_path_v2}'
    description: BazaarPay API v2
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
                  description: Contract token you got from Init Contract
                checkout_token:
                  type: string
                  required: true
                  example: "0123456789"
                  description: Checkout token you got from Init Checkout
      responses:
        '204':
          description: Success
        '401':
          $ref: './en/shared-components/error-responses.md#/responses/401'
        '403':
          $ref: './en/shared-components/error-responses.md#/responses/403'
        '400':
          description: Bad Request
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: './en/shared-components/error-responses.md#/responses/400/content/application/json/schema'
                  - $ref: '#/components/schemas/PayResponse'
                  - $ref: '#/components/schemas/PayResponseV2'
                    description: "only in v2"
        '500':
          description: Internal Server Error
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: './en/shared-components/error-responses.md#/responses/500/content/application/json/schema'
                  - $ref: '#/components/schemas/PayResponseV2'
                    description: "only in v2"
        '503':
          $ref: './en/shared-components/error-responses.md#/responses/503'
components:
  securitySchemes:
    ApiKeyAuth:
      $ref: './en/shared-components/security.md#/securitySchemes/ApiKeyAuth'
```

<h3 id="payment-sample-curl">Sample cURL</h3>

```curl
curl --location 'https://api.bazaar-pay.ir/badje/v1/direct-pay/' \
--header 'Authorization: Token {merchant_token}' \
--data '{
    "contract_token": "7f9bf78c-a5e2-4126-9482-3s484b3706be",
    "checkout_token": "1443280167"
}'
```

* If successful, returns an empty response.
* It is idempotent with respect to the checkout token.
* The difference between this method and the normal payment made by the user via the SDK is that there is no need to call the commit endpoint and the commit (purchase confirmation) is done automatically. Therefore, it is better to call this endpoint after presenting the product to the user and in an atomic transaction.

<h2 id="payment-sample-errors">Sample errors</h2>

```yaml
components:
  schemas:
    PayResponse:
      type: object
      properties:
        detail:
          oneOf:
            - type: array
              items:
                type: string
            - type: string
        examples:
          direct_pay_is_not_active:
            summary: The user's DirectPay contract has not been activated for any reason.
            value:
              detail: ["User contract is not active."]
          contract_pay_over_max:
            summary: The maximum limit allowed in the registered period for a contract (such as a weekly contract with a limit of 10,000 Tomans) has not been met.
            value:
              detail: "Due to your contract, you cannot use the direct payment feature for more than 10,000 Tomans per week."
          insufficient_wallet_balance:
            summary: The user's wallet balance is not sufficient when paying with DirectPay.
            value:
              detail: "Insufficient balance."
          insufficient_direct_debit_balance:
            summary: The balance on the user's card with which Direct Debit has been activated is insufficient when paying with DirectPay.
            value:
              detail: "You do not have sufficient funds in your bank account."
          direct_debit_less_than_allowed:
            summary: The minimum amount allowed for payment by direct debit is not met.
            value:
              detail: "Minimum transaction amount is 10000R."
          direct_debit_is_not_active:
            summary: Payment by direct debit is not possible, such as when the user's contract has expired, is cancelled, or is in processing.
            value:
              detail: "You have not enabled your Instant Payment."
          direct_debit_not_allowed:
            summary: It is not possible to pay by direct debit, such as when the portal is inactive or cannot be used for some reason.
            value:
              detail: "Selected gateway is not permitted for user."
          checkout_token_expired:
            summary: The payment token obtained from the init checkout endpoint has expired.
            value:
              detail: "The payment token has expired, please start the payment process over."
          direct_debit_card_expired:
            summary: The user's card has been deactivated for any reason (expired or by the bank).
            value:
              detail: "Card is inactive."
          direct_debit_service_down:
            summary: The direct debit service has been disabled for any reason.
            value:
              detail: "It is not possible to communicate with the bank, please use other payment methods or try again in a few minutes."
          direct_debit_disabled_by_user:
            summary: The direct debit agreement has been canceled by the user from the bank.
            value:
              detail: "You have canceled your contract from the bank, please cancel the current contract and create a new contract again."
          direct_debit_over_max_per_transaction:
            summary: Attempt to pay more than the authorized amount recorded in the direct debit agreement (this amount may vary depending on the direct debit agreement)
            value:
              detail: "The transaction amount is more than the allowed amount in the direct debit"
          direct_debit_over_max_amount_of_transaction_per_day:
            summary: Attempt to pay more than the daily allowed amount recorded in the direct debit agreement (this amount may vary depending on the direct debit agreement)
            value:
              detail: "You have reached your daily transaction amount limit for Instant Payment."
          direct_debit_over_max_count_of_transactions_per_day:
            summary: Make a payment exceeding the daily limit recorded in the direct debit agreement (this number is equal to 10 transactions).
            value:
              detail: "You have reached your daily transaction count limit for Instant Payment."
          user_account_disabled:
            summary: The user account is inactive.
            value:
              detail: "Account is disabled"
    CancelResponse:
      type: object
      properties:
        oneOf:
          detail:
            - type: string
          contract_token:
            - type: list
              items:
                - type: string
        examples:
          invalid_contract_token:
            summary: The submitted contract token is not a valid token.
            value:
              contract_token: [ "Invalid contract token." ]
          bad_contract_token:
            summary: The sent contract token cannot be canceled (the contract is not activated or is not valid).
            value:
              detail: "Invalid contract token."
    PayResponseV2:
      type: object
      properties:
        detail:
          - type: string
        code:
          - type: string
        examples:
          user_account_disabled:
            summary: The user account is inactive.
            value:
              detail: "Account is disabled."
              code: "account.disabled"
          insufficient_wallet_balance:
            summary: The user's wallet balance is not sufficient when paying with DirectPay
            value:
              detail: "Insufficient balance."
              code: "account.insufficient_balance"
          direct_debit_less_than_allowed:
            summary: The minimum amount allowed for payment by direct debit is not met.
            value:
              detail: "Minimum transaction amount is 10000R."
              code: "purchase.minimum_amount_violation"
          checkout_token_expired:
            summary: The payment token obtained from the init checkout endpoint has expired.
            value:
              detail: "Please start the payment flow from the beginning."
              code: "purchase.timeout"
          direct_pay_is_not_active:
            summary: The user's DirectPe contract has not been activated for any reason.
            value:
              detail: "User contract is not active."
              code: "direct_pay.inactive_contract"
          contract_pay_over_max:
            summary: The maximum limit allowed in the registered period for a contract (such as a weekly contract with a limit of 10,000 Tomans) has not been met.
            value:
              detail: "Due to your contract, you cannot use the direct payment feature for more than 10,000 Tomans per week."
              code: "direct_pay.period_amount_limit_violation"
          insufficient_direct_debit_balance:
            summary: The balance on the user's card with which Direct Debit has been activated is insufficient when paying with DirectPay.
            value:
              detail: "You don't have enough balance in your bank account."
              code: "direct_debit.insufficient_bank_account_balance"
          direct_debit_is_not_active:
            summary: Payment by direct debit is not possible, such as when the user's contract has expired, is cancelled, or is in processing.
            value:
              detail: "You have not enabled your Instant Payment."
              code: "direct_debit.inactive_contract"
          direct_debit_card_expired:
            summary: The user's card has been deactivated for any reason (expired or by the bank).
            value:
              detail: "Card is inactive."
              code: "direct_debit.inactive_card"
          direct_debit_disabled_by_user:
            summary: The direct debit agreement has been canceled by the user from the bank.
            value:
              detail: "You have canceled your contract from the bank, please cancel the current contract and create a new contract again."
              code: "direct_debit.inactive_contract"
          direct_debit_over_max_per_transaction:
            summary: Attempt to pay more than the authorized amount recorded in the direct debit agreement (this amount may vary depending on the direct debit agreement)
            value:
              detail: "The transaction amount is more than the allowed amount in the direct debit"
              code: "direct_debit.transaction_amount_limit_violation"
          direct_debit_over_max_amount_of_transaction_per_day:
            summary: Attempt to pay more than the daily allowed amount recorded in the direct debit agreement (this amount may vary depending on the direct debit agreement)
            value:
              detail: "You have reached your daily transaction amount limit for Instant Payment."
              code: "direct_debit.daily_transaction_amount_limit_violation"
          direct_debit_over_max_count_of_transactions_per_day:
            summary: Attempt to make a payment exceeding the daily limit recorded in the direct debit agreement.
            value:
              detail: "You have reached your daily transaction count limit for Instant Payment."
              code: "direct_debit.daily_transaction_count_limit_violation"
          direct_debit_over_max_amount_of_transactions_per_month:
            summary: Attempt to pay more than the monthly allowed amount recorded in the direct debit agreement (this amount may vary depending on the direct debit agreement)
            value:
              detail: "You have reached your monthly transaction amount limit for Instant Payment."
              code: "direct_debit.monthly_transaction_amount_limit_violation"
          direct_debit_over_max_count_of_transactions_per_month:
            summary: Make more payments than the monthly limit recorded in the direct debit agreement.
            value:
              detail: "You have reached your monthly transaction count limit for Instant Payment."
              code: "direct_debit.monthly_transaction_count_limit_violation"
          direct_debit_withdrawal_not_allowed:
            summary: Withdrawals from bank accounts are not allowed.
            value:
              detail: "Withdrawal from deposit is not allowed."
              code: "direct_debit.withdrawal_not_allowed"
          invalid_national_id:
            summary: The national ID related to the contract is not valid.
            value:
              detail: "National ID is invalid."
              code: "direct_debit.invalid_national_id"
          blocked_bank_account:
            summary: Bank account is blocked.
            value:
              detail: "Your bank account has been blocked, please go to the bank to solve the problem."
              code: "direct_debit.blocked_bank_account"
          inactive_card:
            summary: Bank card is inactive.
            value:
              detail: "Card is inactive"
              code: "direct_debit.inactive_card"
          expired_card:
            summary: Bank card has expired.
            value:
              detail: "Card is expired"
              code: "direct_debit.inactive_card"
          direct_debit_service_down:
            summary: The direct debit service has been disabled for any reason.
            value:
              detail: "It is not possible to communicate with the bank, please use other payment methods or try again in a few minutes."
              code: "internal_server_error"
          direct_debit_not_allowed:
            summary: It is not possible to pay by direct debit, such as when the portal is inactive or cannot be used for some reason.
            value:
              detail: "Selected gateway is not permitted for user."
              code: "error"
```

<h2 id="get-balance">Get user balance with DirectPay</h2>

To call this endpoint, you need the user's DirectPay token, which is a wallet type, and in the output, you will receive the amount that can be paid with this token until the end of the current period and the user's balance.

<h3 id="get-balance-sample">Sample request</h3>

```yaml
openapi: 3.1.0
info:
  title: Get User Balance And Payable Amount from Directpay Contract API
  version: 1.0.0
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
paths:
  /direct-pay/contract/payable-amount/:
    post:
      summary: get-balance
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
                  description: "Contract token you got from Init Contract"
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  payable_amount:
                    type: integer
                    example: 120000
                    description: "Amount (rials) payable with this token until the end of the current period"
                  payable_amount_string:
                    type: string
                    example: "12,000 TMN"
                    description: "Amount (Toman) allowed to be paid for the current period in Persian"
                  user_balance:
                    type: integer
                    example: 220000
                    description: "Contract user balance in Rials"
                  user_balance_string:
                    type: string
                    example: "22,000 TMN"
                    description: "Contract user balance in Toman and Persian"
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

<h3 id="get-balance-sample">Sample curl</h3>

```curl
curl --location 'https://api.bazaar-pay.ir/badje/v1/direct-pay/contract/payable-amount/' \
--header 'Authorization: Token {merchant_token}' \
--header 'Content-Type: application/json' \
--data '{
    "contract_token": "7e3257a7-cfec-4f75-b80b-443a57a255f4"
}'
```

<h3 id="get-balance-sample-success-response">Sample successful response</h3>

```json
{
	"payable_amount": 120000,
	"payable_amount_string": "12,000 TMN",
	"user_balance": 220000,
	"user_balance_string": "22,000 TMN"
}
```
