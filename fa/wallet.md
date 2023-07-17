# کیف پول

برای یکپارچه کردن کیف پول بازارپی و اپلیکیشن مرچنت

## دریافت موجودی کاربر

این قابلیت نیازمند دارا بودن یک دسترسی ویژه می‌باشد، در صورت نیاز، برای دریافت آن با پشتیبانی بازارپی در ارتباط باشید.

```yaml
paths:
  { base_url }/get-balance/
    get:
      security: ApiKeyAuth
      summary: get-user-balance
      parameters:
        - name: user_phone_number
          in: query
          description: شماره تماس کاربر
          schema:
            type: string
            example: "09120000000"
          required: true
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
curl --location 'pardakht.cafebazaar.ir/pardakht/badje/v1/get-balance/?user_phone_number=09120000000' \
  --header 'Authorization: Token merchant_token'
```

## افزایش موجودی کاربر

برای افزایش موجودی کیف پول بازارپی، کاربر را به آدرس زیر هدایت کنید.

```yaml
servers:
  - url: https://cafebazaar.ir/bazaar-pay/
paths:
  /increase-balance:
    get:
      summary: increase-balance
      parameters:
        - name: redirect_url
          in: query
          description: کاربر به آدرس مورد نظر باید منتقل شود، همچنین آدرس باید به صورت encodeURIComponent انکد شود
          schema:
            type: string
            example: "https://cafebazaar.ir/bazaar-pay/increase-balance?redirect_url=https://bazaar-pay.ir"
```

