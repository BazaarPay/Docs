```yaml
responses:
  '401':
    description: Unauthorized
    content:
      application/json:
        schema:
          type: object
          description: هدر احراز هویت نامعتبر یا ارسال نشده باشد
          properties:
            detail:
              type: string
              example: "اطلاعات برای اعتبارسنجی ارسال نشده است."
  '403':
    description: Permission Denied
    content:
      application/json:
        schema:
          type: object
          description: دسترسی لازم برای دسترسی به اندپوینت مورد نظر را نداشته باشید یا حساب کاربری شما مسدود شده باشد
          properties:
            detail:
              type: string
              example: "شما دسترسی لازم را ندارید."
  '400':
    description: Bad Request
    content:
      application/json:
        schema:
          type: object
          properties:
            oneOf:
              anyOf:
                field_with_problem:
                  type: array
                  description: فیلد دارای خطا، به صورت یک آبجکت در رسپانس مربوط به ریکوئست برگردانده می‌شود که نام این آیجکت متناسب با فیلد دارای مشکل تغییر می‌کند.
                  items:
                    type: string
              detail:
                oneOf:
                  - type: array
                    items:
                      type: string
                  - type: string
            examples:
              null_in_request_data:
                value:
                  description: یک فیلد اجباری در ریکوئست ارسال نشود یا null ارسال شود
                  json_object_name: [ "این فیلد نمی‌تواند خالی باشد." ]
              invalid_request_schema:
                value:
                  description: اسکیما (ساختار) ارسال شده در بادی ریکوئست معتبر نباشد
                  detail: "JSON parse error - Extra data: line 4 column 2 (char 57)"
              invalid_request_data:
                value:
                  description: مقدار یک فیلد معتبر نباشد (جزو مقادیر مجاز قابل انتخاب نباشد)
                  json_object_name: [ "\"test\" یک انتخاب معتبر نیست." ]
  '503':
    description: Service Temporarily Unavailable. This response is returned when the server is currently unable to handle the request due to temporary overloading or maintenance.
    content:
      text/html:
        schema:
          type: string
          description: سرویس از سمت سرور دچار اختلال باشد
          example: |
            <html>
            <head><title>503 Service Temporarily Unavailable</title></head>
            <body>
            <center><h1>503 Service Temporarily Unavailable</h1></center>
            <hr><center>nginx</center>
            </body>
            </html>
```