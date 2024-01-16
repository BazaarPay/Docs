```yaml
responses:
  '401':
    description: Unauthorized
    content:
      application/json:
        schema:
          type: object
          description: هدر احراز هویت نامعتبر یا ارسال نشده باشد
          oneOf:
            - "#/components/schemas/ErrorResponseV1"
            - "#/components/schemas/ErrorResponseV2"
        examples:
          response_v1:
            summary: "پاسخ در v1"
            value:
              detail: "اطلاعات برای اعتبارسنجی ارسال نشده است."
          response_v2:
            summary: "پاسخ در v2"
            value:
              detail: "اطلاعات برای اعتبارسنجی ارسال نشده است."
              code: "not_authenticated"
  '403':
    description: Permission Denied
    content:
      application/json:
        schema:
          description: دسترسی لازم برای دسترسی به اندپوینت مورد نظر را نداشته باشید یا حساب کاربری شما مسدود شده باشد
          oneOf:
            - "#/components/schemas/ErrorResponseV1"
            - "#/components/schemas/ErrorResponseV2"
        examples:
          response_v1:
            summary: "پاسخ در v1"
            value:
              detail: "شما دسترسی لازم را ندارید."
          response_v2:
            summary: "پاسخ در v2"
            value:
              detail: "شما دسترسی لازم را ندارید."
              code: "permission_denied"
  '400':
    description: Bad Request
    content:
      application/json:
        schema:
          oneOf:
            - "#/components/schemas/BadRequestResponseV1"
            - "#/components/schemas/BadRequestResponseV2"
  '500':
    description: Internal Server Error
    content:
      application/json:
        schema:
          description: خطای داخلی رخ داده
          oneOf:
            - "#/components/schemas/ErrorResponseV1"
            - "#/components/schemas/ErrorResponseV2"
        examples:
          response_v1:
            summary: "پاسخ در v1"
            value:
              detail: "خطایی سمت سرور رخ داده‌است"
          response_v2:
            summary: "پاسخ در v2"
            value:
              detail: "خطایی سمت سرور رخ داده‌است"
              code: "internal_server_error"
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
components:
  schemas:
    ErrorResponseV1:
      type: object
      properties:
        detail:
          type: string
    ErrorResponseV2:
      type: object
      properties:
        detail:
          type: string
        code:
          type: string
    BadRequestResponseV1:
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
                  summary: یک فیلد اجباری در ریکوئست ارسال نشود یا null ارسال شود
                  value:
                    json_object_name: [ "این فیلد نمی‌تواند خالی باشد." ]
                invalid_request_schema:
                  summary: اسکیما (ساختار) ارسال شده در بادی ریکوئست معتبر نباشد
                  value:
                    detail: "JSON parse error - Extra data: line 4 column 2 (char 57)"
                invalid_request_data:
                  summary: مقدار یک فیلد معتبر نباشد (جزو مقادیر مجاز قابل انتخاب نباشد)
                  value:
                    json_object_name: [ "\"test\" یک انتخاب معتبر نیست." ]
    BadRequestResponseV2:
      description: Bad Request in v2 APIs
      content:
        application/json:
          schema:
            type: object
            properties:
              code:
                type: string
                example: "invalid"
              detail:
                type: string
                example: "درخواست نامعتبر است"
              invalid_params:
                type: array
                description: لیست فیلدهای حاوی ارور
                items:
                  type: object
                  properties:
                    param:
                      type: string
                      example: "password"
                    code:
                      type: string
                      example: "required"
                    detail:
                      type: string
                      example: "این مقدار لازم است."
```
