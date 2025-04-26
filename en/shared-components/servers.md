servers:
  - url: 'https://{base_url}{base_path_v1}'
    description: BazaarPay API v1
    variables:
      base_url:
        default: 'api.bazaar-pay.ir'
        description: BazaarPay base URL
      base_path:
        default: '/badje/v1'
        description: BazaarPay base PATH in version 1
    example: 'https://api.bazaar-pay.ir/badje/v1/trace/'
  - url: 'https://{base_url}{base_path_v2}'
    description: BazaarPay API v2
    variables:
      base_url:
        default: 'api.bazaar-pay.ir'
        description: BazaarPay base URL
      base_path:
        default: '/badje/v2'
        description: BazaarPay base PATH in version 2
    example: 'https://api.bazaar-pay.ir/badje/v2/trace/'
  - url: 'https://{base_url}{base_path}'
    description: BazaarPay Web
    variables:
      base_url:
        default: 'cafebazaar.ir'
        description: BazaarPay web base URL
      base_path:
        default: '/bazaar-pay'
        description: BazaarPay we base PATH
    example: 'https://cafebazaar.ir/bazaar-pay/contract/direct-pay'