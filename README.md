

# Chapter 1 – Introduction

## 1.1 Protocol

AltoPay Biller API used the HTTP protocol version 1.1. The method used for the request is `POST` or `PUT`.

## 1.2 Messge Format

The message format used is JavaScript Object Notation (JSON). JSON is sent through the request body. The request body length must be included in the header.

## 1.3 Request Headers

HTTP headers allow the client and the server to pass additional information with the request or the response. An HTTP header consists of its case-insensitive name followed by a colon '`:`', then by its value (without line breaks). Leading white space before the value is ignored. Some of headers used to make Inquiry Request and Payment are as follows:

1. **X-timestamp**
**`X-timestamp`** is the time stamp of the transaction with ISO Format in UTC. Time stamp can be used to validate the transaction time. For example `2019-12-31T23:59:59.999Z`.

2. **X-api-key**
**`X-api-key`** is used to identity the client.

3. **X-signature**
**`X-signature`** is required for transaction such as Inquiry and Payment. Signature using _hmac_ and _sha256_ with request method, relative path, times tamp, hash of entity body, username and password as parameters.
**Example**
```php
function signature($method, $path, $body, $timestamp, $apikey, $password)
{
    $hash = hash('sha256', $body);
    $string_to_sign = strtoupper($method).":".$path.":".$apikey.":".$hash.":".$timestamp;
    return bin2hex(hash_hmac('sha256', $string_to_sign, $password, true));
}

$request_body = json_encode($request_data);
$method       = "POST";
$path         = "/biller/";
$timestamp    = gmdate("Y-m-d\\TH:i:s").".000Z";
$password     = "--YOUR SECRET KEY HERE--";
$api_key      = "--YOUR API KEY HERE--";
$signature    = signature($method, $path, $request_body, $timestamp, $api_key, $password);
$headers      = array(
    'Content-Type: application/json',
    'Authorization: Basic '.base64_encode('kamshory:password'),
    'X-Signature: '.$signature,
    'X-Api-Key: '.$api_key,
    'X-Timestamp: '.$timestamp
);

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, "https://dev.altopay.id:9090/biller/");
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
curl_setopt($ch, CURLOPT_USERAGENT, 'Planet POS');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
curl_setopt($ch, CURLOPT_POSTFIELDS, $request_body);
$response = curl_exec($ch);
$err = curl_error($ch);
curl_close( $ch );
```

| **Parameter** |  **Description** | **Example** |
| --- | --- | --- |
| method | Request Method (GET, PUT, POST) |  POST |
| path | Relative path of URL | /virtual-account/ |
| body | Original request body without any modification | {“command”:”inquiry”, “data”:{}} |
| timestamp | ISO Time Stamp | 2019-07-07T23:59:59.999Z |
| apikey | API Key | |
| password | Client password | |

**Note:**
a. timestamp and apikey sent on request header.
b. password is not sent on request.

5. **Content-type**
**`Content-type`** is filled with `json/application`.

5. **Content-encoding**
**`Content-type`** is filled with `identity`.

4. **Content-length**
**`Content-length`** is the length of the request body in bytes. The client must calculate the length of the  request body before sending it to the server. Therefore, the merchant may not send a request with partial content.


6. **Host**
**`Host`** is server address that will be accessed by merchant. This address contains: domain/subdomain, domain/subdomain and port, IP address, IP address and port. For example ``dev.altopay.id:9090``.

## 1.4 Request Body

Every request body always contains this two properties:

1. **`command`  (String)**
It is a command that must be execute by the server. This command must be write in lowercase letters. If the command is written incorrectly, the server will send a response with response code “Invalid Transaction” with an error message “Invalid Command”.
2. **`product_code` (String)**
It is the product code that will be requested on the transaction. If product is not found, AltoPay Biller will reject the transaction.

2. **`process_category` (String)**
It is the the alternative of the `product_code`. AltoPay Biller will find the product by combinating the `process_category`, `cutomer_number`, and `amount`. It will simplify the message for prepaid and postpaid cell phone credit payment. If product is not found, AltoPay Biller will reject the transaction.

4. **`data` (Object)**
The data (object) contains the request message detail. Every command has different data properties.

## 1.5 URL
The URL for production is different from the URL for development. The URL for production will be informed after signing the collaboration. For development purposes, use the development URL provided. This URL might change.

# Chapter 2 - Topology

# Chapter 3 – Transaction Flow

# Chapter 4 – General Message Format

The message format on one product will be different from the message format on other products. However, in general, each product has a uniform message format. This chapter will explain the general message format for all products.

## 4.1 Inquiry

Inquiry is an electronic transaction to find out billing information for certain customer ID of the biller.

### 4.1.1 Request

#### a. Header

**Accept**

The `Accept` request HTTP header advertises which content types, expressed as MIME types, the client is able to understand. Using content negotiation, the server then selects one of the proposals, uses it and informs the client of its choice with the Content-Type response header.

**Accept-encoding**

`Accept-encoding` is encoding that can be accepted by client. Acceptable encoding, namely:

1. identity (without compression)
2. gzip (with GZIP compression)

**Content-type**

The `Content-type` entity header indicates the size of the entity-body, in bytes, sent to the recipient.

**Content-Encoding**

The `Content-Encoding` entity header is used to compress the media-type. When present, its value indicates which encodings were applied to the entity-body. It lets the client know how to decode in order to obtain the media-type referenced by the `Content-Type` header.

**Content-length**

The `Content-length` entity header indicates the size of the entity-body, in bytes, sent to recipient.

**Connection**

The `Connection` general header controls whether or not the network connection stays open after the current transaction finishes. If the value sent is `keep-alive`, the connection is persistent and not closed, allowing for subsequent requests to the same server to be done.

**User-agent**

The **User-Agent** request header contains a characteristic string that allows the network protocol peers to identify the application type, operating system, software vendor or software version of the requesting software user agent.  

**Host**

Host is the name of the server accessed by AltoPay. From this header, the merchant server can find out how AltoPay is connected to the merchant server.

**X-api-key**

The `X-api-key` is used to identity the client.

See **Request Headers** section on Chapter 1.

**X-timestamp**

The `X-timestamp` is one of signature component of the request to guarantee data integrity.

See **Request Headers** section on Chapter 1.

**X-signature**

The `X-signature` is signature of the request to guarantee data integrity. The parameters of the X-Signature are:

1. metod
2. path
3. body
4. timestamp
5. apikey
6. password

See **Request Headers** section on Chapter 1.

#### b. Body

| **No** | **Parameter**|**Type**|**Description**|**Note**|
|---|---|---|---|---|
|1|command|String|Transaction command. See **Transaction Command List**|Mandatory|
|2|product_code|Numeric, can begin with 0|Product code|Mandatory|
|3|process_category|Numeric, can begin with 0, alternative of product code|Process category|Conditional|
|4|data|Object|Container of the transaction data|Mandatory|
|5|data.customer_id|Numeric, can begin with 0|Customer ID|Mandatory|
|6|data.date_time|String|Transmission date and time in GMT (Format: yyyy-MM-d’T’HH:mm:ss.SSS’Z’)|Mandatory|
|7|data.reference_number|String (32)|Reference number|Mandatory|

Note: The data length above is the maximum allowed. Shorter will be better.

### 4.1.2 Response

#### a. Header

**Content-type**

The `Content-type` entity header indicates the size of the entity-body, in bytes, sent to the recipient.

**Content-encoding**

`Content-encoding` is content encoding that sent by AltoPay. Content encoding, namely:

1. identity (without compression)
2. gzip (with GZIP compression)

**Content-length**

The `Content-length` entity header indicates the size of the entity-body, in bytes, sent to recipient.

**Date**

The `Date` general HTTP header contains the date and time at which the message was originated.

#### b. Body

| **No** | **Parameter**|**Type**|**Description**|**Note**|
|---|---|---|---|---|
|1|command|String|Transaction command|Mandatory|
|2|product_code|Numeric, can begin with 0|Product code|Mandatory|
|3|response_code|Numeric, can begin with 0|Product code|Mandatory|
|4|response_text|String|Description of the response transaction|Mandatory|
|5|data|Object|Container of the transaction data|Mandatory|
|6|data.customer_id|Numeric, can begin with 0|Customer ID|Mandatory|
|7|data.customer_name|String (32)|Customer name|Mandatory|
|8|data.amount|Numeric|Bill amount|Mandatory|
|9|data.date_time|String|Transmission date and time in GMT (Format: yyyy-MM-d’T’HH:mm:ss.SSS’Z’)|Mandatory|
|10|data.reference_number|String (32)|Reference number|Mandatory|

Note: The data length above is the maximum allowed. Shorter will be better.

## 4.2 Payment

Payment is an electronic transaction to pay the bills for certain customer ID of the biller.


### 4.2.1 Request

#### a. Header

**Accept**

The `Accept` request HTTP header advertises which content types, expressed as MIME types, the client is able to understand. Using content negotiation, the server then selects one of the proposals, uses it and informs the client of its choice with the Content-Type response header.

**Accept-encoding**

`Accept-encoding` is encoding that can be accepted by client. Acceptable encoding, namely:

1. identity (without compression)
2. gzip (with GZIP compression)

**Content-type**

The `Content-type` entity header indicates the size of the entity-body, in bytes, sent to the recipient.

**Content-Encoding**

The `Content-Encoding` entity header is used to compress the media-type. When present, its value indicates which encodings were applied to the entity-body. It lets the client know how to decode in order to obtain the media-type referenced by the `Content-Type` header.

**Content-length**

The `Content-length` entity header indicates the size of the entity-body, in bytes, sent to recipient.

**Connection**

The `Connection` general header controls whether or not the network connection stays open after the current transaction finishes. If the value sent is `keep-alive`, the connection is persistent and not closed, allowing for subsequent requests to the same server to be done.

**User-agent**

The **User-Agent** request header contains a characteristic string that allows the network protocol peers to identify the application type, operating system, software vendor or software version of the requesting software user agent.  

**Host**

Host is the name of the server accessed by AltoPay. From this header, the merchant server can find out how AltoPay is connected to the merchant server.

**X-api-key**

The `X-api-key` is used to identity the client.

See **Request Headers** section on Chapter 1.

**X-timestamp**

The `X-timestamp` is one of signature component of the request to guarantee data integrity.

See **Request Headers** section on Chapter 1.

**X-signature**

The `X-signature` is signature of the request to guarantee data integrity. The parameters of the X-Signature are:

1. metod
2. path
3. body
4. timestamp
5. apikey
6. password

See **Request Headers** section on Chapter 1.

#### Body

| **No** | **Parameter**|**Type**|**Description**|**Note**|
|---|---|---|---|---|
|1|command|String|Transaction command|Mandatory|
|2|product_code|Numeric, can begin with 0|Product code|Mandatory|
|3|process_category|Numeric, can begin with 0, alternative of product code|Process category|Conditional|
|4|data|Object|Container of the transaction data|Mandatory|
|5|data.customer_id|Numeric, can begin with 0|Customer ID|Mandatory|
|6|data.customer_name|String|Customer ID|Mandatory for several billers|
|7|data.amount|Numeric|Payment amount|For several product, amount must be same with amount on inquiry response
|8|data.date_time|String|Transmission date and time in GMT (Format: yyyy-MM-d’T’HH:mm:ss.SSS’Z’)|Mandatory|
|9|data.fwd_reference_number|String (32)|Reference number|For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response|
|8|data.fwd_stan|Numeric|Payment amount|For several product, fwd_stan must be same with fwd_stan on inquiry response|

### 4.2.2 Response

#### a. Header

**Content-type**

The `Content-type` entity header indicates the size of the entity-body, in bytes, sent to the recipient.

**Content-encoding**

`Content-encoding` is content encoding that sent by AltoPay. Content encoding, namely:

1. identity (without compression)
2. gzip (with GZIP compression)

**Content-length**

The `Content-length` entity header indicates the size of the entity-body, in bytes, sent to recipient.

**Date**

The `Date` general HTTP header contains the date and time at which the message was originated.

#### b. Body

| **No** | **Parameter**|**Type**|**Description**|**Note**|
|---|---|---|---|---|
|1|command|String|Transaction command|Mandatory|
|2|product_code|Numeric, can begin with 0|Product code|Mandatory|
|3|response_code|Numeric, can begin with 0|Product code|Mandatory|
|4|response_text|String|Description of the response transaction|Mandatory|
|5|data|Object|Container of the transaction data|Mandatory|
|6|data.customer_id|Numeric, can begin with 0|Customer ID|Mandatory|
|7|data.customer_name|String|Customer ID|Mandatory|
|8|data.amount|Numeric|Payment amount||
|9|data.time_stamp|String|Transmission date and time in GMT (Format: yyyy-MM-d’T’HH:mm:ss.SSS’Z’)|Mandatory|
|10|data.fwd_reference_number|String (32)|Reference number|Mandatory|
|11|data.fwd_stan|Numeric|Payment amount|Mandatory|

Note: The data length above is the maximum allowed. Shorter will be better.


# Chapter 5 – Product Message Format

Each product has a different message format depending on each biller. AltoPay adjusts the message format for each product according to its needs. Some products have far more data attributes than other products because they are related to biller policy.

Some products that have the same attributes and processes are combined into one product category, namely general billing. Some more products are made with special categories because they have very big differences.

Every message is always listed one of _product_code_ and _category_process. category_process_ is only used for prepaid cellular phone top up and postpaid cell phone bill payments.

## 5.1 Prepaid Electricity

Prepaid electricity is a product of the _Perusahaan Listrik Negara_ (PLN) where customers can use electricity in accordance with the quota that has been purchased. By purchasing a power quota, customers will get a token that can be entered into the power controller.

The customer must enter the customer ID or meter number at the time of purchase because the generated token will be adjusted to the customer's meter number.

To prevent mistakes when purchasing, PLN requires customers to conduct an inquiry to confirm whether the customer number or meter number entered is correct.

### 5.1.1 Inquiry Request

**Message Sample**

```
POST /biller/ HTTP/1.1
Content-type: application/json
Accept: application/json
Content-encoding: identity
Accept-encoding: identity
Host: dev.altopay.id:9090/biller/
Connection: close
User-agent: Planet POS
X-api-key: 650a8e7e-b97f-11e9-a2a3-2a2ae2dbcce4
X-timestamp: 2019-08-08T08:08:08
X-signature: 769ab57e146beaefa1f0536f230e2ca0a86bafbe4186a8c9b2fe69fdac1f2026
Content-length: 388

{
	"command":"inquiry",
	"product_code":"00500050001",
	"data":{
		"date_time":"2019-10T23:56:59.987Z",
		"reference_number":"1234567889",
		"customer_id":"000000000000",
		"meter_id":"14987654321",
		"id_selector":"0",
		"merchant_type":"6021",
		"locket_name":"PPOB",
		"locket_address":"Jalan Anggrek Neli Murni",
		"locket_code":"1234",
		"locket_phone":"02199999"
	}
}
```

**Field Description**

| No | Type    | Length | Variable    | Description                                                                        |
|----|---------|--------|-------------|------------------------------------------------------------------------------------|
| 1  | String  | 11     | meter_id    | Meter ID                                                                           |
| 2  | String  | 12     | customer_id | Customer ID                                                                        |
| 3  | Decimal | 1      | id_selector | ID Selector. 0 if the reference is meter ID and 1 if the reference is customer ID. |

### 5.1.2 Inquiry Response

**Message Sample**

```
HTTP/1.1 200 OK
Content-type: application/json
Content-encoding: identity
Date: Thu, 8 Aug 2019 08:08:08 GMT
Content-length: 935

{
	"command":"inquiry",
	"product_code":"00500050001",
	"response_code":"00",
	"response_text":"Approve Transaction",
	"biller_response":"Sukses",
	"biller_message":"Sukses",
	"data":{
		"time_stamp":"2019-10T23:57:59.987Z",
		"reference_number":"1234567890",
		"customer_id":"149999999911",
		"meter_id":"14987654321",
		"id_selector":"0",
		"fwd_stan":"000003",
		"fwd_reference_number":"000000002161",
		"biller_reference_number":"6BCB14D5D8AF9000003DCA6F4E1369D8",
		"ba_reference_number":"DB33B717655A8246B5DA7AF722EBD408",
		"customer_name":"HAMDANIE LESTALUHUANI",
		"maximum_power_unit_print":"06000",
		"transaction_currency_code":"360",
		"power_purchase_unsold_print_1":"0",
		"power_purchase_unsold_print_2":"0",
		"tariff":"R1",
		"distribution_code_print":"51",
		"ceiling":"000000900",
		"total_repeat_print":"0",
		"service_unit_phone_print":"123",
		"service_unit_name_print":"51106",
	}
}
```

**Field Description**

| No | Type    | Length | Variable                      | Description                                                                                                                                                                                                                                                                                                                                              |
|----|---------|--------|-------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1  | Decimal | 11     | meter_id                      | Meter ID                                                                                                                                                                                                                                                                                                                                                 |
| 2  | Decimal | 12     | customer_id                   | Costomer ID |
| 3  | Decimal | 1      | id_selector                   | ID Selector. 0 if the reference is meter ID and 1 if the reference is customer ID.                                                                                                                                                                                                                                                                       |
| 4  | String  | 32     | biller_reference_number       | Biller reference number                                                                                                                                                                                                                                                                                                                                  |
| 5  | String  | 32     | ba_reference_number           | Biller aggregator reference number                                                                                                                                                                                                                                                                                                                       |
| 6  | String  | 25     | customer_name                 | Customer Name                                                                                                                                                                                                                                                                                                                                            |
| 7  | String  | 4      | tariff                        | Tariff applied                                                                                                                                                                                                                                                                                                                                           |
| 8  | String  | 9      | ceiling                       | Maximum power limits that can be used                                                                                                                                                                                                                                                                                                                    |
| 9  | String  | 2      | distribution_code             | Distribution Code                                                                                                                                                                                                                                                                                                                                        |
| 10 | String  | 5      | service_unit_name             | Service Unit Name                                                                                                                                                                                                                                                                                                                                        |
| 11 | String  | 15     | service_unit_phone            | Service Unit Phone                                                                                                                                                                                                                                                                                                                                       |
| 12 | String  | 5      | maximum_power_unit            | Maximum power unit                                                                                                                                                                                                                                                                                                                                       |
| 13 | String  | 1      | total_repeat                  | Total Repeat                                                                                                                                                                                                                                                                                                                                             |
| 14 | String  | 11     | power_purchase_unsold         | Power Purchase Unsold                                                                                                                                                                                                                                                                                                                                    |
| 15 | String  | 2      | distribution_code_print       | Distribution Code for Print                                                                                                                                                                                                                                                                                                                              |
| 16 | String  | 5      | service_unit_name_print       | Service Unit Name for Print                                                                                                                                                                                                                                                                                                                              |
| 17 | String  | 15     | service_unit_phone_print      | Service Unit Phone for Print                                                                                                                                                                                                                                                                                                                             |
| 18 | String  | 5      | maximum_power_unit_print      | Maximum power unit for print                                                                                                                                                                                                                                                                                                                             |
| 19 | String  | 1      | total_repeat_print            | Total Repeat for Print                                                                                                                                                                                                                                                                                                                                   |
| 20 | Decimal | 11     | power_purchase_unsold_print_1 | Power purchase unsold 1 for print                                                                                                                                                                                                                                                                                                                        |
| 21 | Decimal | 11     | power_purchase_unsold_print_2 | Power purchase unsold 2 for print                                                                                                                                                                                                                                                                                                                        |
| 22 | Decimal | 12     | amount| Amount |

### 5.1.3 Purchase Request

**Message Sample**

```
POST /biller/ HTTP/1.1
Content-type: application/json
Accept: application/json
Content-encoding: identity
Accept-encoding: identity
Host: dev.altopay.id:9090/biller/
Connection: close
User-agent: Planet POS
X-api-key: 650a8e7e-b97f-11e9-a2a3-2a2ae2dbcce4
X-timestamp: 2019-08-08T08:08:08
X-signature: a01dcd3b41fd6e71ed55e4d92402826f7f369e0af01db194cc9d32d7eb2c322a
Content-length: 879

{
	"command":"payment",
	"product_code":"00500050001",
	"data":{
		"date_time":"2019-10T23:58:59.987Z",
		"reference_number":"1234567891",
		"customer_id":"149999999911",
		"meter_id":"14987654321",
		"id_selector":"0",
		"merchant_type":"6021",
		"locket_name":"PPOB",
		"locket_address":"Jalan Anggrek Neli Murni",
		"locket_code":"1234",
		"locket_phone":"02199999",
		"fwd_stan":"000003",
		"fwd_reference_number":"000000002161",
		"purchase_option":"0",
		"maximum_power_unit_print":"06000",
		"power_purchase_unsold_print_1":"0",
		"power_purchase_unsold_print_2":"0",
		"tariff":"R1",
		"distribution_code_print":"51",
		"ceiling":"000000900",
		"amount":"20000",
		"admin_fee":"2500",
		"transaction_currency_code":"360",
		"service_unit_name_print":"51106",
		"service_unit_phone_print":"123",
		"customer_name":"HAMDANIE LESTALUHUANI"
	}
}
```

**Field Description**

| No | Type    | Length | Variable                      | Description                                                                        |
|----|---------|--------|-------------------------------|------------------------------------------------------------------------------------|
| 1  | String  | 11     | meter_id                      | Meter ID                                                                           |
| 2  | String  | 12     | customer_id                   | Customer ID                                                                        |
| 3  | Decimal | 1      | id_selector                   | ID Selector. 0 if the reference is meter ID and 1 if the reference is customer ID. |
| 4  | String  | 32     | biller_reference_number       | Biller reference number                                                            |
| 5  | String  | 32     | ba_reference_number           | Biller aggregator reference number                                                 |
| 6  | String  | 25     | customer_name                 | Customer Name                                                                      |
| 7  | String  | 4      | tariff                        | Tariff applied                                                                     |
| 8  | String  | 9      | ceiling                       | Maximum power limits that can be used                                              |
| 9  | Decimal | 1      | purchase_option               | Purchase Option                                                                    |
| 10 | String  | 2      | distribution_code_print       | Distribution Code for Print                                                        |
| 11 | String  | 5      | service_unit_name_print       | Service Unit Name for Print                                                        |
| 12 | String  | 15     | service_unit_phone_print      | Service Unit Phone for Print                                                       |
| 13 | String  | 5      | maximum_power_unit_print      | Maximum power unit for print                                                       |
| 14 | String  | 1      | total_repeat_print            | Total Repeat for Print                                                             |
| 15 | Decimal | 11     | power_purchase_unsold_print_1 | Power purchase unsold 1 for print                                                  |
| 16 | Decimal | 11     | power_purchase_unsold_print_2 | Power purchase unsold 2 for print                                                  |
| 17 | String  | 32     | locket_code                   | Locket code where customer pay                                                     |
| 18 | String  | 30     | locket_name                   | Locket name where customer pay                                                     |
| 19 | String  | 50     | locket_address                | Locket address where customer pay                                                  |
| 20 | String  | 18     | locket_phone                  | Locket phone where customer pay                                                    |
| 21 | Decimal | 12     | amount| Amount |


### 5.1.4 Purchase Response

**Message Sample**

```
HTTP/1.1 200 OK
Content-type: application/json
Content-encoding: identity
Date: Thu, 8 Aug 2019 08:08:08 GMT
Content-length: 1298

{
	"command":"payment",
	"product_code":"00500050001",
	"response_code":"00",
	"response_text":"Approve Transaction",
	"biller_response":"Sukses",
	"biller_message":"Sukses",
	"data":{
		"time_stamp":"2019-10T23:59:59.987Z",
		"reference_number":"1234567891",
		"customer_id":"149999999911",
		"meter_id":"14987654321",
		"id_selector":"0",
		"fwd_stan":"000002",
		"fwd_reference_number":"000000002159",
		"biller_reference_number":"5530F9CFAAA43000002D9B282B210247",
		"ba_reference_number":"B662C985367FCE7F492DACADAB244F0F",
		"purchase_option":"0",
		"power_purchase_unsold":"0",
		"maximum_power_unit":"0",
		"decimal_installment":"0",
		"decimal_unit_price":"0",
		"local_time":"173425",
		"decimal_ppju":"0",
		"distribution_code":"",
		"installment":"0",
		"admin_fee":"2500",
		"decimal_stamp":"0",
		"decimal_admin_fee":"0",
		"stamp":"0",
		"transaction_currency_code":"360",
		"decimal_ppn":"0",
		"service_unit_name":"",
		"vending_receipt_number":"HAMDANIE",
		"tariff":"R1  ",
		"power":"0",
		"ceiling":"000000900",
		"amount":"0",
		"service_unit_phone":"",
		"unit_price":"0",
		"total_repeat":"0",
		"ppn":"0",
		"token":"",
		"decimal_power":"0",
		"date_paid_off":"",
		"customer_name":"HAMDANIE LESTALUHUANI",
		"ppju":"0"
	}
}
```

**Field Description**

| No | Type    | Length | Variable                | Description                                                                        |
|----|---------|--------|-------------------------|------------------------------------------------------------------------------------|
| 1  | Decimal | 11     | meter_id                | Meter ID                                                                           |
| 2  | Decimal | 12     | customer_id             | Customer ID                                                                        |
| 3  | Decimal | 1      | id_selector             | ID Selector. 0 if the reference is meter ID and 1 if the reference is customer ID. |
| 4  | String  | 32     | biller_reference_number | Biller reference number                                                            |
| 5  | String  | 32     | ba_reference_number     | Biller aggregator reference number                                                 |
| 6  | Decimal | 8      | vending_receipt_number  | Vending Receipt Number                                                             |
| 7  | String  | 25     | customer_name           | Customer Name                                                                      |
| 8  | String  | 4      | tariff                  | Tariff applied                                                                     |
| 9  | String  | 9      | ceiling                 | Maximum power limits that can be used                                              |
| 10 | String  | 1      | purchase_option         | Purchase Option                                                                    |
| 11 | Decimal | 1      | decimal_admin_fee       | Decimal Admin Fee                                                                  |
| 12 | Decimal | 10     | admin_fee               | Admin fee for the transaction                                                      |
| 13 | Decimal | 1      | decimal_stamp           | Decimal Stamp                                                                      |
| 14 | Decimal | 10     | stamp                   | Stamp                                                                              |
| 15 | Decimal | 1      | decimal_ppn             | Decimal PPN                                                                        |
| 16 | Decimal | 10     | ppn                     | PPN                                                                                |
| 17 | Decimal | 1      | decimal_ppju            | Decimal PPJU                                                                       |
| 18 | Decimal | 10     | ppju                    | PPJU                                                                               |
| 19 | Decimal | 1      | decimal_installment     | Decimal Installment                                                                |
| 20 | Decimal | 10     | installment             | Installment                                                                        |
| 21 | Decimal | 1      | decimal_power           | Decimal Power                                                                      |
| 22 | Decimal | 12     | power                   | Power                                                                              |
| 23 | Decimal | 1      | decimal_unit_price      | Decimal Unit Price                                                                 |
| 24 | Decimal | 10     | unit_price              | Unit Price                                                                         |
| 25 | String  | 20     | token                   | Token of electrical quota                                                          |
| 26 | String  | 14     | date_paid_off           | Date Paid Off                                                                      |
| 27 | String  | 2      | distribution_code       | Distribution Code                                                                  |
| 28 | String  | 5      | service_unit_name       | Service Unit Name                                                                  |
| 29 | String  | 15     | service_unit_phone      | Service Unit Phone                                                                 |
| 30 | Decimal | 5      | maximum_power_unit      | Maximum power unit                                                                 |
| 31 | Decimal | 1      | total_repeat            | Total Repeat                                                                       |
| 32 | Decimal | 11     | power_purchase_unsold   | Power Purchase Unsold                                                              |
| 33 | Decimal | 12     | amount| Amount |

### 5.1.5 Prepaid Advice Request

**Message Sample**

```
POST /biller/ HTTP/1.1
Content-type: application/json
Accept: application/json
Content-encoding: identity
Accept-encoding: identity
Host: dev.altopay.id:9090/biller/
Connection: close
User-agent: Planet POS
X-api-key: 650a8e7e-b97f-11e9-a2a3-2a2ae2dbcce4
X-timestamp: 2019-08-08T08:08:08
X-signature: a01dcd3b41fd6e71ed55e4d92402826f7f369e0af01db194cc9d32d7eb2c322a
Content-length: 879

{
	"command":"advice",
	"product_code":"00500050001",
	"data":{
		"date_time":"2019-10T23:58:59.987Z",
		"reference_number":"1234567891",
		"customer_id":"149999999911",
		"meter_id":"14987654321",
		"id_selector":"0",
		"merchant_type":"6021",
		"locket_name":"PPOB",
		"locket_address":"Jalan Anggrek Neli Murni",
		"locket_code":"1234",
		"locket_phone":"02199999",
		"fwd_stan":"000003",
		"fwd_reference_number":"000000002161",
		"purchase_option":"0",
		"maximum_power_unit_print":"06000",
		"power_purchase_unsold_print_1":"0",
		"power_purchase_unsold_print_2":"0",
		"tariff":"R1",
		"distribution_code_print":"51",
		"ceiling":"000000900",
		"amount":"20000",
		"admin_fee":"2500",
		"transaction_currency_code":"360",
		"service_unit_name_print":"51106",
		"service_unit_phone_print":"123",
		"customer_name":"HAMDANIE LESTALUHUANI"
	}
}
```

**Field Description**

| No | Type    | Length | Variable                      | Description                                                                        |
|----|---------|--------|-------------------------------|------------------------------------------------------------------------------------|
| 1  | String  | 11     | meter_id                      | Meter ID                                                                           |
| 2  | String  | 12     | customer_id                   | Customer ID                                                                        |
| 3  | Decimal | 1      | id_selector                   | ID Selector. 0 if the reference is meter ID and 1 if the reference is customer ID. |
| 4  | String  | 32     | biller_reference_number       | Biller reference number                                                            |
| 5  | String  | 32     | ba_reference_number           | Biller aggregator reference number                                                 |
| 6  | String  | 25     | customer_name                 | Customer Name                                                                      |
| 7  | String  | 4      | tariff                        | Tariff applied                                                                     |
| 8  | String  | 9      | ceiling                       | Maximum power limits that can be used                                              |
| 9  | Decimal | 1      | purchase_option               | Purchase Option                                                                    |
| 10 | String  | 2      | distribution_code_print       | Distribution Code for Print                                                        |
| 11 | String  | 5      | service_unit_name_print       | Service Unit Name for Print                                                        |
| 12 | String  | 15     | service_unit_phone_print      | Service Unit Phone for Print                                                       |
| 13 | String  | 5      | maximum_power_unit_print      | Maximum power unit for print                                                       |
| 14 | String  | 1      | total_repeat_print            | Total Repeat for Print                                                             |
| 15 | Decimal | 11     | power_purchase_unsold_print_1 | Power purchase unsold 1 for print                                                  |
| 16 | Decimal | 11     | power_purchase_unsold_print_2 | Power purchase unsold 2 for print                                                  |
| 17 | String  | 32     | locket_code                   | Locket code where customer pay                                                     |
| 18 | String  | 30     | locket_name                   | Locket name where customer pay                                                     |
| 19 | String  | 50     | locket_address                | Locket address where customer pay                                                  |
| 20 | String  | 18     | locket_phone                  | Locket phone where customer pay                                                    |

### 5.1.6 Prepaid Advice Response

## Prepaid Cell Phone Credit

### Payment Request

```
POST /biller/ HTTP/1.1
Content-type: application/json
Accept: application/json
Content-encoding: identity
Accept-encoding: identity
Host: dev.altopay.id:9090/biller/
Connection: close
User-agent: Planet POS
X-api-key: 650a8e7e-b97f-11e9-a2a3-2a2ae2dbcce4
X-timestamp: 2019-08-08T08:08:08
X-signature: 21834cad678df58049af0a0b17d44d850a84f0525e4443e41e9385a4208ceb1e
Content-length: 200

{
	"command":"payment",
	"process_category":"1010",
	"data":{
		"date_time":"2019-10T23:56:59.987Z",
		"reference_number":"1234567889",
		"customer_id":"08123456789",
		"amount":"100000"
	}
}
```

### Payment Response

**Message Sample**

```
HTTP/1.1 200 OK
Content-type: application/json
Content-encoding: identity
Date: Thu, 8 Aug 2019 08:08:08 GMT
Content-length: 336

{
	"command":"payment",
	"process_category":"1010",
	"response_code":"00",
	"response_text":"Success",
	"command":"payment",
	"data":{
		"time_stamp":"2019-10T23:56:59.987Z",
		"reference_number":"1234567889",
		"fwd_reference_number":"99999",
		"fwd_stan":"556287",
		"customer_id":"08123456789",
		"amount":"100000"
	}
}
```


## Postpaid Cell Phone Credit

### Inquiry Request

**Message Sample**

```
POST /biller/ HTTP/1.1
Content-type: application/json
Accept: application/json
Content-encoding: identity
Accept-encoding: identity
Host: dev.altopay.id:9090/biller/
Connection: close
User-agent: Planet POS
X-api-key: 650a8e7e-b97f-11e9-a2a3-2a2ae2dbcce4
X-timestamp: 2019-08-08T08:08:08
X-signature: f3571b283dfd6579e9208c25e24a19e07941dae1e0c5b11d905ece2f34ec8585
Content-length: 176

{
	"command":"inquiry",
	"process_category":"1011",
	"data":{
		"date_time":"2019-10T23:56:59.987Z",
		"reference_number":"1234567889",
		"customer_id":"081234567"
	}
}
```


### Inquiry Response

**Message Sample**

```
HTTP/1.1 200 OK
Content-type: application/json
Content-encoding: identity
Date: Thu, 8 Aug 2019 08:08:08 GMT
Content-length: 363

{
	"command":"payment",
	"process_category":"1011",
	"response_code":"00",
	"response_text":"Success",
	"command":"payment",
	"data":{
		"time_stamp":"2019-10T23:56:59.987Z",
		"reference_number":"1234567889",
		"fwd_reference_number":"99999",
		"fwd_stan":"556287",
		"customer_id":"08123456789",
		"customer_name":"Jono",
		"amount":"100000"
	}
}
```

### Payment Request

**Message Sample**

```
POST /biller/ HTTP/1.1
Content-type: application/json
Accept: application/json
Content-encoding: identity
Accept-encoding: identity
Host: dev.altopay.id:9090/biller/
Connection: close
User-agent: Planet POS
X-api-key: 650a8e7e-b97f-11e9-a2a3-2a2ae2dbcce4
X-timestamp: 2019-08-08T08:08:08
X-signature: 8a3f728a8b2ae70e80b46742c74dc9bad45dea8a9671bb7b2e403c6ba4cca125
Content-length: 296

{
	"command":"payment",
	"process_category":"6666",
	"data":{
		"date_time":"2019-10T23:56:59.987Z",
		"reference_number":"1234567889",
		"reference_number":"1234567889",
		"fwd_reference_number":"99999",
		"customer_id":"081234567",
		"customer_name":"Jono",
		"amount":"100000"
	}
}
```


### Payment Response

**Message Sample**

```
HTTP/1.1 200 OK
Content-type: application/json
Content-encoding: identity
Date: Thu, 8 Aug 2019 08:08:08 GMT
Content-length: 363

{
	"command":"payment",
	"process_category":"6666",
	"response_code":"00",
	"response_text":"Success",
	"command":"payment",
	"data":{
		"time_stamp":"2019-10T23:56:59.987Z",
		"reference_number":"1234567889",
		"fwd_reference_number":"99999",
		"fwd_stan":"556287",
		"customer_id":"08123456789",
		"customer_name":"Jono",
		"amount":"100000"
	}
}
```


## General Bill


### Inquiry Request

**Message Sample**

```
POST /biller/ HTTP/1.1
Content-type: application/json
Accept: application/json
Content-encoding: identity
Accept-encoding: identity
Host: dev.altopay.id:9090/biller/
Connection: close
User-agent: Planet POS
X-api-key: 650a8e7e-b97f-11e9-a2a3-2a2ae2dbcce4
X-timestamp: 2019-08-08T08:08:08
X-signature: 526709aaf22e9523cb5dcea826e586b36955a17d6ceb0da06d9d629913d29c48
Content-length: 177

{
	"command":"inquiry",
	"product_code":"482382366",
	"data":{
		"date_time":"2019-10T23:56:59.987Z",
		"reference_number":"1234567889",
		"customer_id":"081234567"
	}
}
```

### Inquiry Response

**Message Sample**

```
HTTP/1.1 200 OK
Content-type: application/json
Content-encoding: identity
Date: Thu, 8 Aug 2019 08:08:08 GMT
Content-length: 364

{
	"command":"payment",
	"product_code":"482382366",
	"response_code":"00",
	"response_text":"Success",
	"command":"payment",
	"data":{
		"time_stamp":"2019-10T23:56:59.987Z",
		"reference_number":"1234567889",
		"fwd_reference_number":"99999",
		"fwd_stan":"556287",
		"customer_id":"08123456789",
		"customer_name":"Jono",
		"amount":"100000"
	}
}
```

### Payment Request

**Message Sample**

```
POST /biller/ HTTP/1.1
Content-type: application/json
Accept: application/json
Content-encoding: identity
Accept-encoding: identity
Host: dev.altopay.id:9090/biller/
Connection: close
User-agent: Planet POS
X-api-key: 650a8e7e-b97f-11e9-a2a3-2a2ae2dbcce4
X-timestamp: 2019-08-08T08:08:08
X-signature: 41479a908c083b4c782b2fe8be7f9582c932d83126e5ad3fa3fde941b025a6c1
Content-length: 285

{
	"command":"payment",
	"product_code":"482382366",
	"data":{
		"date_time":"2019-10T23:56:59.987Z",
		"reference_number":"1234567889",
		"fwd_reference_number":"99999",
		"fwd_stan":"556287",
		"customer_id":"081234567",
		"customer_name":"Jono",
		"amount":"100000"
	}
}
```

### Payment Response

**Message Sample**

```
HTTP/1.1 200 OK
Content-type: application/json
Content-encoding: identity
Date: Thu, 8 Aug 2019 08:08:08 GMT
Content-length: 364

{
	"command":"payment",
	"product_code":"482382366",
	"response_code":"00",
	"response_text":"Success",
	"command":"payment",
	"data":{
		"time_stamp":"2019-10T23:56:59.987Z",
		"reference_number":"1234567889",
		"fwd_reference_number":"99999",
		"fwd_stan":"556287",
		"customer_id":"08123456789",
		"customer_name":"Jono",
		"amount":"100000"
	}
}
```

