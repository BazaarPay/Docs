```yaml
servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
    variables:
      base_url:
        default: 'pardakht.cafebazaar.ir'
        description: آدرس پایه اندپوینت‌های بازارپی
      base_path:
        default: '/pardakht/badje/v1'
        description: مسیر پایه اندپوینت‌های بازارپی نسخه ۱
    example: 'https://pardakht.cafebazaar.ir/pardakht/badje/v1/trace/'
  - url: 'https://{base_url}{base_path_v2}'
    description: BazaarPay API v2
    variables:
      base_url:
        default: 'pardakht.cafebazaar.ir'
        description: آدرس پایه اندپوینت‌های بازارپی
      base_path:
        default: '/pardakht/badje/v2'
        description: مسیر پایه اندپوینت‌های بازارپی نسخه ۲
    example: 'https://pardakht.cafebazaar.ir/pardakht/badje/v1/trace/'
  - url: 'https://{base_url}{base_path}'
    description: BazaarPay Web
    variables:
      base_url:
        default: 'cafebazaar.ir'
        description: آدرس پایه وب بازارپی
      base_path:
        default: '/bazaar-pay'
        description: مسیر پایه وب بازارپی
    example: 'https://cafebazaar.ir/bazaar-pay/contract/direct-pay'
```