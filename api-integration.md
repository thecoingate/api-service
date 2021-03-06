# thecoingate.com API integration

## Environments
| Name      | API Endpoint | 
| ----------- | ----------- |
| production      | https://api.thecoingate.com/v1/public-api|
| staging      | https://api-staging.thecoingate.com/v1/public-api|



## How to generate HMAC in the request header

The HMAC would be generated by the request body and your private key in SHA512


### Example: Get supported coins

Headers

>  HMAC: c03901975d6dd4cb480ab94453a62a35bd4b44771dfb95a6ce5da789bde3f8391e823281941a7f035c9aa16f07631ca5a55cba64a48ce4db5475234a6b15a37d
>
> Content-Type: application/x-www-form-urlencoded
>
 Payload:

>`version=1.0&key=dc8b7b4b23cf4151e2dc00ff014ca2be5831447f`


## Api integration
### 1. Supported coins
URI: `{{uri}}/supported-coins`

Headers

>  HMAC: `YOUR HMAC`
>
> Content-Type: application/x-www-form-urlencoded
>
Payload:

| Field name      | Description | Require |
| -------- | ----------- |----------- |
| version | The API version | true
| key | Your api public key | true



Reponse: 


```json 
[
  {
    "shortCode": "string",
    "fullCode": "string",
    "assetToken": "string",
    "logo": "string"
  }
]
```

### 2. Balance
URI: `{{uri}}/balance`

Headers

>  HMAC: `YOUR HMAC`
>
> Content-Type: application/x-www-form-urlencoded
>
 Payload:


| Field name      | Description | Require |
| -------- | ----------- |----------- |
| version | The API version | true
| key | Your API public key | true


Reponse: 


```json 
[
  {
    "balance": 0,
    "balancef": "string",
    "code": "string",
    "tokenType": "string",
    "assetBalances": [
      {
        "assetToken": "string",
        "balance": "string",
        "code": "string",
        "tokenType": "string"
      }
    ]
  }
]
```

### 3. Get deposit address
URI: `{{uri}}/deposit-address`

Headers

>  HMAC: `YOUR HMAC`
>
> Content-Type: application/x-www-form-urlencoded
>
 Payload:


| Field name      | Description | Require |
| -------- | ----------- |----------- |
| version | The API version | yes
| key | Your API public key | yes
| identifier | This is unique key helps you to save the deposit address. To avoid re-generating a new deposit address, we recommend that you provide us your user's identifier. Then you have got 1 user = 1 deposit address  | yes
| ipnUrl | The callback URL that you want to receive the instant payment notifications | no
|label | The label for the address | no



Reponse: 


```json 
{
    "address": "string", // Hex in base58 format
    "hexAddress": "string", // Hex address
    "identifer": "string" // User's indentifier
}
```

### 4. Withdrawal
URI: `{{uri}}/withdraw`

Headers

>  HMAC: `YOUR HMAC`
>
> Content-Type: application/x-www-form-urlencoded
>
 Payload:


| Field name      | Description | Require |
| -------- | ----------- |----------- |
| version | The API version | yes
| key | Your API public key | yes
|address | The base 58 address that you want to withdraw to | yes
|currency | The currency that you want to withdraw `TRX, USDT.TRC20 ... `| yes
|tokenType | The type of token. Currently, there are 2 kinds of type: `TRC10, TRC20` | yes
|amount | The amount that you want to withdraw | yes
|autoComfirm | Default is -1. When you withdraw, the system will you a confirmation email to your merchant's email address. You must confirm the transaction before withdrawing. Unless it will be confirmed automatically | no
| ipnUrl | The callback URL that you want to receive the instant payment notifications | no
|label | The label for the address | no

### 5. IPN
After deposit or withdrawal, Thecoingate system will send an POST request to your ipn url as follow;
Headers

>  HMAC: `HMAC`
>
> Content-Type: application/x-www-form-urlencoded
>

Payload:

| Field name      | Description
| -------- | ----------- 
| address | Wallet address (base58 format)
| amount | Amount
| merchant | Merchant id
| label | The label for the address
| txId | Transaction id
| type | value is "deposit" or "withdrawal"
| id | ipn id
| fee | Fee
| feef | feef
| currency | Currency. For example: TRX.TRC10, USDT.TRC20,...

HMAC is calculated with stringied data in the payload encrypted (using SHA512) with your ipn secret key.

Stringified data exmaple: 
address=TG2QgQadkvXgiyoJWQcrvxRmfS4Naq9s2Z&amount=2%2C999.00000000&currency=USDT.TRC20&merchant=decda395-c3a4-4808-9032-0c6a6155e00b&label=&txId=8f18d00095f34c8ada2ae107c2ed9e474ac51bf8ca0c66489c6fe698e138f79d&type=deposit&id=2f89c0e0-6993-4da6-99d1-31d6746ef555&fee=0&feef=0.000000

Response: Your ipn callback must return status 2xx for success. Otherwise failed.
Note: Thecoingate will retry 10 times on fail. 
