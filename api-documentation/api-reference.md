# API Reference

This document covers the basic concepts of the payment transaction types and the technical details of the dLocal - Payments API. It contains functional examples of the requests and important observations to be taken into account during integration.

JSON is returned by all API responses, including errors.

For sandbox environment replace api.dlocal.com for sandbox.dlocal.com.

## Security

### Signature

The signature should use SHA256 as HMAC hash function. The signature header always should have the version prefix that contains the signature version and the used hash function. For now, V2-HMAC-SHA256.

### Headers

| **Header** | **Type** | **Description** |
| --- | --- | --- | --- | --- |
| `X-Date` | String | ISO8601 Datetime with Timezone |
| `X-Login` | String | Merchant xLogin |
| `X-Trans-Key` | String  | Merchant xTransKey |
| Authorization | String | &lt;auth version&gt;, Signature: &lt;hmac\(secretKey, "X-Login+X-Date Header+RequestBody"\)&gt; |

### Sensitive data encryption

Credit Card data, such as number and cvv, will be encrypted inside the Json Request Body using [JWE](https://tools.ietf.org/html/rfc7516). This standard is being widely used in the market, so most used languages have any libraries to support it, simplifying the integration for our merchants.

The following parameters can be encrypted and added to a new `encrypted_data` field:

{% tabs %}
{% tab title="Properties" %}
| **Property** | **Type** | **Description** |
| --- | --- | --- |
| `cvv` | String | Credit Card security code |
| `number` | String | Credit Card number |
{% endtab %}

{% tab title="Example credit card encrypted body" %}
```yaml
{
    "holder_name": "Thiago Gabriel",
    "expiration_month": 10,
    "expiration_year": 2040,
    "encrypted_data": "eyJlbmMiOiJBMjU2R0NNIiwiYW..."
}
```
{% endtab %}
{% endtabs %}

## Idempotent Requests

To perform an idempotent request, provide an additional 'X-Idempotency-Key' header to the request.

{% tabs %}
{% tab title="Properties" %}
| **Header** | **Type** | **Description** |
| --- | --- |
| `X-Idempotency-KeyString` | String \(Optional\) | Key used for perform an idempotent request. |
{% endtab %}

{% tab title="Example Idempotent Request" %}
```bash
curl -X POST \
    -H 'X-Date: 2018-02-20T15:44:42.310Z' \
    -H 'X-Login: sak223k2wdksdl2' \
    -H 'X-Idempotency-Key: a8a85bce-5733-4a6c-91b5-553ed4b3de16' \
    -H 'Authorization: V2-HMAC-SHA256, Signature: 1bd227f9d892a7f4581b998c21e353b1686a6bdad5940e7bb6aa596c96e0a6ec' \
    -d '{body}'
    https://api.dlocal.com/payments
```
{% endtab %}
{% endtabs %}

## Payment Methods

Object that identifies each payment method accepted by dLocal.

### The Payment Method Object

{% tabs %}
{% tab title="Payment Method Object" %}
| **Property** | **Type** | **Description** |
| --- | --- | --- | --- | --- | --- | --- |
| `id` | String | Payment method id |
| `type` | String | Type of method can be `CARD` `BANK_TRANSFER` `DIRECT_DEBIT` `TICKET`. |
| `name` | String | Payment type name |
| `countries` | String\[\] | Countries where the payment method is available. |
| `logo` | String | Payment method image url. |
| `allowed_flows` | String\[\] | Flows allowed for this payment, can be `DIRECT` or`REDIRECT`. |
{% endtab %}

{% tab title="Example Payment Method Object" %}
```yaml
{
  "id": "OX",
  "type": "TICKET",
  "name": "Oxxo",
  "countries" : ["MX"],
  "logo": "http://static.dlocal.com/payments-methods/OX/60x60",
  "allowed_flows": ["DIRECT", "REDIRECT"]
}
```
{% endtab %}
{% endtabs %}

{% api-method method="get" host=" https://api.dlocal.com/" path="payments-methods" %}
{% api-method-summary %}
Search Payment Methods
{% endapi-method-summary %}

{% api-method-description %}
This function returns a list of valid payment methods for the requested country.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-headers %}
{% api-method-parameter name="X-Date" type="string" required=true %}
ISO8601 Datetime with TimeZone.
{% endapi-method-parameter %}

{% api-method-parameter name="X-Login" type="string" required=true %}
Merchant xLogin
{% endapi-method-parameter %}

{% api-method-parameter name="X-Trans-Key" type="string" required=false %}
Merchant xTransKey
{% endapi-method-parameter %}

{% api-method-parameter name="Authorization" type="string" required=true %}
&lt;auth version&gt;, Signature: &lt;hmac\(secretKey, "X-Login+X-Date Header+RequestBody"\)&gt;
{% endapi-method-parameter %}
{% endapi-method-headers %}

{% api-method-query-parameters %}
{% api-method-parameter name="country" type="string" required=true %}
Payment method country code. See all country codes here.
{% endapi-method-parameter %}
{% endapi-method-query-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
[
  {
      "id": "VI",
      "type": "CARD",
      "name": "Visa credit card",
      "logo": "http://static.dlocal.com/payments-methods/VI/60x60.png",
      "allowed_flows": ["DIRECT", "REDIRECT"]
  },
  {
      "id": "AE",
      "type": "CARD",
      "name": "American Express",
      "logo": "http://static.dlocal.com/payments-methods/AE/60x60.png",
      "allowed_flows": ["DIRECT", "REDIRECT"]
  },
  {
      "id": "GL",
      "type": "BANK_TRANSFER",
      "name": "Banco Galicia",
      "logo": "http://static.dlocal.com/payments-methods/BT/60x60.png",
      "allowed_flows": ["REDIRECT"]
  },
  {
      "id": "RP",
      "type": "TICKET",
      "name": "Rapi pago",
      "logo": "http://static.dlocal.com/payments-methods/RP/60x60.png",
      "allowed_flows": ["REDIRECT"]
  }
]
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

### Example Request

```bash
curl -X GET \
    -H 'X-Date: 2018-02-20T15:44:42.310Z' \
    -H 'X-Login: sak223k2wdksdl2' \
    -H 'Authorization: V2-HMAC-SHA256, Signature: 1bd227f9d892a7f4581b998c21e353b1686a6bdad5940e7bb6aa596c96e0a6ec' \
    https://api.dlocal.com/payments-methods?country=AR
```

## Payments

This service allows you to create, modify or read payments.

### The Payment Object

{% tabs %}
{% tab title="Payment Object" %}
| **Property** | **Type** | **Description** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `id` | String | Id of payment |
| `amount` | Positive Float | Transaction amount \(in the currency entered in the field “currency”\). |
| `currency` | String | Three-letter [ISO currency code](https://www.iso.org/iso-4217-currency-codes.html), in uppercase. |
| `payment_method_id` | String | Payment method id of the payment method chosen.payment\_method\_id. |
| `payment_method_type` | String | Payment method type of the payment method chosen. |
| `payment_method_flow` | String | Payment method flow of the payment method chosen, can be `DIRECT` or `REDIRECT`. |
| `country` | String | User’s country code. ISO 3166-1 alpha-2 codes. |
| `payer` $$ $$ | [Payer Object](api-reference.md#the-payer-object) | Identifies the payer |
| `card` | [Card Object](api-reference.md#the-card-object) | Credit card information \( only for CARD payment methods \). |
| `bank_transfer` | [Bank Transfer Object](api-reference.md#the-bank-transfer-object) | Bank transfer information \( only for BANK\_TRANSFER payment methods \). |
| `direct_debit` | [Direct Debit Object](api-reference.md#the-direct-debit-object) | Bank information for direct debit \( only for DIRECT\_DEBIT payment methods \). |
| `ticket` | [Ticket Object](api-reference.md#the-ticket-object) | Ticket information \( only for TICKET payment methods \). |
| `refunds` | Date\(ISO\_8601\) | Payment's refunds of refund object. |
| `created_date` | Date\(ISO\_8601\) | Payment's creation date. |
| `approved_date` | Date\(ISO\_8601\) | Payment's approval date. |
| `status` | String | Payment status. |
| `status_detail` | String | Payment status detail. |
| `reject_code` | Integer | Rejection status code. |
| `external_reference` | String | ID given by the merchant in their system. |
| `description` | String | Payment description |
| `notification_url` | String | URL where dlocal will send notifications associated to changes in this payment. |
| `callback_url` | String | URL where dlocal does the final redirect \(only for bank transfers and tickets\). |
| `redirect_url` | String | URL where the merchant must redirect the user to complete the payment. |
{% endtab %}

{% tab title="Example Payment Object" %}
```yaml
{
    "id": "3436735432",
    "amount": 120.00,
    "currency" : "USD",
    "country": "BR",
    "payment_method_id" : "VI",
    "payment_method_type" : "CARD",
    "payment_method_flow" : "DIRECT",
    "payer":{
        "name" : "Thiago Gabriel",
        "email" : "thiago@example.com",
        "document" : "53033315550",
        "document_type" : "CPF",
        "user_reference": "12345",
        "address": {
            "country" : "BR",
            "state"  : "Rio de Janeiro",
            "city" : "Volta Redonda",
            "zip_code" : "27275-595",
            "street" : "Servidão B-1",
            "number" : "1106"
        }
    },
    "card":{
        "holder_name": "Thiago Gabriel",
        "expiration_month": 10,
        "expiration_year": 2040,
        "last4": "4544",
        "brand": "VI",
        "capture": true
    },
    "refunds": [],
    "created_date" : "2018-02-15T15:14:52-00:00",
    "approved_date" : "2018-02-15T15:14:52-00:00",
    "status" : "PAID",
    "status_detail" : "The payment was paid.",
    "external_reference": "657434343",
    "notification_url": "http://merchant.com/notifications"
}
```
{% endtab %}
{% endtabs %}

### The Payer Object

{% tabs %}
{% tab title="Payer Object" %}
| **Property** | **Type** | **Description** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `name` | String | User's full name |
| `email` | String | User’s email address. |
| `birth_date` | String | User’s birthdate \(DD-MM-YYYY\). |
| `phone` | String | User’s phone. |
| `document` | String | User’s personal identification number: CPF or CNPJ for Brazil, DNI for Argentina and ID for other countries. |
| `document_type` | String | User’s document type. |
| `user_reference` | String | Unique user id at the merchant side. |
| `address` | [Address Object](api-reference.md#the-address-object) | User’s address. |
{% endtab %}

{% tab title="Example Payer Object" %}
```yaml
{
"name" : "Thiago Gabriel",
"email" : "thiago@example.com",
"document" : "53033315550",
"document_type" : "CPF",
"user_reference": "12345",
"address": {
    "country" : "BR",
    "state"  : "Rio de Janeiro",
    "city" : "Volta Redonda",
    "zip_code" : "27275-595",
    "street" : "Servidão B-1",
    "number" : "1106"
}
}
```
{% endtab %}
{% endtabs %}

### The Address Object

{% tabs %}
{% tab title="Address Object" %}
| **Property** | **Type** | **Description** |
| --- | --- | --- | --- | --- | --- |
| `state` | String | User's address state. |
| `city` | String | User’s address city. |
| `zip_code` | String | User’s address zip\_code. |
| `street` | String | User’s address street. |
| `number` | String | User’s address number. |
{% endtab %}

{% tab title="Example Address Object" %}
```yaml
{
"country" : "BR",
"state"  : "Rio de Janeiro",
"city" : "Volta Redonda",
"zip_code" : "27275-595",
"street" : "Servidão B-1",
"number" : "1106"
}
```
{% endtab %}
{% endtabs %}

### The Card Object

| **Property** | **Type** | **Description** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `holder_name` | String | Cardholder's full name |
| `expiration_month` | Integer | Two digit number representing the card's expiration month. |
| `expiration_year` | Integer | Four digit number representing the card's expiration year. |
| `token` | String | Credit card token. |
| `number` | String | The card number, as a string without any separators. |
| `cvv` | String | Credit card verification value. |
| `brand` | String | Card brand. |
| `installments` | String | Number of installments. |
| `installments_id` | String | Installments id of a [installments plan](https://dlocal.com/docs/?language=cURL#installments-plan). |
| `descriptor` | String | Dynamic Descriptor. |
| `last4` | String | The last 4 digits of the card. |
| `encrypted_data` | String | [JWE](https://tools.ietf.org/html/rfc7516) encrypted params. |
| `save` | String | Indicate if the card must be save for future payments, can be `YES`, `NO`, `ASK_USER` \( ask user is only for `REDIRECT` payment method flows \) |
| `capture` | Boolean | Whether or not to immediately capture the charge. When false, the charge issues an authorization, and will need to be captured later. |

### The Bank Transfer Object

| **Property** | **Type** | **Description** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `type` | String | Bank account type |
| `name` | String | Bank name. |
| `code` | String | Bank code. |
| `beneficiary` | String | Beneficiary name. |
| `account` | String | Bank account number. |
| `document_type` | String | Beneficiary document type. |
| `document` | String | Beneficiary document number. |
| `amount_to_transfer` | Positive Float | Amount to transfer \(only for not referenced bank transfers\). |
| `reference` | String | Reference to make the bank transfer. |

### The Direct Debit Object

| **Property** | **Type** | **Description** |
| --- | --- | --- | --- | --- | --- |
| `holder_name` | String | Name of the owner of the |
| `email` | String | Email of the owner of the bank account. |
| `document_type` | String | Document of the owner of the bank account. |
| `document` | String | Document of the owner of the bank account. |
| `cbu` | String | CBU of the owner of the bank account \(only for AR country\). |

### The Ticket Object

| **Property** | **Type** | **Description** |
| --- | --- | --- | --- | --- |
| `type` | String | Type of ticket, can be `REFERENCE_CODE`, or `BAR_CODE` |
| `format` | String | Ticket barcode format only for `BAR_CODE`, for example `CODE_128`. |
| `number` | String | Ticket number. |
| `expiration_date` | Date\(ISO-8601\) | The expiration date of the ticket. |

{% api-method method="post" host="https://api.dlocal.com" path="/payments" %}
{% api-method-summary %}
Create a Payment
{% endapi-method-summary %}

{% api-method-description %}
Creates a new payment.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="amount" type="string" required=true %}
Transaction amount \(in the currency entered in the field "currency"\)
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}

{% api-method-body-parameters %}
{% api-method-parameter name="amount" type="string" required=true %}
Transaction amount \(in the currency entered in the field `currency`\)
{% endapi-method-parameter %}

{% api-method-parameter name="currency" type="string" required=true %}
Three-letter ISO currency code, in uppercase.
{% endapi-method-parameter %}

{% api-method-parameter name="payment\_method\_id" type="string" required=true %}
Payment method id chosen to make the payment.
{% endapi-method-parameter %}

{% api-method-parameter name="payment\_method\_type" type="string" required=true %}
Payment method type chosen to make the payment.
{% endapi-method-parameter %}

{% api-method-parameter name="payment\_method\_flow" type="string" required=true %}
Payment method flow, can be `DIRECT` or `REDIRECT`
{% endapi-method-parameter %}

{% api-method-parameter name="country" type="string" required=true %}
User's country code. ISO 3166-1 alpha-2 code.
{% endapi-method-parameter %}

{% api-method-parameter name="payer" type="object" required=true %}
Payer Object
{% endapi-method-parameter %}

{% api-method-parameter name="card" type="object" required=true %}
Card Object **\(required only for** `CARD` **payment type\)**
{% endapi-method-parameter %}

{% api-method-parameter name="direct\_debit" type="object" required=true %}
Direct Debit Object **\(required only for** `DIRECT_DEBIT` **payment type\)**
{% endapi-method-parameter %}

{% api-method-parameter name="external\_reference" type="string" required=true %}
ID given by the merchant in their system.
{% endapi-method-parameter %}

{% api-method-parameter name="description" type="string" required=false %}
Payment description.
{% endapi-method-parameter %}

{% api-method-parameter name="notification\_url" type="string" required=true %}
URL where dlocal will send notifications associated to changes to this payment.
{% endapi-method-parameter %}

{% api-method-parameter name="callback\_url" type="string" required=false %}
URL where dlocal does the final redirect \(only for bank transfers and tickets\)
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}
Example Response
{% endapi-method-response-example-description %}

```yaml
{
    "id": "PAY2323243343543",
    "amount": 120.00,
    "currency" : "USD",
    "country": "BR",
    "payment_method_type" : "CARD",
    "payment_method_flow" : "DIRECT",
    "payer":{
        "name" : "Thiago Gabriel",
        "email" : "thiago@example.com",
        "document" : "53033315550",
        "document_type" : "CPF",
        "user_reference": "12345",
        "address": {
            "country" : "BR",
            "state"  : "Rio de Janeiro",
            "city" : "Volta Redonda",
            "zip_code" : "27275-595",
            "street" : "Servidão B-1",
            "number" : "1106"
        }
    },
    "card":{
        "token": "CV-e90078f7-e027-4ce4-84cb-534c877be33c",
        "holder_name": "Thiago Gabriel",
        "expiration_month": 10,
        "expiration_year": 2040,
        "last4": "1111",
        "brand": "VI",
        "capture": false,
        "save" : "YES"
    },
    "created_date" : "2018-02-15T15:14:52-00:00",
    "approved_date" : "2018-02-15T15:14:52-00:00",
    "status" : "PAID",
    "status_detail" : "The payment was paid.",
    "external_reference": "657434343",
    "notification_url": "http://merchant.com/notifications"
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

### The Payer Object

{% tabs %}
{% tab title="Payer Object" %}
| **Property** | **Type** | **Description** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `name` | String **\(Required\)** | User's full name |
| `email` | String **\(Required\)** | User’s email address. |
| `birth_date` | String \(Optional\) | User’s birthdate \(DD-MM-YYYY\). |
| `phone` | String \(Optional\) | User’s phone. |
| `document` | String \(Optional\) | User’s personal identification number: CPF or CNPJ for Brazil, DNI for Argentina and ID for other countries. |
| `document_type` | String \(Optional\) | User’s document type. |
| `user_reference` | String \(Optional\) | Unique user id at the merchant side. |
| `address` | [Address Object ](api-reference.md#the-address-object)\(Optional\) | User’s address. |
{% endtab %}

{% tab title="Example Payer Object" %}
```yaml
{
"name" : "Thiago Gabriel",
"email" : "thiago@example.com",
"document" : "53033315550",
"document_type" : "CPF",
"user_reference": "12345",
"address": {
    "country" : "BR",
    "state"  : "Rio de Janeiro",
    "city" : "Volta Redonda",
    "zip_code" : "27275-595",
    "street" : "Servidão B-1",
    "number" : "1106"
}
}
```
{% endtab %}
{% endtabs %}

### The Address Object

{% tabs %}
{% tab title="Address Object" %}
| **Property** | **Type** | **Description** |
| --- | --- | --- | --- | --- | --- |
| `state` | String \(Optional\) | User's address state. |
| `city` | String \(Optional\) | User’s address city. |
| `zip_code` | String \(Optional\) | User’s address zip\_code. |
| `street` | String \(Optional\) | User’s address street. |
| `number` | String \(Optional\) | User’s address number. |
{% endtab %}

{% tab title="Example Address Object" %}
```yaml
{
"country" : "BR",
"state"  : "Rio de Janeiro",
"city" : "Volta Redonda",
"zip_code" : "27275-595",
"street" : "Servidão B-1",
"number" : "1106"
}
```
{% endtab %}
{% endtabs %}

### The Card Object

| **Property** | **Type** | **Description** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `holder_name` | String **\(Required if token not present\)** | Cardholder's full name |
| `expiration_month` | Integer **\(Required if token not present\)** | Two digit number representing the card's expiration month. |
| `expiration_year` | Integer **\(Required if token not present\)** | Four digit number representing the card's expiration year. |
| `token` | String \(Optional\) | Credit card token. |
| `number` | String **\(Required if encrypted\_data or token is not present\)** | The card number, as a string without any separators. |
| `cvv` | String **\(Required if encrypted\_data or token is not present\)** | Credit card verification value. |
| `installments` | String \(Optional\) | Number of installments. Default 1. |
| `installments_id` | String \(Optional\) | Installments id of a installments plan. |
| `descriptor` | String \(Optional\) | Dynamic Descriptor |
| `encrypted_data` | String \(Optional\) | [JWE](https://tools.ietf.org/html/rfc7516) encrypted params. |
| `save` | String \(Optional\) | Indicate if the card must be save for future payments, can be `YES`, `NO`, `ASK_USER` \( ask user is only for `REDIRECT` payment methods flows \). Default `NO` |
| `capture` | Boolean \(Optional\) | Whether or not to immediately capture the charge. When false, the charge issues an authorization, and will need to be captured later. Default `TRUE` |

### The Direct Debit Object

| **Property** | **Type** | **Description** |
| --- | --- | --- | --- | --- | --- |
| `holder_name` | String \(Required\) | Name of the owner of the |
| `email` | String \(Required\) | Email of the owner of the bank account. |
| `document_type` | String \(Required\) | Document of the owner of the bank account. |
| `document` | String \(Required\) | Document of the owner of the bank account. |
| `cbu` | String \(Required\) | CBU of the owner of the bank account \(only for AR country\). |

### Example Request

```bash
curl -X POST \
    -H 'X-Date: 2018-02-20T15:44:42.310Z' \
    -H 'X-Login: sak223k2wdksdl2' \
    -H 'Authorization: V2-HMAC-SHA256, Signature: 1bd227f9d892a7f4581b998c21e353b1686a6bdad5940e7bb6aa596c96e0a6ec' \
    -d '{body}'
    https://api.dlocal.com/payments
```

### Example Request Body

```yaml
{
    "amount": 120.00,
    "currency" : "USD",
    "country": "BR",
    "payment_method_type" : "CARD",
    "payment_method_flow" : "DIRECT",
    "payer":{
        "name" : "Thiago Gabriel",
        "email" : "thiago@example.com",
        "document" : "53033315550",
        "document_type" : "CPF",
        "user_reference": "12345",
        "address": {
            "country" : "BR",
            "state"  : "Rio de Janeiro",
            "city" : "Volta Redonda",
            "zip_code" : "27275-595",
            "street" : "Servidão B-1",
            "number" : "1106"
        }
    },
    "card":{
        "holder_name": "Thiago Gabriel",
        "expiration_month": 10,
        "expiration_year": 2040,
        "number": "41111111111111",
        "cvv": "123",
        "capture": false,
        "save" : "YES"
    },
    "external_reference": "657434343",
    "notification_url": "http://merchant.com/notifications"
}
```

{% api-method method="get" host=" https://api.dlocal.com/payments/" path="{payment\_id}" %}
{% api-method-summary %}
Retrieve a Payment
{% endapi-method-summary %}

{% api-method-description %}
Retrieve information about an existing payment.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="payment\_id" type="string" required=true %}
The payment id
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```yaml
{
    "id": "PAY4334346343",
    "amount": 150.00,
    "currency" : "ARS",
    "country": "AR",
    "payment_method_id" : "GL",
    "payment_method_type" : "BANK_TRANSFER",
    "payment_method_flow" : "DIRECT",
    "payer":{
        "name" : "Juan Gomez",
        "email" : "juan@example.com",
        "document" : "58473832",
        "document_type" : "CUIT",
        "address": {
            "country" : "AR",
            "state"  : "Buenos Aires",
            "city" : "Buenos Aires",
            "zip_code" : "272235-595",
            "street" : "Gobernador",
            "number" : "5433"
        }
    },
    "bank_transfer":{
        "type": "CURRENT_ACCOUNT",
        "name": "Banco de Galicia",
        "code": "GL",
        "beneficiary": "ARS CAPITAL SA",
        "document": "4234234243",
        "document_type": "CUIT",
        "cbu": "00700415-20000021948168",
        "reference": "43423245",
        "amount_to_transfer": 150.01
    },
    "created_date" : "2018-02-15T15:44:42.310Z",
    "approved_date" : "2018-02-15T15:44:42.310Z",
    "status" : "PENDING",
    "status_detail" : "The payment is pending.",
    "status_code" : 100,
    "external_reference": "657434343",
    "notification_url": "http://merchant.com/notifications"
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

### Example Request

```bash
$ curl \
    -H 'X-Date: 2018-02-20T15:44:42.310Z' \
    -H 'X-Login: sak223k2wdksdl2' \
    -H 'Authorization: V2-HMAC-SHA256, Signature: 1bd227f9d892a7f4581b998c21e353b1686a6bdad5940e7bb6aa596c96e0a6ec' \
    https://api.dlocal.com/payments/PAY4334346343
```

{% api-method method="get" host="https://api.dlocal.com/payments/" path="{payment\_id}/status" %}
{% api-method-summary %}
Retrieve a Payment Status
{% endapi-method-summary %}

{% api-method-description %}
Retrieve information about the status of an existing payment.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="payment\_id" type="string" required=true %}
The payment id
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}
Example Response
{% endapi-method-response-example-description %}

```yaml
{
    "id": "PAY4334346343",
    "status" : "PENDING",
    "status_detail" : "The payment is pending."
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

### Example Request

```bash
$ curl \
    -H 'X-Date: 2018-02-20T15:44:42.310Z' \
    -H 'X-Login: sak223k2wdksdl2' \
    -H 'Authorization: V2-HMAC-SHA256, Signature: 1bd227f9d892a7f4581b998c21e353b1686a6bdad5940e7bb6aa596c96e0a6ec' \
    https://api.dlocal.com/payments/PAY4334346343/status
```

### Payment Status Codes

| **Status** | **Status code** | **Description** |
| --- | --- | --- | --- | --- | --- | --- |
| `PENDING` | 100 | The payment is pending. |
| `PAID` | 200 | The payment was paid. |
| `REJECTED` | 300 | The payment was rejected. |
| `CANCELLED` | 400 | The payment was cancelled. |
| `EXPIRED` | 500 | The payment was expired. |
| `AUTHORIZED` | 600 | The payment was authorized. |

### Rejection Status

| **Status** | **Status code** | **Description** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `REJECTED` | 301 | Rejected by bank. |
| `REJECTED` | 302 | Insufficient amount. |
| `REJECTED` | 303 | Card blacklisted. |
| `REJECTED` | 304 | Score validation. |
| `REJECTED` | 305 | Max attempts reached. |
| `REJECTED` | 306 | Call bank for authorize. |
| `REJECTED` | 307 | Duplicated payment. |
| `REJECTED` | 308 | Credit card disabled. |

### Errors

All the errors are returned with appropriate HTTP status code, 4XX or 5XX. The format of all errors is:

| **Property** | **Description** |
| --- | --- | --- | --- |
| `codeinteger` | Error code. |
| `messagestring` | Human readable message. |
| `paramstring` | In case one parameter is wrong. |

**Example error**

```yaml
{
    "code": 5005,
    "message": "User unauthorized due to cadastral situation"
}
```

### Http Errors {#http-errors}

| **HTTP Status Code** | **Error Code** | **Error Detail** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `403 Forbidden` | 3001 | Invalid Credentials. |
|  | 3002 | Unregistered IP address. |
|  | 3003 | Merchant has no authorization to use this API. |
| `404 Not Found` | 4000 | Payment not found. |
| `400 Bad Request` | 5000 | Invalid request. |
|  | 5001 | Invalid param. |
|  | 5002 | Invalid transaction status. |
|  | 5003 | Country not supported. |
|  | 5004 | Currency not allowed for this country. |
|  | 5005 | User unauthorized due to cadastral situation. |
|  | 5006 | User limit exceeded. |
|  | 5007 | Amount exceeded. |
|  | 5008 | Token not found or inactive. |
| `429 Too many requests` | 6000 | Too many requests to the api \(Not implemented yet\). |
| `500 Internal Server Error` | 7000 | The input is correct, but dLocal fails to process the payment. Rare case. |

### Notifications

Notifications will be sent in every change of status of a payment to the notification URL specified by the merchant. This URL is taken from the `notification_url` field of the payment, if it differs from the one specified in the merchant panel. The body of the request will always be the payment object.

### Example Notification POST

POST: _{payment.notification\_url}_

```yaml
{
    "id": "PAY4334346343",
    "amount": 150.00,
    "currency" : "ARS",
    "country": "AR",
    "payment_method_id" : "GL",
    "payment_method_type" : "BANK_TRANSFER",
    "payment_method_flow" : "DIRECT",
    "payer":{
        "name" : "Juan Gomez",
        "email" : "juan@example.com",
        "document" : "58473832",
        "document_type" : "CUIT",
        "address": {
            "country" : "AR",
            "state"  : "Buenos Aires",
            "city" : "Buenos Aires",
            "zip_code" : "272235-595",
            "street" : "Gobernador",
            "number" : "5433"
        }
    },
    "bank_transfer":{
        "type": "CURRENT_ACCOUNT",
        "name": "Banco de Galicia",
        "code": "GL",
        "beneficiary": "ARS CAPITAL SA",
        "document": "4234234243",
        "document_type": "CUIT",
        "cbu": "00700415-20000021948168",
        "reference": "43423245",
        "amount_to_transfer": 150.01
    },
    "created_date" : "2018-02-15T15:14:52-00:00",
    "status" : "PENDING",
    "external_reference": "657434343",
    "notification_url": "http://merchant.com/notifications"
}
```

## Refunds

This service allows you to create and read refunds of an existing payment.

A Refund is the reversal of a credit card [payment](https://dlocal.com/docs/?language=cURL#create-a-payment), where the funds are taken from the Merchant and given back to the Card Holder. A refund processing fee may apply.

{% api-method method="post" host=" https://api.dlocal.com/" path="refunds" %}
{% api-method-summary %}
Make a Refund
{% endapi-method-summary %}

{% api-method-description %}

{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-body-parameters %}
{% api-method-parameter name="id\_payment" type="string" required=true %}
The payment id
{% endapi-method-parameter %}

{% api-method-parameter name="notification\_url" type="string" required=true %}
If a refund is pending, the refund confirmation is sent asynchronously to this URL.
{% endapi-method-parameter %}

{% api-method-parameter name="amount" type="string" required=false %}
Amount to refund. Default is total amount of the payment.
{% endapi-method-parameter %}

{% api-method-parameter name="currency" type="string" required=true %}
Currency of the Amount. **Only required if** `amount` **is present.**
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```yaml
{
    "id": "REF42342",
    "id_payment": "PAY4334346343",
    "amount": 100.00,
    "amount_refunded": 100.00,
    "currency": "USD",
    "status": "SUCCESS",
    "status_detail": "The refund was paid.",
    "notification_url": "http://some.url",
    "created_date" : "2018-02-15T15:14:52-00:00"
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

### Example Request

```bash
$ curl -X POST \
    -H 'X-Date: 2018-02-20T15:44:42.310Z' \
    -H 'X-Login: sak223k2wdksdl2' \
    -H 'Authorization: V2-HMAC-SHA256, Signature: 1bd227f9d892a7f4581b998c21e353b1686a6bdad5940e7bb6aa596c96e0a6ec' \
    https://api.dlocal.com/refunds
```

### Example Request Body

```yaml
{
    "id_payment" : "PAY4334346343",
    "amount": 100.00,
    "currency": "USD",
    "notification_url": "http://some.url"
}
```

### Refund Asynchronous Notification

If a refund is pending, the refund confirmation is sent asynchronously to the refund notification URL by POST, sending the following parameters:

| Property | Type | Description |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `id` | String | The refund id. |
| `payment_id` | String | The payment id. |
| `amount` | Positive Float | The amount of the refund. |
| `amount_refunded` | Positive Float | The refunded amount. |
| `currency` | String | The currency code of the refund. |
| `status` | String | The status of the refund. |
| `status_code` | Integer | The [status](https://dlocal.com/docs/?language=cURL#refund-status) code of the refund. |
| `created_date` | String | The date of when the refund was executed. |

### Example POST

POST**:** _{refund.notification\_url}_

```yaml
{
    "id": "REF42342",
    "payment_id": "PAY245235",
    "amount": 100.00,
    "amount_refunded": 100.00,
    "currency": "USD",
    "status": "SUCCESS",
    "status_code": 200,
    "created_date" : "2018-02-15T15:14:52-00:00"
}
```

### Refund Status Codes {#refund-status}

| Status | Status code | Description |
| --- | --- | --- | --- | --- |
| `PENDING` | 100 | The refund is pending. |
| `SUCCESS` | 200 | The refund was paid. |
| `REJECTED` | 300 | The refund was rejected. |
| `CANCELLED` | 400 | The refund was cancelled. |

## Credit Card Payment Operations

{% api-method method="post" host="https://api.dlocal.com/payments/" path="{id}/capture" %}
{% api-method-summary %}
Capture a credit card payment
{% endapi-method-summary %}

{% api-method-description %}
Capture an existing, uncaptured payment. This is the second half of the two-step payment flow, where first you created a paymentwith the capture option set to false.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="id" type="string" required=true %}
The payment id
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```yaml
{
    "id": "PAY4334346343",
    "amount": 120.00,
    "currency" : "USD",
    "country": "BR",   
    "payment_method_type" : "CARD",
    "payment_method_flow" : "DIRECT",
    "payer":{
        "name" : "Thiago Gabriel",
        "email" : "thiago@example.com",
        "document" : "53033315550",
        "document_type" : "CPF",
        "address": {
            "country" : "BR",
            "state"  : "Rio de Janeiro",
            "city" : "Volta Redonda",
            "zip_code" : "27275-595",
            "street" : "Servidão B-1",
            "number" : "1106"
        }
    },
    "card":{
        "token": "CV-e90078f7-e027-4ce4-84cb-534c877be33c",
        "holder_name": "Thiago Gabriel",
        "expiration_month": 10,
        "expiration_year": 2040,
        "last4": "1111",
        "brand": "VI",
        "capture": true
    },
    "created_date" : "2018-02-15T15:14:52-00:00",
    "approved_date" : "2018-02-15T15:17:52-00:00",
    "status" : "PAID",
    "status_detail" : "The payment was paid.",
    "status_code": 200,
    "external_reference": "657434343",
    "notification_url": "http://merchant.com/notifications"
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

### **Example request**

```bash
$ curl -X POST \
    -H 'X-Date: 2018-02-20T15:44:42.310Z' \
    -H 'X-Login: sak223k2wdksdl2' \
    -H 'Authorization: V2-HMAC-SHA256, Signature: 1bd227f9d892a7f4581b998c21e353b1686a6bdad5940e7bb6aa596c96e0a6ec' \
    https://api.dlocal.com/payments/PAY4334346343/capture
```

{% api-method method="post" host="https://api.dlocal.com/payments/" path="{id}/cancel" %}
{% api-method-summary %}
Cancel a credit card payment
{% endapi-method-summary %}

{% api-method-description %}
Cancel an existing, uncaptured payment. This is the second half of the two-step payment flow, where first you created a payment with the capture option set to false.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="id" type="string" required=true %}
The payment id.
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```yaml
{
    "id": "PAY4334346343",
    "amount": 120.00,
    "currency" : "USD",
    "country": "BR",
    "payment_method_type" : "CARD",
    "payment_method_flow" : "DIRECT",
    "payer":{
        "name" : "Thiago Gabriel",
        "email" : "thiago@example.com",
        "document" : "53033315550",
        "document_type" : "CPF",
        "address": {
            "country" : "BR",
            "state"  : "Rio de Janeiro",
            "city" : "Volta Redonda",
            "zip_code" : "27275-595",
            "street" : "Servidão B-1",
            "number" : "1106"
        }
    },
    "card":{
        "holder_name": "Thiago Gabriel",
        "expiration_month": 10,
        "expiration_year": 2040,
        "last4": "1111",
        "brand": "VI",
        "capture": false
    },
    "created_date" : "2018-02-15T15:14:52-00:00",
    "approved_date" : "2018-02-15T15:14:52-00:00",
    "status" : "CANCELLED",
    "status_detail" : "The payment was cancelled.",
    "status_code": 400,
    "external_reference": "657434343",
    "notification_url": "http://merchant.com/notifications"
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

### Example Request

```bash
$ curl -X POST \
    -H 'X-Date: 2018-02-20T15:44:42.310Z' \
    -H 'X-Login: sak223k2wdksdl2' \
    -H 'Authorization: V2-HMAC-SHA256, Signature: 1bd227f9d892a7f4581b998c21e353b1686a6bdad5940e7bb6aa596c96e0a6ec' \
    https://api.dlocal.com/payments/PAY4334346343/cancel
```

### Chargeback asynchronous notification

If a charge back was applied \(requested by the user\) a notification is sent to the merchant to the previously registered Merchant chargeback notification URL by POST protocol, sending the following parameters:

| **Property** | **Type** | **Description** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `id` | String | The chargeback id. |
| `payment_id` | String | The payment id. |
| `amount` | Positive Float | The amount of the chargeback. |
| `amount_chargedback` | Positive Float | The chargedback amount. |
| `currency` | String | The currency of the chargeback. |
| `status` | String | The status of the chargeback. |
| `status_code` | String | The [status](https://dlocal.com/docs/?language=cURL#chargeback-asynchronous-notification) code of the chargeback. |
| `created_date` | String | The date of when the chargeback was executed. |

**Example post**

POST: _{merchant.chargeback\_url}_

```yaml
{
    "id": "CHAR42342",
    "payment_id": "PAY245235",
    "amount": 100.00,
    "amount_chargedback": 100.00,
    "currency": "USD",
    "status": "SUCCESS",
    "status_code": 200,
    "created_date" : "2018-02-15T15:14:52-00:00"
}
```

### Chargeback status {#chargeback-status}

| **Status** | **Status code** | **Description** |
| --- | --- | --- |
| `PENDING` | 100 | The chargeback is pending. |
| `SUCCESS` | 200 | The chargeback was executed. |

## Installments

You can specify an installment plan, to guarantee the surcharge per installment that will be charged.

### The Installment Plan Object

{% tabs %}
{% tab title="Installment Plan Object" %}
| **Property** | Type | **Description** |
| --- | --- | --- | --- | --- | --- | --- |
| `id` | String | The installments plan id. |
| `country` | String | The country of the installments plan. |
| `currency` | String | The currency of the installments plan. |
| `bin` | String | The credit card bin. |
| `amount` | Positive Float | The amount of the installments plan. |
| `installments` | [Installment Object](api-reference.md#the-installment-object)\[ \] | The installments plan information |
{% endtab %}

{% tab title="Example Installment Plan Object" %}
```yaml
{
    "id" : "INS54434",
    "country" : "BR",
    "currency" : "BRL",
    "bin" : "435921",
    "amount": 1500,
    "installments" : [
        {
            "id" : "INS54434-3",
            "installment_amount" : 550,
            "installments" : 3,
            "total_amount" : 1650
        },
        {
            "id" : "INS54434-6",
            "installment_amount" : 350,
            "installments" : 6,
            "total_amount" : 2100
        },
        {
            "id" : "INS54434-8",
            "installment_amount" : 250,
            "installments" : 12,
            "total_amount" : 3000
        }
    ]
}
```
{% endtab %}
{% endtabs %}

### The Installment Object

{% tabs %}
{% tab title="Installment Object" %}
| **Property** | **Type** | **Description** |
| --- | --- | --- | --- | --- |
| `id` | String | The installment id |
| `installment_amount` | Positive Float | Instalment amount. |
| `total_amount` | Positive Float | Instalments total amount. |
| `installments` | Integer | Number of installments. |
{% endtab %}

{% tab title="Example Installment Object" %}
```yaml
{
"id" : "INS54434-3",
"installment_amount" : 550,
"installments" : 3,
"total_amount" : 1650
}
```
{% endtab %}
{% endtabs %}

{% api-method method="get" host="https://api.dlocal.com/" path="installments-plans" %}
{% api-method-summary %}
Create an Installment Plan
{% endapi-method-summary %}

{% api-method-description %}
Create an installments plan.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-body-parameters %}
{% api-method-parameter name="country" type="string" required=true %}
The country of the installment plan.
{% endapi-method-parameter %}

{% api-method-parameter name="bin" type="string" required=true %}
The credit card bin.
{% endapi-method-parameter %}

{% api-method-parameter name="amount" type="number" required=true %}
The amount of the installment plan. Decimal.
{% endapi-method-parameter %}

{% api-method-parameter name="currency" type="string" required=true %}
The currency of the installments plan.
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```yaml
{
    "id" : "INS54434",
    "country" : "BR",
    "bin" : "435921",
    "amount": 1500,
    "currency" : "BRL",
    "installments" : [
        {
            "id" : "INS54434-3",
            "installment_amount" : 550,
            "installments" : 3,
            "total_amount" : 1650
        },
        {
            "id" : "INS54434-6",
            "installment_amount" : 350,
            "installments" : 6,
            "total_amount" : 2100
        },
        {
            "id" : "INS54434-8",
            "installment_amount" : 250,
            "installments" : 12,
            "total_amount" : 3000
        }
    ]
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

### **Example request**

```bash
$ curl -X POST \
    -H 'X-Date: 2018-02-20T15:44:42.310Z' \
    -H 'X-Login: sak223k2wdksdl2' \
    -H 'Authorization: V2-HMAC-SHA256, Signature: 1bd227f9d892a7f4581b998c21e353b1686a6bdad5940e7bb6aa596c96e0a6ec' \
    -d {body}
    https://api.dlocal.com/installments-plans
```

**Example request body**

```yaml
{
    "country" : "BR",
    "bin" : "435921",
    "amount": 30,
    "currency" : "BRL"
}
```

## Saving Cards

{% api-method method="post" host="" path="" %}
{% api-method-summary %}
Create a Card
{% endapi-method-summary %}

{% api-method-description %}
Stores a card's information on dLocal's servers, and returns a token.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-body-parameters %}
{% api-method-parameter name="country" type="string" required=true %}
Card's country
{% endapi-method-parameter %}

{% api-method-parameter name="card" type="object" required=true %}
The Card Object
{% endapi-method-parameter %}

{% api-method-parameter name="payer" type="object" required=true %}
The Payer Object
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}
Example Response
{% endapi-method-response-example-description %}

```yaml
{
    "token": "CV-af85ddba-f5f4-4644-b3fa-3c922e0687e9",
    "holder_name": "Thiago Gabriel",
    "expiration_month": 10,
    "expiration_year": 2040,
    "last4": "1111",
    "brand": "VI"
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

### The Card Object

| **Argument** | **Type** | **Description** |
| --- | --- | --- | --- | --- | --- | --- |
| `holder_name` | String **\(Required\)** | Cardholder's full name |
| `expiration_month` | Integer **\(Required\)** | Number representing the card's expiration month \(start in 1\). |
| `expiration_year` | Integer **\(Required\)** | Number representing the card's expiration year. |
| `number` | String **\(Required if** `encrypted_data` **not present\)** | The card number, as a string without any separators. |
| `cvv` | String **\(Required is** `encrypted_data` **not present\)** | Credit card verification value. |
| `encrypted_data` | String \(Optional\) | [JWE](https://tools.ietf.org/html/rfc7516) encrypted params. |

### The Payer Object

| **Argument** | **Type** | **Description** |
| --- | --- | --- |
| `name` | String **\(Required\)** | Payer's name |
| `document` | String **\(Required\)** | Payer's document |

### Example Request

```bash
curl -X POST \
    -H 'X-Date: 2018-02-20T15:44:42.310Z' \
    -H 'X-Login: sak223k2wdksdl2' \
    -H 'Authorization: V2-HMAC-SHA256, Signature: 1bd227f9d892a7f4581b998c21e353b1686a6bdad5940e7bb6aa596c96e0a6ec' \
    -d '{body}'
    https://api.dlocal.com/secure_cards
```

**Example request body**

```yaml
{
    "country": "BR",
    "card": {
        "holder_name": "Thiago Gabriel",
        "expiration_month": 10,
        "expiration_year": 2040,
        "number": "41111111111111",
        "cvv": "123",
    },
    "payer": {
        "name": "Luis Gabriel",
        "document": "53033315550",
    }
}
```

{% api-method method="get" host=" https://api.dlocal.com/cards/" path="{token}" %}
{% api-method-summary %}
Retrieve a Card
{% endapi-method-summary %}

{% api-method-description %}
Retrieve information about an existing card.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="token" type="string" required=true %}
The card token.
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```yaml
{
    "token": "CV-af85ddba-f5f4-4644-b3fa-3c922e0687e9",
    "holder_name": "Thiago Gabriel",
    "expiration_month": 10,
    "expiration_year": 2040,
    "last4": "1111",
    "brand": "VI"
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

**Example request**

```bash
$ curl \
    -H 'X-Date: 2018-02-20T15:44:42.310Z' \
    -H 'X-Login: sak223k2wdksdl2' \
    -H 'Authorization: V2-HMAC-SHA256, Signature: 1bd227f9d892a7f4581b998c21e353b1686a6bdad5940e7bb6aa596c96e0a6ec' \
    https://api.dlocal.com/cards/CV-e90078f7-e027-4ce4-84cb-534c877be33c
```

## Currency Exchange

**The Currency Exchange Object**

| **Property** | **Type** | **Description** |
| --- | --- | --- | --- |
| `from` | String | Origin currency code |
| `to` | String | Destination currency code |
| `rate` | Decimal | Ratio of conversion from `from` currency to `to` currency. |

{% api-method method="get" host="https://api.dlocal.com/currency-exchanges?" path="from={from}&to={to}" %}
{% api-method-summary %}
Get an Exchange Rate
{% endapi-method-summary %}

{% api-method-description %}
Get the exchange rate between two currencies.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-query-parameters %}
{% api-method-parameter name="from" type="string" required=true %}
Origin currency code.
{% endapi-method-parameter %}

{% api-method-parameter name="to" type="string" required=true %}
Destination currency code.
{% endapi-method-parameter %}
{% endapi-method-query-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}
Example Response
{% endapi-method-response-example-description %}

```yaml
{
    "from": "USD",
    "to": "BRL",
    "rate": 3.33
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

**Example request**

```bash
curl -X GET \
    -H 'X-Date: 2018-02-20T15:44:42.310Z' \
    -H 'X-Login: sak223k2wdksdl2' \
    -H 'Authorization: V2-HMAC-SHA256, Signature: 1bd227f9d892a7f4581b998c21e353b1686a6bdad5940e7bb6aa596c96e0a6ec' \
    https://api.dlocal.com/currency-exchanges?from=USD&to=BRL
```
