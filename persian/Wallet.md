## Wallet

برای یکپارچه کردن کیف پول بازارپی و اپلیکیشن مرچنت

#### Get User Balance:

برای استفاده از این API، ما باید دسترسی جداگانه‌ای را به شما بدهیم. در صورت نیاز، از ما بخواهید.

```yaml
paths:
  { base_url }/get-balance/
    get:
      security: ApiKeyAuth
      summary: get-user-balance
      parameters:
        - name: user_phone_number
          in: query
          description: user's phone_number / شماره تماس کاربر
          schema:
            type: string
            example: "09999999999"
          required: true
        - name: lang
          in: query
          description: response language
          schema:
            type: string
            enum: [ en, fa ]    # default is en
            example: "fa"
      responses:
        '200':
          content:
            application/json:
              schema:
                balance:
                  type: int
                  example: 50000
                balance_string:
                  type: string
                  example: "۵۰۰۰ تومان"
```

example:

```bash
curl --location 'pardakht.cafebazaar.ir/pardakht/badje/v1/get-balance/?lang=en&user_phone_number=09120000000' \
  --header 'Authorization: Token some_auth_token'
```

#### Increase User Balance

برای افزایش موجودی کیف پول بازارپی، کاربر را به آدرس زیر هدایت کنید.

```yaml
servers:
  - url: https://cafebazaar.ir/bazaar-pay/

paths:
  { servers/url }/increase-balance
    get:
      summary: increase-balance
      parameters:
        - name: redirect_url
          in: query
          description: user will be redirected to this address after the process. should be encoded with encodeURIComponent
          schema:
            type: string
            example: "https%3A%2F%2Fexample.com"
```

