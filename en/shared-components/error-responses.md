```yaml
responses:
  '401':
    description: Unauthorized
    content:
      application/json:
        schema:
          type: object
          description: The authentication header is invalid or not sent.
          oneOf:
            - "#/components/schemas/ErrorResponseV1"
            - "#/components/schemas/ErrorResponseV2"
        examples:
          response_v1:
            summary: "Response in v1"
            value:
              detail: "Information not submitted for validation."
          response_v2:
            summary: "Response in v2"
            value:
              detail: "Information not submitted for validation."
              code: "not_authenticated"
  '403':
    description: Permission Denied
    content:
      application/json:
        schema:
          type: object
          description: You do not have the necessary access to access the desired endpoint or your account has been blocked.
          oneOf:
            - "#/components/schemas/ErrorResponseV1"
            - "#/components/schemas/ErrorResponseV2"
        examples:
          response_v1:
            summary: "Response in v1"
            value:
              detail: "You do not have the necessary access.."
          response_v2:
            summary: "Response in v2"
            value:
              detail: "You do not have the necessary access.."
              code: "permission_denied"
  '400':
    description: Bad Request
    content:
      application/json:
        schema:
          type: object
          oneOf:
            - "#/components/schemas/BadRequestResponseV1"
            - "#/components/schemas/BadRequestResponseV2"
  '500':
    description: Internal Server Error
    content:
      application/json:
        schema:
          type: object
          description: An internal error occurred
          oneOf:
            - "#/components/schemas/ErrorResponseV1"
            - "#/components/schemas/ErrorResponseV2"
        examples:
          response_v1:
            summary: "Response in v1"
            value:
              detail: "A server-side error has occurred."
          response_v2:
            summary: "Response in v2"
            value:
              detail: "A server-side error has occurred."
              code: "internal_server_error"
  '503':
    description: Service Temporarily Unavailable. This response is returned when the server is currently unable to handle the request due to temporary overloading or maintenance.
    content:
      text/html:
        schema:
          type: string
          description: The service is disrupted on the server side.
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
      properties:
        detail:
          type: string
    ErrorResponseV2:
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
            properties:
              oneOf:
                anyOf:
                  field_with_problem:
                    type: array
                    description: The field with the error is returned as an object in the response to the request, and the name of this object changes according to the field with the problem.
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
                  summary: A required field is not sent in the request or null is sent.
                  value:
                    json_object_name: [ "This field may not be null." ]
                invalid_request_schema:
                  summary: The schema (structure) sent in the request body is not valid.
                  value:
                    detail: "JSON parse error - Extra data: line 4 column 2 (char 57)"
                invalid_request_data:
                  summary: The value of a field is not valid (not among the allowed values ​​that can be selected)
                  value:
                    json_object_name: [ "\"test\" is not a valid choice." ]
    BadRequestResponseV2:
      description: Bad Request in v2 APIs
      content:
        application/json:
          schema:
            properties:
              code:
                type: string
                example: "invalid"
              detail:
                type: string
                example: "The request is invalid."
              invalid_params:
                type: array
                description: List of fields containing errors
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
                      example: "This field is required."
```
