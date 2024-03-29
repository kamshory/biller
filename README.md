# Chapter 1 – Introduction

## 1.1 Protocol

AltoPay Biller API used the HTTP protocol version 1.1. The method used for the request is `POST` or `PUT`.

## 1.2 Messge Format

The message format used is JavaScript Object Notation (JSON). JSON is sent through the request body. The request body length must be included in the header.

## 1.3 Request Headers

HTTP headers allow the client and the server to pass additional information with the request or the response. An HTTP header consists of its case-insensitive name followed by a colon "`:`", then by its value (without line breaks). Leading white space before the value is ignored. Some of headers used to make Inquiry Request and Payment are as follows:

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
**Parameter Description**

| Parameter | Description | Example |
| -- | -- | -- |
| method | Request Method (GET, PUT, POST). | POST |
| path | Relative path of URL | /biller/json |
| body | Original request body without any modification | {“command”:”inquiry”, “data”:{}} |
| timestamp | ISO Time stamp. | 2019-07-07T23:59:59.999Z |
| apikey | API Key. It will be different for each client. |   |
| password | Client password. Client can change this password anytime. |   |

**Note:**
a. timestamp and apikey are sent on request header.
b. password is not sent on request header.

5. **Content-type**
**`Content-type`** is filled with `json/application`.

5. **Content-encoding**
**`Content-type`** is filled with `identity`.

4. **Content-length**
**`Content-length`** is the length of the request body in bytes. The client must calculate the length of the  request body before sending it to the server. Therefore, the merchant may not send a request with partial content.

6. **Host**
**`Host`** is server address that will be accessed by merchant. This address contains: domain/subdomain, domain/subdomain and port, IP address, IP address and port. For example ``dev.altopay.id:9090``.

## 1.4 Request Body

Every request body always contains properties:

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

# Chapter 3 - Transaction Flow

# Chapter 4 - Product and Service

The list of products and services attached separately from this document is due to the large and growing numbers in accordance with the development of billers. Each product has a different message format according to the type of bill and also the policy of each biller.

The following are product categories based on the transaction process.

| No | Code | Description |
| -- | -- | -- |
| 1 | 1004 | Postpaid Electricity |
| 2 | 1005 | Prepaid Electricity |
| 3 | 1003 | Nonusage Electricity |
| 4 | 1010 | Prepaid Pone Cell |
| 5 | 1008 | Postpaid Phone Cell |
| 6 | 1009 | Mobile Data |
| 7 | 1007 | Water |
| 8 | 1006 | General Bill |

# Chapter 5 - General Message Format

The message format on one product will be different from the message format on other products. However, in general, each product has a uniform message format. This chapter will explain the general message format for all products.

## 5.1 Inquiry

Inquiry is an electronic transaction to find out billing information for certain customer ID of the biller. Some products has no inquiry and some products require inquiry. 

### 5.1.1 Request

#### a. Header

**Accept**

The `Accept` request HTTP header advertises which content types, expressed as MIME types, the client is able to understand. Using content negotiation, the server then selects one of the proposals, uses it and informs the client of its choice with the `Content-Type` response header.

**Accept-encoding**

`Accept-encoding` is encoding that can be accepted by client. Acceptable encoding, namely:

1. `identity` (without compression)
2. `gzip` (with GZIP compression)

**Content-type**

The `Content-type` entity header indicates the size of the entity-body, in bytes, sent to the recipient.

**Content-Encoding**

The `Content-Encoding` entity header is used to compress the media-type. When present, its value indicates which encodings were applied to the entity-body. It lets the client know how to decode in order to obtain the media-type referenced by the `Content-Type` header.

**Content-length**

The `Content-length` entity header indicates the size of the entity-body, in bytes, sent to recipient.

**Connection**

The `Connection` general header controls whether or not the network connection stays open after the current transaction finishes. If the value sent is `keep-alive`, the connection is persistent and not closed, allowing for subsequent requests to the same server to be done.

**User-agent**

The `User-Agent` request header contains a characteristic string that allows the network protocol peers to identify the application type, operating system, software vendor or software version of the requesting software user agent.  

**Host**

`Host` is the name of the server accessed by client. From this header, the AltoPay can find out how client is connected to AltoPay.

**X-api-key**

The `X-api-key` is used to identity the client.

See **Request Headers** section on Chapter 1.

**X-timestamp**

The `X-timestamp` is one of signature component of the request to guarantee data integrity.

See **Request Headers** section on Chapter 1.

**X-signature**

The `X-signature` is signature of the request to guarantee data integrity. The parameters of the `X-Signature` are:

1. metod
2. path
3. body
4. timestamp
5. apikey
6. password

See **Request Headers** section on Chapter 1.

#### b. Body

| No | Parameter | Type | Description | Note |
| -- | -- | -- | -- | -- |
| 1 | command | String | Transaction command. See **Transaction Command List** | Mandatory |
| 2 | product_code | Numeric, can begin with 0 | Product code | Mandatory |
| 3 | process_category | Numeric, can begin with 0, alternative of product code | Process category | Conditional |
| 4 | data | Object | Container of the transaction data | Mandatory |
| 5 | data.customer_id | Numeric, can begin with 0 | Customer ID | Mandatory |
| 6 | data.date_time | String | Transmission date and time in GMT (Format: yyyy-MM-d’T’HH:mm:ss.SSS’Z’) | Mandatory |
| 7 | data.reference_number | String (32) | Reference number | Mandatory |

Note: The data length above is the maximum allowed. Shorter will be better.

### 5.1.2 Response

#### a. Header

**Content-type**

The `Content-type` entity header indicates the size of the entity-body, in bytes, sent to the recipient.

**Content-encoding**

`Content-encoding` is content encoding that sent by AltoPay. Content encoding, namely:

1. `identity` (without compression)
2. `gzip` (with GZIP compression)

**Content-length**

The `Content-length` entity header indicates the size of the entity-body, in bytes, sent to recipient.

**Date**

The `Date` general HTTP header contains the date and time at which the message was originated.

#### b. Body

| No | Parameter | Type | Description | Note |
| -- | -- | -- | -- | -- |
| 1 | command | String | Transaction command | Mandatory |
| 2 | product_code | Numeric, can begin with 0 | Product code | Mandatory |
| 3 | response_code | Numeric, can begin with 0 | Product code | Mandatory |
| 4 | response_text | String | Description of the response transaction | Mandatory |
| 5 | data | Object | Container of the transaction data | Mandatory |
| 6 | data.customer_id | Numeric, can begin with 0 | Customer ID | Mandatory |
| 7 | data.customer_name | String (32) | Customer name | Mandatory |
| 8 | data.amount | Numeric | Bill amount | Mandatory |
| 9 | data.date_time | String | Transmission date and time in GMT (Format: yyyy-MM-d’T’HH:mm:ss.SSS’Z’) | Mandatory |
| 10 | data.reference_number | String (32) | Reference number | Mandatory |

Note: The data length above is the maximum allowed. Shorter will be better.

## 5.2 Payment

Payment is an electronic transaction to pay the bills for certain customer ID of the biller.

### 5.2.1 Request

#### a. Header

**Accept**

The `Accept` request HTTP header advertises which content types, expressed as MIME types, the client is able to understand. Using content negotiation, the server then selects one of the proposals, uses it and informs the client of its choice with the Content-Type response header.

**Accept-encoding**

`Accept-encoding` is encoding that can be accepted by client. Acceptable encoding, namely:

1. `identity` (without compression)
2. `gzip` (with GZIP compression)

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

`Host` is the name of the server accessed by client. From this header, the AltoPay can find out how client is connected to AltoPay.

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

| No | Parameter | Type | Description | Note |
| -- | -- | -- | -- | -- |
| 1 | command | String | Transaction command | Mandatory |
| 2 | product_code | Numeric, can begin with 0 | Product code | Mandatory |
| 3 | process_category | Numeric, can begin with 0, alternative of product code | Process category | Conditional |
| 4 | data | Object | Container of the transaction data | Mandatory |
| 5 | data.customer_id | Numeric, can begin with 0 | Customer ID | Mandatory |
| 6 | data.customer_name | String | Customer ID | Mandatory for several billers |
| 7 | data.amount | Numeric | Payment amount | For several product, amount must be same with amount on inquiry response |
| 8 | data.date_time | String | Transmission date and time in GMT (Format: yyyy-MM-d’T’HH:mm:ss.SSS’Z’) | Mandatory |
| 9 | data.fwd_reference_number | String (32) | Reference number | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 8 | data.fwd_stan | Numeric | Payment amount | For several product, fwd_stan must be same with fwd_stan on inquiry response |

### 5.2.2 Response

#### a. Header

**Content-type**

The `Content-type` entity header indicates the size of the entity-body, in bytes, sent to the recipient.

**Content-encoding**

`Content-encoding` is content encoding that sent by AltoPay. Content encoding, namely:

1. `identity` (without compression)
2. `gzip` (with GZIP compression)

**Content-length**

The `Content-length` entity header indicates the size of the entity-body, in bytes, sent to recipient.

**Date**

The `Date` general HTTP header contains the date and time at which the message was originated.

#### b. Body

| No | Parameter | Type | Description | Note |
| -- | -- | -- | -- | -- |
| 1 | command | String | Transaction command | Mandatory |
| 2 | product_code | Numeric, can begin with 0 | Product code | Mandatory |
| 3 | process_category | Numeric, can begin with 0, alternative of product code | Process category | Conditional |
| 4 | data | Object | Container of the transaction data | Mandatory |
| 5 | data.customer_id | Numeric, can begin with 0 | Customer ID | Mandatory |
| 6 | data.customer_name | String | Customer ID | Mandatory for several billers |
| 7 | data.amount | Numeric | Payment amount | For several product, amount must be same with amount on inquiry response |
| 8 | data.date_time | String | Transmission date and time in GMT (Format: yyyy-MM-d’T’HH:mm:ss.SSS’Z’) | Mandatory |
| 9 | data.reference_number | String (32) | Reference number | Mandatory |
| 10 | data.fwd_reference_number | String (32) | Reference number | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 11 | data.fwd_stan | Numeric | Payment amount | For several product, fwd_stan must be same with fwd_stan on inquiry response |

Note: The data length above is the maximum allowed. Shorter will be better.

# Chapter 6 – Product Message Format

Each product has a different message format depending on each biller. AltoPay adjusts the message format for each product according to its needs. Some products have far more data attributes than other products because they are related to biller policy.

Some products that have the same attributes and processes are combined into one product category, namely general billing. Some more products are made with special categories because they have very big differences.

Every message is always listed one of _product_code_ and _category_process. category_process_ is only used for prepaid cellular phone top up and postpaid cell phone bill payments.

## 6.1 Prepaid Electricity

Prepaid electricity is a product of the _Perusahaan Listrik Negara_ (PLN) where customers can use electricity in accordance with the quota that has been purchased. By purchasing a power quota, customers will get a token that can be entered into the power controller.

The customer must enter the customer ID or meter number at the time of purchase because the generated token will be adjusted to the customer's meter number.

To prevent mistakes when purchasing, PLN requires customers to conduct an inquiry to confirm whether the customer number or meter number entered is correct.

### 6.1.1 Inquiry Request

**Message Sample**

```http
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

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | data.date_time | String | 23 | Transmission date and time in GMT (Format: yyyy-MM-d’T’HH:mm:ss.SSS’Z’) |
| 4 | data.reference_number | String | 32 | Reference number |
| 5 | data.meter_id | String | 11 | Meter ID |
| 6 | data.customer_id | String | 12 | Customer ID |
| 7 | data.id_selector | Decimal | 1 | ID Selector. 0 if the reference is meter ID and 1 if the reference is customer ID. |


### 6.1.2 Inquiry Response

**Message Sample**

```http
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

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | response_code | String |  |  |
| 4 | response_text | String |  |  |
| 5 | data.time_stamp | String | 23 | Response time stamp |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 8 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 9 | data.meter_id | String | 11 | Meter ID |
| 10 | data.customer_id | String | 12 | Customer ID |
| 11 | data.id_selector | Decimal | 1 | ID Selector. 0 if the reference is meter ID and 1 if the reference is customer ID. |
| 12 | data.biller_reference_number | String | 32 | Biller reference number |
| 13 | data.ba_reference_number | String | 32 | Biller aggregator reference number |
| 14 | data.customer_name | String | 25 | Customer Name |
| 15 | data.ariff | String | 4 | Tariff applied |
| 16 | data.ceiling | String | 9 | Maximum power limits that can be used |
| 17 | data.distribution_code_print | String | 2 | Distribution Code for Print |
| 18 | data.service_unit_name_print | String | 5 | Service Unit Name for Print |
| 19 | data.service_unit_phone_print | String | 15 | Service Unit Phone for Print |
| 20 | data.maximum_power_unit_print | String | 5 | Maximum power unit for print |
| 21 | data.total_repeat_print | String | 1 | Total Repeat for Print |
| 22 | data.power_purchase_unsold_print_1 | Decimal | 11 | Power purchase unsold 1 for print |
| 23 | data.power_purchase_unsold_print_2 | Decimal | 11 | Power purchase unsold 2 for print |
### 6.1.3 Purchase Request

**Message Sample**

```http
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

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | data.date_time | String (23) |  | Response time stamp |
| 4 | data.data.reference_number | String (32) |  | Reference number |
| 5 | data.meter_id | String | 11 | Meter ID |
| 6 | data.customer_id | String | 12 | Customer ID |
| 7 | data.id_selector | Decimal | 1 | ID Selector. 0 if the reference is meter ID and 1 if the reference is customer ID. |
| 8 | data.biller_reference_number | String | 32 | Biller reference number |
| 9 | data.ba_reference_number | String | 32 | Biller aggregator reference number |
| 10 | data.customer_name | String | 25 | Customer Name |
| 11 | data.tariff | String | 4 | Tariff applied |
| 12 | data.ceiling | String | 9 | Maximum power limits that can be used |
| 13 | data.purchase_option | Decimal | 1 | Purchase Option |
| 14 | data.distribution_code_print | String | 2 | Distribution Code for Print |
| 15 | data.service_unit_name_print | String | 5 | Service Unit Name for Print |
| 16 | data.service_unit_phone_print | String | 15 | Service Unit Phone for Print |
| 17 | data.maximum_power_unit_print | String | 5 | Maximum power unit for print |
| 18 | data.total_repeat_print | String | 1 | Total Repeat for Print |
| 19 | data.power_purchase_unsold_print_1 | Decimal | 11 | Power purchase unsold 1 for print |
| 20 | data.power_purchase_unsold_print_2 | Decimal | 11 | Power purchase unsold 2 for print |
| 21 | data.amount | Decimal | 12 | Amount |
| 22 | data.merchant_type | String | 4 | Merchant type of the channel where customer pay the bill |
| 23 | data.locket_code | String | 32 | Locket code where customer pay |
| 24 | data.locket_name | String | 30 | Locket name where customer pay |
| 25 | data.locket_address | String | 50 | Locket address where customer pay |
| 26 | data.locket_phone | String | 18 | Locket phone where customer pay |

### 6.1.4 Purchase Response

**Message Sample**

```http
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

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | response_code | String |  | Response code |
| 4 | response_text | String |  | Response text |
| 5 | data.time_stamp | String | 23 | Response time stamp |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 8 | data.fwd_stan | Numeric |  | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 9 | data.meter_id | Decimal | 11 | Meter ID |
| 10 | data.customer_id | Decimal | 12 | Customer ID |
| 11 | data.id_selector | Decimal | 1 | ID Selector. 0 if the reference is meter ID and 1 if the reference is customer ID. |
| 12 | data.biller_reference_number | String | 32 | Biller reference number |
| 13 | data.ba_reference_number | String | 32 | Biller aggregator reference number |
| 14 | data.vending_receipt_number | Decimal | 8 | Vending Receipt Number |
| 15 | data.customer_name | String | 25 | Customer Name |
| 16 | data.tariff | String | 4 | Tariff applied |
| 17 | data.ceiling | String | 9 | Maximum power limits that can be used |
| 18 | data.purchase_option | String | 1 | Purchase Option |
| 19 | data.decimal_admin_fee | Decimal | 1 | Decimal Admin Fee |
| 20 | data.admin_fee | Decimal | 10 | Admin fee for the transaction |
| 21 | data.decimal_stamp | Decimal | 1 | Decimal Stamp |
| 22 | data.stamp | Decimal | 10 | Stamp |
| 23 | data.decimal_ppn | Decimal | 1 | Decimal PPN |
| 24 | data.ppn | Decimal | 10 | PPN |
| 25 | data.decimal_ppju | Decimal | 1 | Decimal PPJU |
| 26 | data.ppju | Decimal | 10 | PPJU |
| 27 | data.decimal_installment | Decimal | 1 | Decimal Installment |
| 28 | data.installment | Decimal | 10 | Installment |
| 29 | data.decimal_power | Decimal | 1 | Decimal Power |
| 30 | data.power | Decimal | 12 | Power |
| 31 | data.decimal_unit_price | Decimal | 1 | Decimal Unit Price |
| 32 | data.unit_price | Decimal | 10 | Unit Price |
| 33 | data.token | String | 20 | Token of electrical quota |
| 34 | data.date_paid_off | String | 14 | Date Paid Off |
| 35 | data.distribution_code | String | 2 | Distribution Code |
| 36 | data.service_unit_name | String | 5 | Service Unit Name |
| 37 | data.service_unit_phone | String | 15 | Service Unit Phone |
| 38 | data.maximum_power_unit | Decimal | 5 | Maximum power unit |
| 39 | data.total_repeat | Decimal | 1 | Total Repeat |
| 40 | data.power_purchase_unsold | Decimal | 11 | Power Purchase Unsold |
| 41 | data.amount | Decimal | 12 | Amount |

### 6.1.5 Prepaid Advice Request


If the connection is lost when the token has been generated by the vending machine but has not been received by the customer, the customer can request the token again by sending a advice message. The same thing happens if the token has been received by the customer but is lost while the token has not been entered into the power controller.

To request a return token that has been purchased, the customer must send a advice message. The parameters of the advice message are as follows:
1. `customer_id`
2. `meter_id`
3. `id_selector`
4. `adv_reference_number`

`adv_reference_number` is the reference number that was sent when making a purchase.
If the customer does not know the reference number, AltoPay Biller will select the last transaction and include all reference numbers and transaction times on that day. Customers can choose transactions from the options provided if the requested token is not the last token of the day.

**Message Sample**

```http
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
Content-length: 274

{
	"command":"advice",
	"product_code":"00500050001",
	"data":{
		"date_time":"2019-10T23:58:59.987Z",
		"reference_number":"1234567891",
		"customer_id":"149999999911",
		"meter_id":"14987654321",
		"id_selector":"0",
		"adv_reference_number":"000000002161"
	}
}
```

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String | | Transaction command |
| 2 | product_code | Numeric, can begin with 0 | | Product code |
| 3 | data.date_time | String | 23 | Transmission date and time in GMT (Format: yyyy-MM-d’T’HH:mm:ss.SSS’Z’) |
| 4 | data.reference_number | String | 32 | Reference number |
| 5 | adv_reference_number | String | 32 | Advice reference number. If this reference number is not provided by customer, AltoPay Biller will choose last transaction |
| 6 | data.meter_id | String | 11 | Meter ID |
| 7 | data.customer_id | String | 12 | Customer ID |
| 8 | data.id_selector | Decimal | 1 | ID Selector. 0 if the reference is meter ID and 1 if the reference is customer ID. |
| 9 | data.merchant_type | Numeric | 4 | Merchant type of the channel where customer pay the bill |
| 10 | data.locket_code | String | 32 | Locket code where customer pay |
| 11 | data.locket_name | String | 30 | Locket name where customer pay |
| 12 | data.locket_address | String | 50 | Locket address where customer pay |
| 13 | data.locket_phone | String | 18 | Locket phone where customer pay |

### 6.1.6 Prepaid Advice Response

**Message Sample**

```http
HTTP/1.1 200 OK
Content-type: application/json
Content-encoding: identity
Date: Thu, 8 Aug 2019 08:08:08 GMT
Content-length: 1819

{
	"command": "advice",
	"product_code": "00500050001",
	"response_code": "00",
	"biller_message": "Sukses",
	"biller_response": "Sukses",
	"response_text": "Sukses",
	"data": {
		"customer_id": "149999999911",
		"meter_id": "14987654321",
		"id_selector": "0",
		"customer_name": "Kamshory",
		"purchase_option": "0",
		"time_stamp": "2019-08-08T05:15:24.238Z",
		"power_purchase_unsold": "0",
		"reference_number": "28",
		"fwd_reference_number": "00000000869",
		"fwd_stan": "000025",
		"ba_reference_number": "764937988121A58D555D849F6A01B2BE",
		"biller_reference_number": "FC032C2F8B26C0000253C22367005D81",
		"amount": "5000000",
		"stamp": "0",
		"decimal_stamp": "0",
		"ppn": "0",
		"decimal_ppn": "0",
		"ppju": "0"
		"decimal_ppju": "0",
		"power": "0",
		"decimal_power": "0",
		"installment": "0",
		"decimal_installment": "0",
		"unit_price": "0",
		"decimal_unit_price": "0",
		"admin_fee": "0",
		"decimal_admin_fee": "0",
		"maximum_power_unit": "0",
		"transaction_currency_code": "IDR",
		"service_unit_name": "",
		"vending_receipt_number": "HAMDANIE",
		"tariff": "R1",
		"ceiling": "000000900",
		"service_unit_phone": "",
		"distribution_code": "",
		"advice_list": [
			{
				"reference_number": "00000000805",
				"date_time": "2019-08-08T06:28:01.000Z"
			},
			{
				"reference_number": "00000000804",
				"date_time": "2019-08-08T05:15:24.000Z"
			},
			{
				"reference_number": "00000000803",
				"date_time": "2019-08-08T05:15:21.000Z"
			},
			{
				"reference_number": "00000000802",
				"date_time": "2019-08-08T05:15:08.000Z"
			},
			{
				"reference_number": "00000000801",
				"date_time": "2019-08-08T05:15:05.000Z"
			}
		],
		"total_repeat": "0",
		"token": "53684812841284184121",
		"date_paid_off": ""
	}
}
```

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | response_code | String |  | Response code |
| 4 | response_text | String |  | Response text |
| 5 | data.time_stamp | String | 23 | Response time stamp |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 8 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 9 | data.meter_id | Decimal | 11 | Meter ID |
| 10 | data.customer_id | Decimal | 12 | Customer ID |
| 11 | data.id_selector | Decimal | 1 | ID Selector. 0 if the reference is meter ID and 1 if the reference is customer ID. |
| 12 | data.biller_reference_number | String | 32 | Biller reference number |
| 13 | data.ba_reference_number | String | 32 | Biller aggregator reference number |
| 14 | data.vending_receipt_number | Decimal | 8 | Vending Receipt Number |
| 15 | data.customer_name | String | 25 | Customer Name |
| 16 | data.tariff | String | 4 | Tariff applied |
| 17 | data.ceiling | String | 9 | Maximum power limits that can be used |
| 18 | data.purchase_option | Decimal | 1 | Purchase Option |
| 19 | data.decimal_admin_fee | Decimal | 1 | Decimal Admin Fee |
| 20 | data.admin_fee | Decimal | 10 | Admin fee for the transaction |
| 21 | data.decimal_stamp | Decimal | 1 | Decimal Stamp |
| 22 | data.stamp | Decimal | 10 | Stamp |
| 23 | data.decimal_ppn | Decimal | 1 | Decimal PPN |
| 24 | data.ppn | Decimal | 10 | PPN |
| 25 | data.decimal_ppju | Decimal | 1 | Decimal PPJU |
| 26 | data.ppju | Decimal | 10 | PPJU |
| 27 | data.decimal_installment | Decimal | 1 | Decimal Installment |
| 28 | data.installment | Decimal | 10 | Installment |
| 29 | data.decimal_power | Decimal | 1 | Decimal Power |
| 30 | data.power | Decimal | 12 | Power |
| 31 | data.decimal_unit_price | Decimal | 1 | Decimal Unit Price |
| 32 | data.unit_price | Decimal | 10 | Unit Price |
| 33 | data.token | String | 20 | Token of electrical quota |
| 34 | data.date_paid_off | String | 14 | Date Paid Off |
| 35 | data.distribution_code | String | 2 | Distribution Code |
| 36 | data.service_unit_name | String | 5 | Service Unit Name |
| 37 | data.service_unit_phone | String | 15 | Service Unit Phone |
| 38 | data.maximum_power_unit | Decimal | 5 | Maximum power unit |
| 39 | data.total_repeat | Decimal | 1 | Total Repeat |
| 40 | data.power_purchase_unsold | Decimal | 11 | Power Purchase Unsold |
| 41 | data.amount | Decimal | 12 | Amount |


## 6.2 Postpaid Electrictity

### 6.2.1 Inquiry Request

**Message Sample**

```http
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
Content-length: 274

{
	"command":"inquiry",
	"product_code":"00500050001",
	"data":{
		"date_time":"2019-10T23:58:59.987Z",
		"customer_id":"149999999911",
		"reference_number":"000000002161"
	}
}
```

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String | | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | data.date_time | String | 23 | Response time stamp |
| 4 | data.reference_number | String | 32 | Reference number |
| 6 | data.customer_id | String | 12 | Customer ID |

### 6.2.2 Inquiry Response

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | response_code | String |  | Response code |
| 4 | response_text | String |  | Response text |
| 5 | data.time_stamp | String | 23 | Response time stamp |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 8 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 9 | data.customer_id | String | 12 | Customer ID |
| 10 | data.bill_count | Decimal | 1 | Count of bills to pay |
| 11 | data.outstanding_bill_count | Decimal | 2 | Count of outstanding bill to pay |
| 12 | data.biller_reference_number | String | 32 | Biller reference number |
| 13 | data.customer_name | String | 25 | Customer Name |
| 14 | data.service_unit_name | String | 5 | Service Unit Name |
| 15 | data.service_unit_phone | String | 15 | Service Unit Phone |
| 16 | data.tariff | String | 4 | Tariff applied |
| 17 | data.ceiling | Decimal | 9 | Maximum power limits that can be used |
| 18 | data.admin_fee | Decimal | 9 | Admin fee for the transaction |
| 19 | data.bill_period_1 | String | 6 | The first bill period |
| 20 | data.due_date_1 | String | 8 | The first period billing due date |
| 21 | data.bill_record_date_1 | String | 8 | Date of the first period of electricity usage |
| 22 | data.bill_amount_1 | Decimal | 12 | Total bill amount for the first period |
| 23 | data.incentive_sign_1 | String | 1 | Incentive sign of the first bills period |
| 24 | data.incentive_amount_1 | Decimal | 10 | Incentive amount of the first bills period |
| 25 | data.ppn_1 | Decimal | 10 | PPN for period 1 |
| 26 | data.late_fees_1 | Decimal | 12 | Late fee for period 1 |
| 27 | data.lwbp_before_1 | Decimal | 8 | LWBP before period 1 |
| 28 | data.lwbp_after_1 | Decimal | 8 | LWBP after for period 1 |
| 29 | data.wbp_before_1 | Decimal | 8 | WBP after for period 1 |
| 30 | data.wbp_after_1 | Decimal | 8 | BWP after for period 1 |
| 31 | data.kvarh_before_1 | Decimal | 8 | KVARH before for period 1 |
| 32 | data.kvarh_after_1 | Decimal | 8 | KVARH after for period 1 |
| 33 | data.bill_period_2 | String | 6 | The second bill period |
| 34 | data.due_date_2 | String | 8 | The second period billing due date |
| 35 | data.bill_record_date_2 | String | 8 | Date of the second period of electricity usage |
| 36 | data.bill_amount_2 | Decimal | 12 | Total bill amount for the second period |
| 37 | data.incentive_sign_2 | String | 1 | Incentive sign of the second bills period |
| 38 | data.incentive_amount_2 | String | 10 | Incentive amount of the second bills period |
| 39 | data.ppn_2 | Decimal | 10 | PPN for period 2 |
| 40 | data.late_fees_2 | Decimal | 12 | Late fees for period 2 |
| 41 | data.lwbp_before_2 | Decimal | 8 | LWBP before for period 2 |
| 42 | data.lwbp_after_2 | Decimal | 8 | LWBP after for period 2 |
| 43 | data.wbp_before_2 | Decimal | 8 | WBP before for period 2 |
| 44 | data.wbp_after_2 | Decimal | 8 | BWP after for period 2 |
| 45 | data.kvarh_before_2 | Decimal | 8 | KVARH before for period 2 |
| 46 | data.kvarh_after_2 | Decimal | 8 | KVARH after for period 2 |
| 47 | data.bill_period_3 | String | 6 | The third bill period |
| 48 | data.due_date_3 | String | 8 | The third period billing due date |
| 49 | data.bill_record_date_3 | String | 8 | Date of the third period of electricity usage |
| 50 | data.bill_amount_3 | Decimal | 12 | Total bill amount for the third period |
| 51 | data.incentive_sign_3 | String | 1 | Incentive sign of the third bills period |
| 52 | data.incentive_amount_3 | Decimal | 10 | Incentive amount of the third bills period |
| 53 | data.ppn_3 | Decimal | 10 | PPN for period 3 |
| 54 | data.late_fees_3 | Decimal | 12 | Late fees for period 3 |
| 55 | data.lwbp_before_3 | Decimal | 8 | LWBP before for period 3 |
| 56 | data.lwbp_after_3 | Decimal | 8 | LWBP after for period 3 |
| 57 | data.wbp_before_3 | Decimal | 8 | WBP before for period 3 |
| 58 | data.wbp_after_3 | Decimal | 8 | BWP after for period 3 |
| 59 | data.kvarh_before_3 | Decimal | 8 | KVARH before for period 3 |
| 60 | data.kvarh_after_3 | Decimal | 8 | KVARH after for period 3 |
| 61 | data.bill_period_4 | String | 6 | The fourth bill period |
| 62 | data.due_date_4 | String | 8 | The fourth period billing due date |
| 63 | data.bill_record_date_4 | String | 8 | Date of the fourth period of electricity usage |
| 64 | data.bill_amount_4 | Decimal | 12 | Total bill amount for the fourth period |
| 65 | data.incentive_sign_4 | String | 1 | Incentive sign of the fourth bills period |
| 66 | data.incentive_amount_4 | Decimal | 10 | Incentive amount of the fourth bills period |
| 67 | data.ppn_4 | Decimal | 10 | PPN for period 4 |
| 68 | data.late_fees_4 | Decimal | 12 | Late fees for period 4 |
| 69 | data.lwbp_before_4 | Decimal | 8 | LWBP before for period 4 |
| 70 | data.lwbp_after_4 | Decimal | 8 | LWBP after for period 4 |
| 71 | data.wbp_before_4 | Decimal | 8 | WBP before for period 4 |
| 72 | data.wbp_after_4 | Decimal | 8 | BWP after for period 4 |
| 73 | data.kvarh_before_4 | Decimal | 8 | LWBP before for period 4 |
| 74 | data.kvarh_after_4 | Decimal | 8 | KVARH after for period 4 |
| 75 | data.amount | Decimal | 12 | Amount |

### 6.2.3 Payment Request

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | data.date_time | String | 23 | Transmission date and time in GMT (Format: yyyy-MM-d’T’HH:mm:ss.SSS’Z’) |
| 4 | data.reference_number | String | 32 | Reference number |
| 5 | data.fwd_reference_number | String | 12 | Reference number forwarded from inquiry response to be used on related transaction |
| 6 | data.fwd_stan | String | 6 | STAN forwarded from inquiry response to be used on related transaction |
| 7 | data.customer_id | String | 12 | Customer ID |
| 8 | data.bill_count | Decimal | 1 | Count of bills to pay |
| 9 | data.payment_count | Decimal | 1 | The number of payment on the date |
| 10 | data.outstanding_bill_count | Decimal | 2 | Count of outstanding bill to pay |
| 11 | data.biller_reference_number | String | 32 | Biller reference number |
| 12 | data.customer_name | String | 25 | Customer Name |
| 13 | data.service_unit_name | String | 5 | Service Unit Name |
| 14 | data.service_unit_phone | String | 15 | Service Unit Phone |
| 15 | data.tariff | String | 4 | Tariff applied |
| 16 | data.ceiling | Decimal | 9 | Maximum power limits that can be used |
| 17 | data.admin_fee | Decimal | 9 | Admin fee for the transaction |
| 18 | data.bill_period_1 | String | 6 | The first bill period |
| 19 | data.due_date_1 | String | 8 | The first period billing due date |
| 20 | data.bill_record_date_1 | String | 8 | Date of the first period of electricity usage |
| 21 | data.bill_amount_1 | Decimal | 12 | Total bill amount for the first period |
| 22 | data.incentive_sign_1 | String | 1 | Incentive sign of the first bills period |
| 23 | data.incentive_amount_1 | Decimal | 10 | Incentive amount of the first bills period |
| 24 | data.ppn_1 | Decimal | 10 | PPN for periode 1 |
| 25 | data.late_fees_1 | Decimal | 12 | Late fee for period 1 |
| 26 | data.lwbp_before_1 | Decimal | 8 | LWBP before period 1 |
| 27 | data.lwbp_after_1 | Decimal | 8 | LWBP after for period 1 |
| 28 | data.wbp_before_1 | Decimal | 8 | WBP after for period 1 |
| 29 | data.wbp_after_1 | Decimal | 8 | BWP after for period 1 |
| 30 | data.kvarh_before_1 | Decimal | 8 | KVARH before for period 1 |
| 31 | data.kvarh_after_1 | Decimal | 8 | KVARH after for period 1 |
| 32 | data.bill_period_2 | String | 6 | The second bill period |
| 33 | data.due_date_2 | String | 8 | The second period billing due date |
| 34 | data.bill_record_date_2 | String | 8 | Date of the second period of electricity usage |
| 35 | data.bill_amount_2 | Decimal | 12 | Total bill amount for the second period |
| 36 | data.incentive_sign_2 | String | 1 | Incentive sign of the second bills period |
| 37 | data.incentive_amount_2 | String | 10 | Incentive amount of the second bills period |
| 38 | data.ppn_2 | Decimal | 10 | PPN for periode 2 |
| 39 | data.late_fees_2 | Decimal | 12 | Late fees for period 2 |
| 40 | data.lwbp_before_2 | Decimal | 8 | LWBP before for period 2 |
| 41 | data.lwbp_after_2 | Decimal | 8 | LWBP after for period 2 |
| 42 | data.wbp_before_2 | Decimal | 8 | WBP before for period 2 |
| 43 | data.wbp_after_2 | Decimal | 8 | BWP after for period 2 |
| 44 | data.kvarh_before_2 | Decimal | 8 | KVARH before for period 2 |
| 45 | data.kvarh_after_2 | Decimal | 8 | KVARH after for period 2 |
| 46 | data.bill_period_3 | String | 6 | The third bill period |
| 47 | data.due_date_3 | String | 8 | The third period billing due date |
| 48 | data.bill_record_date_3 | String | 8 | Date of the third period of electricity usage |
| 49 | data.bill_amount_3 | Decimal | 12 | Total bill amount for the third period |
| 50 | data.incentive_sign_3 | String | 1 | Incentive sign of the third bills period |
| 51 | data.incentive_amount_3 | Decimal | 10 | Incentive amount of the third bills period |
| 52 | data.ppn_3 | Decimal | 10 | PPN for periode 3 |
| 53 | data.late_fees_3 | Decimal | 12 | Late fees for period 3 |
| 54 | data.lwbp_before_3 | Decimal | 8 | LWBP before for period 3 |
| 55 | data.lwbp_after_3 | Decimal | 8 | LWBP after for period 3 |
| 56 | wbp_before_3 | Decimal | 8 | WBP before for period 3 |
| 57 | data.wbp_after_3 | Decimal | 8 | BWP after for period 3 |
| 58 | data.kvarh_before_3 | Decimal | 8 | KVARH before for period 3 |
| 59 | data.kvarh_after_3 | Decimal | 8 | KVARH after for period 3 |
| 60 | data.bill_period_4 | String | 6 | The fourth bill period |
| 61 | data.due_date_4 | String | 8 | The fourth period billing due date |
| 62 | data.bill_record_date_4 | String | 8 | Date of the fourth period of electricity usage |
| 63 | data.bill_amount_4 | Decimal | 12 | Total bill amount for the fourth period |
| 64 | data.incentive_sign_4 | String | 1 | Incentive sign of the fourth bills period |
| 65 | data.incentive_amount_4 | Decimal | 10 | Incentive amount of the fourth bills period |
| 66 | data.ppn_4 | Decimal | 10 | PPN for period 4 |
| 67 | data.late_fees_4 | Decimal | 12 | Late fees for period 4 |
| 68 | data.lwbp_before_4 | Decimal | 8 | LWBP before for period 4 |
| 69 | data.lwbp_after_4 | Decimal | 8 | LWBP after for period 4 |
| 70 | data.wbp_before_4 | Decimal | 8 | WBP before for period 4 |
| 71 | data.wbp_after_4 | Decimal | 8 | BWP after for period 4 |
| 72 | data.kvarh_before_4 | Decimal | 8 | LWBP before for period 4 |
| 73 | data.kvarh_after_4 | Decimal | 8 | KVARH after for period 4 |
| 74 | data.amount | Decimal | 12 | Amount |
| 75 | data.merchant_type | Numeric | 4 | Merchant type of the channel where customer pay the bill |
| 76 | data.locket_code | String | 32 | Locket code where customer pay |
| 77 | data.locket_name | String | 30 | Locket name where customer pay |
| 78 | data.locket_address | String | 50 | Locket address where customer pay |
| 79 | data.locket_phone | String | 18 | Locket phone where customer pay |

### 6.2.4 Payment Response

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | response_code | String |  | Response code |
| 4 | response_text | String |  | Response text |
| 5 | data.time_stamp | String |  | Response time stamp |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 8 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 9 | data.customer_id | String | 12 | Customer ID |
| 10 | data.bill_count | Decimal | 1 | Count of bills to pay |
| 11 | data.payment_count | Decimal | 1 | The number of payment on the date |
| 12 | data.outstanding_bill_count | Decimal | 2 | Count of outstanding bill to pay |
| 13 | data.biller_reference_number | String | 32 | Biller reference number |
| 14 | data.ba_reference_number | String | 32 | Biller aggregator reference number |
| 15 | data.customer_name | String | 25 | Customer Name |
| 16 | data.service_unit_name | String | 5 | Service Unit Name |
| 17 | data.service_unit_phone | String | 15 | Service Unit Phone |
| 18 | data.tariff | String | 4 | Tariff applied |
| 19 | data.ceiling | Decimal | 9 | Maximum power limits that can be used |
| 20 | data.admin_fee | Decimal | 9 | Admin fee for the transaction |
| 21 | data.bill_period_1 | String | 6 | The first bill period |
| 22 | data.due_date_1 | String | 8 | The first period billing due date |
| 23 | data.bill_record_date_1 | String | 8 | Date of the first period of electricity usage |
| 24 | data.bill_amount_1 | Decimal | 12 | Total bill amount for the first period |
| 25 | data.incentive_sign_1 | String | 1 | Incentive sign of the first bills period |
| 26 | data.incentive_amount_1 | Decimal | 10 | Incentive amount of the first bills period |
| 27 | data.ppn_1 | Decimal | 10 | PPN for period 1 |
| 28 | data.late_fees_1 | Decimal | 12 | Late fee for period 1 |
| 29 | data.lwbp_before_1 | Decimal | 8 | LWBP before period 1 |
| 30 | data.lwbp_after_1 | Decimal | 8 | LWBP after for period 1 |
| 31 | data.wbp_before_1 | Decimal | 8 | WBP after for period 1 |
| 32 | data.wbp_after_1 | Decimal | 8 | BWP after for period 1 |
| 33 | data.kvarh_before_1 | Decimal | 8 | KVARH before for period 1 |
| 34 | data.kvarh_after_1 | Decimal | 8 | KVARH after for period 1 |
| 35 | data.bill_period_2 | String | 6 | The second bill period |
| 36 | data.due_date_2 | String | 8 | The second period billing due date |
| 37 | data.bill_record_date_2 | String | 8 | Date of the second period of electricity usage |
| 38 | data.bill_amount_2 | Decimal | 12 | Total bill amount for the second period |
| 39 | data.incentive_sign_2 | String | 1 | Incentive sign of the second bills period |
| 40 | data.incentive_amount_2 | String | 10 | Incentive amount of the second bills period |
| 41 | data.ppn_2 | Decimal | 10 | PPN for period 2 |
| 42 | data.late_fees_2 | Decimal | 12 | Late fees for period 2 |
| 43 | data.lwbp_before_2 | Decimal | 8 | LWBP before for period 2 |
| 44 | data.lwbp_after_2 | Decimal | 8 | LWBP after for period 2 |
| 45 | data.wbp_before_2 | Decimal | 8 | WBP before for period 2 |
| 46 | data.wbp_after_2 | Decimal | 8 | BWP after for period 2 |
| 47 | data.kvarh_before_2 | Decimal | 8 | KVARH before for period 2 |
| 48 | data.kvarh_after_2 | Decimal | 8 | KVARH after for period 2 |
| 49 | data.bill_period_3 | String | 6 | The third bill period |
| 50 | data.due_date_3 | String | 8 | The third period billing due date |
| 51 | data.bill_record_date_3 | String | 8 | Date of the third period of electricity usage |
| 52 | data.bill_amount_3 | Decimal | 12 | Total bill amount for the third period |
| 53 | data.incentive_sign_3 | String | 1 | Incentive sign of the third bills period |
| 54 | data.incentive_amount_3 | Decimal | 10 | Incentive amount of the third bills period |
| 55 | data.ppn_3 | Decimal | 10 | PPN for period 3 |
| 56 | data.late_fees_3 | Decimal | 12 | Late fees for period 3 |
| 57 | data.lwbp_before_3 | Decimal | 8 | LWBP before for period 3 |
| 58 | data.lwbp_after_3 | Decimal | 8 | LWBP after for period 3 |
| 59 | data.wbp_before_3 | Decimal | 8 | WBP before for period 3 |
| 60 | data.wbp_after_3 | Decimal | 8 | BWP after for period 3 |
| 61 | data.kvarh_before_3 | Decimal | 8 | KVARH before for period 3 |
| 62 | data.kvarh_after_3 | Decimal | 8 | KVARH after for period 3 |
| 63 | data.bill_period_4 | String | 6 | The fourth bill period |
| 64 | data.due_date_4 | String | 8 | The fourth period billing due date |
| 65 | data.bill_record_date_4 | String | 8 | Date of the fourth period of electricity usage |
| 66 | data.bill_amount_4 | Decimal | 12 | Total bill amount for the fourth period |
| 67 | data.incentive_sign_4 | String | 1 | Incentive sign of the fourth bills period |
| 68 | data.incentive_amount_4 | Decimal | 10 | Incentive amount of the fourth bills period |
| 69 | data.ppn_4 | Decimal | 10 | PPN for period 4 |
| 70 | data.late_fees_4 | Decimal | 12 | Late fees for period 4 |
| 71 | data.lwbp_before_4 | Decimal | 8 | LWBP before for period 4 |
| 72 | data.lwbp_after_4 | Decimal | 8 | LWBP after for period 4 |
| 73 | data.wbp_before_4 | Decimal | 8 | WBP before for period 4 |
| 74 | data.wbp_after_4 | Decimal | 8 | BWP after for period 4 |
| 75 | data.kvarh_before_4 | Decimal | 8 | LWBP before for period 4 |
| 76 | data.kvarh_after_4 | Decimal | 8 | KVARH after for period 4 |
| 77 | data.payment_date | String | 8 | Payment date |
| 78 | data.payment_time | String | 6 | Payment date |
| 79 | data.amount | Decimal | 12 | Amount |

### 6.2.5 Advice Request

**Message Sample**

```http
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
Content-length: 274

{
	"command":"advice",
	"product_code":"00500050001",
	"data":{
		"date_time":"2019-10T23:58:59.987Z",
		"customer_id":"149999999911",
		"adv_reference_number":"000000002161"
	}
}
```

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String | | Transaction command |
| 2 | product_code | Numeric, can begin with 0 | | Product code |
| 3 | data.date_time | String | 23 | Transmission date and time in GMT (Format: yyyy-MM-d’T’HH:mm:ss.SSS’Z’) |
| 4 | data.reference_number | String | 32 | Reference number |
| 5 | data.customer_id | String | 12 | Customer ID |
| 6 | data.adv_reference_number | String | 32 | Advice reference number. If this reference number is not provided by customer, AltoPay Biller will choose last transaction |

### 6.2.6 Advice Response

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | response_code | String |  | Response code |
| 4 | response_text | String |  | Response text |
| 5 | data.time_stamp | String |  | Response time stamp |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 8 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 9 | data.customer_id | String | 12 | Customer ID |
| 10 | data.bill_count | Decimal | 1 | Count of bills to pay |
| 11 | data.payment_count | Decimal | 1 | The number of payment on the date |
| 12 | data.outstanding_bill_count | Decimal | 2 | Count of outstanding bill to pay |
| 13 | data.biller_reference_number | String | 32 | Biller reference number |
| 14 | data.ba_reference_number | String | 32 | Biller aggregator reference number |
| 15 | data.customer_name | String | 25 | Customer Name |
| 16 | data.service_unit_name | String | 5 | Service Unit Name |
| 17 | data.service_unit_phone | String | 15 | Service Unit Phone |
| 18 | data.tariff | String | 4 | Tariff applied |
| 19 | data.ceiling | Decimal | 9 | Maximum power limits that can be used |
| 20 | data.admin_fee | Decimal | 9 | Admin fee for the transaction |
| 21 | data.bill_period_1 | String | 6 | The first bill period |
| 22 | data.due_date_1 | String | 8 | The first period billing due date |
| 23 | data.bill_record_date_1 | String | 8 | Date of the first period of electricity usage |
| 24 | data.bill_amount_1 | Decimal | 12 | Total bill amount for the first period |
| 25 | data.incentive_sign_1 | String | 1 | Incentive sign of the first bills period |
| 26 | data.incentive_amount_1 | Decimal | 10 | Incentive amount of the first bills period |
| 27 | data.ppn_1 | Decimal | 10 | PPN for period 1 |
| 28 | data.late_fees_1 | Decimal | 12 | Late fee for period 1 |
| 29 | data.lwbp_before_1 | Decimal | 8 | LWBP before period 1 |
| 30 | data.lwbp_after_1 | Decimal | 8 | LWBP after for period 1 |
| 31 | data.wbp_before_1 | Decimal | 8 | WBP after for period 1 |
| 32 | data.wbp_after_1 | Decimal | 8 | BWP after for period 1 |
| 33 | data.kvarh_before_1 | Decimal | 8 | KVARH before for period 1 |
| 34 | data.kvarh_after_1 | Decimal | 8 | KVARH after for period 1 |
| 35 | data.bill_period_2 | String | 6 | The second bill period |
| 36 | data.due_date_2 | String | 8 | The second period billing due date |
| 37 | data.bill_record_date_2 | String | 8 | Date of the second period of electricity usage |
| 38 | data.bill_amount_2 | Decimal | 12 | Total bill amount for the second period |
| 39 | data.incentive_sign_2 | String | 1 | Incentive sign of the second bills period |
| 40 | data.incentive_amount_2 | String | 10 | Incentive amount of the second bills period |
| 41 | data.ppn_2 | Decimal | 10 | PPN for period 2 |
| 42 | data.late_fees_2 | Decimal | 12 | Late fees for period 2 |
| 43 | data.lwbp_before_2 | Decimal | 8 | LWBP before for period 2 |
| 44 | data.lwbp_after_2 | Decimal | 8 | LWBP after for period 2 |
| 45 | data.wbp_before_2 | Decimal | 8 | WBP before for period 2 |
| 46 | data.wbp_after_2 | Decimal | 8 | BWP after for period 2 |
| 47 | data.kvarh_before_2 | Decimal | 8 | KVARH before for period 2 |
| 48 | data.kvarh_after_2 | Decimal | 8 | KVARH after for period 2 |
| 49 | data.bill_period_3 | String | 6 | The third bill period |
| 50 | data.due_date_3 | String | 8 | The third period billing due date |
| 51 | data.bill_record_date_3 | String | 8 | Date of the third period of electricity usage |
| 52 | data.bill_amount_3 | Decimal | 12 | Total bill amount for the third period |
| 53 | data.incentive_sign_3 | String | 1 | Incentive sign of the third bills period |
| 54 | data.incentive_amount_3 | Decimal | 10 | Incentive amount of the third bills period |
| 55 | data.ppn_3 | Decimal | 10 | PPN for period 3 |
| 56 | data.late_fees_3 | Decimal | 12 | Late fees for period 3 |
| 57 | data.lwbp_before_3 | Decimal | 8 | LWBP before for period 3 |
| 58 | data.lwbp_after_3 | Decimal | 8 | LWBP after for period 3 |
| 59 | data.wbp_before_3 | Decimal | 8 | WBP before for period 3 |
| 60 | data.wbp_after_3 | Decimal | 8 | BWP after for period 3 |
| 61 | data.kvarh_before_3 | Decimal | 8 | KVARH before for period 3 |
| 62 | data.kvarh_after_3 | Decimal | 8 | KVARH after for period 3 |
| 63 | data.bill_period_4 | String | 6 | The fourth bill period |
| 64 | data.due_date_4 | String | 8 | The fourth period billing due date |
| 65 | data.bill_record_date_4 | String | 8 | Date of the fourth period of electricity usage |
| 66 | data.bill_amount_4 | Decimal | 12 | Total bill amount for the fourth period |
| 67 | data.incentive_sign_4 | String | 1 | Incentive sign of the fourth bills period |
| 68 | data.incentive_amount_4 | Decimal | 10 | Incentive amount of the fourth bills period |
| 69 | data.ppn_4 | Decimal | 10 | PPN for period 4 |
| 70 | data.late_fees_4 | Decimal | 12 | Late fees for period 4 |
| 71 | data.lwbp_before_4 | Decimal | 8 | LWBP before for period 4 |
| 72 | data.lwbp_after_4 | Decimal | 8 | LWBP after for period 4 |
| 73 | data.wbp_before_4 | Decimal | 8 | WBP before for period 4 |
| 74 | data.wbp_after_4 | Decimal | 8 | BWP after for period 4 |
| 75 | data.kvarh_before_4 | Decimal | 8 | LWBP before for period 4 |
| 76 | data.kvarh_after_4 | Decimal | 8 | KVARH after for period 4 |
| 77 | data.payment_date | String | 8 | Payment date |
| 78 | data.payment_time | String | 6 | Payment date |
| 79 | data.amount | Decimal | 12 | Amount |

### 6.2.7 Reversal Request

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | data.date_time | String | 23 | Transmission date and time in GMT (Format: yyyy-MM-d’T’HH:mm:ss.SSS’Z’) |
| 4 | data.reference_number | String | 32 | Reference number |
| 5 | data.customer_id | String | 12 | Customer ID |
| 6 | data.rev_reference_number | String | 32 | Reference number to be reversed. If this reference number is not provided by customer, AltoPay Biller will choose last transaction |

### 6.2.8 Reversal Response

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | response_code | String |  | Response code |
| 4 | response_text | String |  | Response text |
| 5 | data.time_stamp | String | 24 | Response date and time |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.fwd_reference_number | String | 32 | Forwarding reference number received from inquiry |
| 8 | data.fwd_stan | String | 6 | Forwarding STAN received from inquiry |
| 9 | data.customer_id | String | 12 | Customer ID |
| 10 | data.bill_count | Decimal | 1 | Count of bills to pay |
| 11 | data.payment_count | Decimal | 1 | The number of payment on the date |
| 12 | data.outstanding_bill_count | Decimal | 2 | Count of outstanding bill to pay |
| 13 | data.biller_reference_number | String | 32 | Biller reference number |
| 14 | data.ba_reference_number | String | 32 | Biller aggregator reference number |
| 15 | data.customer_name | String | 25 | Customer Name |
| 16 | data.service_unit_name | String | 5 | Service Unit Name |
| 17 | data.service_unit_phone | String | 15 | Service Unit Phone |
| 18 | data.tariff | String | 4 | Tariff applied |
| 19 | data.ceiling | Decimal | 9 | Maximum power limits that can be used |
| 20 | data.admin_fee | Decimal | 9 | Admin fee for the transaction |
| 21 | data.bill_period_1 | String | 6 | The first bill period |
| 22 | data.due_date_1 | String | 8 | The first period billing due date |
| 23 | data.bill_record_date_1 | String | 8 | Date of the first period of electricity usage |
| 24 | data.bill_amount_1 | Decimal | 12 | Total bill amount for the first period |
| 25 | data.incentive_sign_1 | String | 1 | Incentive sign of the first bills period |
| 26 | data.incentive_amount_1 | Decimal | 10 | Incentive amount of the first bills period |
| 27 | data.ppn_1 | Decimal | 10 | PPN for periode 1 |
| 28 | data.late_fees_1 | Decimal | 12 | Late fee for period 1 |
| 29 | data.lwbp_before_1 | Decimal | 8 | LWBP before period 1 |
| 30 | data.lwbp_after_1 | Decimal | 8 | LWBP after for period 1 |
| 31 | data.wbp_before_1 | Decimal | 8 | WBP after for period 1 |
| 32 | data.wbp_after_1 | Decimal | 8 | BWP after for period 1 |
| 33 | data.kvarh_before_1 | Decimal | 8 | KVARH before for period 1 |
| 34 | data.kvarh_after_1 | Decimal | 8 | KVARH after for period 1 |
| 35 | data.bill_period_2 | String | 6 | The second bill period |
| 36 | data.due_date_2 | String | 8 | The second period billing due date |
| 37 | data.bill_record_date_2 | String | 8 | Date of the second period of electricity usage |
| 38 | data.bill_amount_2 | Decimal | 12 | Total bill amount for the second period |
| 39 | data.incentive_sign_2 | String | 1 | Incentive sign of the second bills period |
| 40 | data.incentive_amount_2 | String | 10 | Incentive amount of the second bills period |
| 41 | data.ppn_2 | Decimal | 10 | PPN for periode 2 |
| 42 | data.late_fees_2 | Decimal | 12 | Late fees for period 2 |
| 43 | data.lwbp_before_2 | Decimal | 8 | LWBP before for period 2 |
| 44 | data.lwbp_after_2 | Decimal | 8 | LWBP after for period 2 |
| 45 | data.wbp_before_2 | Decimal | 8 | WBP before for period 2 |
| 46 | data.wbp_after_2 | Decimal | 8 | BWP after for period 2 |
| 47 | data.kvarh_before_2 | Decimal | 8 | KVARH before for period 2 |
| 48 | data.kvarh_after_2 | Decimal | 8 | KVARH after for period 2 |
| 49 | data.bill_period_3 | String | 6 | The third bill period |
| 50 | data.due_date_3 | String | 8 | The third period billing due date |
| 51 | data.bill_record_date_3 | String | 8 | Date of the third period of electricity usage |
| 52 | data.bill_amount_3 | Decimal | 12 | Total bill amount for the third period |
| 53 | data.incentive_sign_3 | String | 1 | Incentive sign of the third bills period |
| 54 | data.incentive_amount_3 | Decimal | 10 | Incentive amount of the third bills period |
| 55 | data.ppn_3 | Decimal | 10 | PPN for periode 3 |
| 56 | data.late_fees_3 | Decimal | 12 | Late fees for period 3 |
| 57 | data.lwbp_before_3 | Decimal | 8 | LWBP before for period 3 |
| 58 | data.lwbp_after_3 | Decimal | 8 | LWBP after for period 3 |
| 59 | data.wbp_before_3 | Decimal | 8 | WBP before for period 3 |
| 60 | data.wbp_after_3 | Decimal | 8 | BWP after for period 3 |
| 61 | data.kvarh_before_3 | Decimal | 8 | KVARH before for period 3 |
| 62 | data.kvarh_after_3 | Decimal | 8 | KVARH after for period 3 |
| 63 | data.bill_period_4 | String | 6 | The fourth bill period |
| 64 | data.due_date_4 | String | 8 | The fourth period billing due date |
| 65 | data.bill_record_date_4 | String | 8 | Date of the fourth period of electricity usage |
| 66 | data.bill_amount_4 | Decimal | 12 | Total bill amount for the fourth period |
| 67 | data.incentive_sign_4 | String | 1 | Incentive sign of the fourth bills period |
| 68 | data.incentive_amount_4 | Decimal | 10 | Incentive amount of the fourth bills period |
| 69 | data.ppn_4 | Decimal | 10 | PPN for period 4 |
| 70 | data.late_fees_4 | Decimal | 12 | Late fees for period 4 |
| 71 | data.lwbp_before_4 | Decimal | 8 | LWBP before for period 4 |
| 72 | data.lwbp_after_4 | Decimal | 8 | LWBP after for period 4 |
| 73 | data.wbp_before_4 | Decimal | 8 | WBP before for period 4 |
| 74 | data.wbp_after_4 | Decimal | 8 | BWP after for period 4 |
| 75 | data.kvarh_before_4 | Decimal | 8 | LWBP before for period 4 |
| 76 | data.kvarh_after_4 | Decimal | 8 | KVARH after for period 4 |
| 77 | data.reversal_date | String | 8 | Reversal date |
| 78 | data.reversal_time | String | 6 | Reversal date |
| 79 | data.amount | Decimal | 12 | Amount |

## 6.3 Nonusage Electrictity

### 6.3.1 Inquiry Request

**Message Sample**

```http
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
Content-length: 274

{
	"command":"inquiry",
	"product_code":"00500050003",
	"data":{
		"date_time":"2019-10T23:58:59.987Z",
		"customer_id":"149999999911",
		"registration_number":"5200112006074"
	}
}
```

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String | | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | data.date_time | String | 23 | Request time stamp |
| 4 | data.reference_number | String | 32 | Reference number |
| 6 | data.registration_number | String | 13 | Registration number |
| 7 | data.transaction_type | String | 25 | Transaction Type |

### 6.3.2 Inquiry Response

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | response_code | String |  | Response code |
| 4 | response_text | String |  | Response text |
| 5 | data.time_stamp | String | 23 | Response time stamp |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 8 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 9 | data.registration_number | String | 13 | Registration number |
| 10 | data.transaction_type | String | 25 | Transaction Type |
| 11 | data.registration_date | String | 8 | Registration date |
| 12 | data.registration_expire_date | String | 8 | Registration expiration date |
| 13 | data.customer_id | String | 12 | Customer ID |
| 14 | data.customer_name | String | 25 | Customer Name |
| 15 | data.ba_reference_number | String | 32 | Biller aggregator reference number |
| 16 | data.reference_id | String | 20 | Reference ID |
| 17 | data.total_amount_100 | Decimal | 17 |  |
| 18 | data.amount_100 | Decimal | 17 |  |
| 19 | data.admin_fee_100 | Decimal | 10 |  |
| 20 | data.amount | Decimal | 12 | Amount |

### 6.3.3 Payment Request

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | response_code | String |  | Response code |
| 4 | response_text | String |  | Response text |
| 5 | data.date_time | String | 23 | Request time stamp |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 8 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 9 | data.registration_number | String | 13 | Registration number |
| 10 | data.transaction_type | String | 25 | Transaction Type |
| 11 | data.registration_date | String | 8 | Registration date |
| 12 | data.registration_expire_date | String | 8 | Registration expiration date |
| 13 | data.customer_id | String | 12 | Customer ID |
| 14 | data.customer_name | String | 25 | Customer Name |
| 15 | data.ba_reference_number | String | 32 | Biller aggregator reference number |
| 16 | data.reference_id | String | 20 | Reference ID |
| 17 | data.total_amount_100 | Decimal | 17 |  |
| 18 | data.amount_100 | Decimal | 17 |  |
| 19 | data.admin_fee_100 | Decimal | 10 |  |
| 20 | data.amount | Decimal | 12 | Amount |
| 21 | data.merchant_type | Numeric | 4 | Merchant type of the channel where customer pay the bill |
| 22 | data.locket_code | String | 32 | Locket code where customer pay |
| 23 | data.locket_name | String | 30 | Locket name where customer pay |
| 24 | data.locket_address | String | 50 | Locket address where customer pay |
| 25 | data.locket_phone | String | 18 | Locket phone where customer pay |


### 6.3.4 Payment Response

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | response_code | String |  | Response code |
| 4 | response_text | String |  | Response text |
| 5 | data.time_stamp | String | 23 | Response time stamp |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 8 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 9 | registration_number | String | 13 |  |
| 10 | transaction_type | String | 25 |  |
| 11 | registration_date | String | 8 |  |
| 12 | registration_expire_date | String | 8 |  |
| 13 | customer_id | String | 12 | Customer ID |
| 14 | customer_name | String | 25 | Customer Name |
| 15 | reference_id | String | 32 |  |
| 16 | ba_reference_number | String | 20 | Biller aggregator reference number |
| 17 | total_amount_100 | Decimal | 17 |  |
| 18 | amount_100 | Decimal | 17 |  |
| 19 | admin_fee_100 | Decimal | 10 |  |
| 20 | payment_date | String | 8 | Payment date |
| 21 | payment_time | String | 6 | Payment date |
| 22 | data.amount | Decimal |  | Amount |


### 6.3.5 Advice Request

**Message Sample**

```http
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
Content-length: 274

{
	"command":"advice",
	"product_code":"00500050001",
	"data":{
		"date_time":"2019-10T23:58:59.987Z",
		"registration_number":"149999999911",
		"adv_reference_number":"000000002161"
	}
}
```

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String | | Transaction command |
| 2 | product_code | Numeric, can begin with 0 | | Product code |
| 3 | data.date_time | String | 23 | Transmission date and time in GMT (Format: yyyy-MM-d’T’HH:mm:ss.SSS’Z’) |
| 4 | data.reference_number | String | 32 | Reference number |
| 5 | data.registration_number | String | 13 | Registration number |
| 6 | data.adv_reference_number | String | 32 | Advice reference number. If this reference number is not provided by customer, AltoPay Biller will choose last transaction |
| 7 | data.date_time | String | 24 | Transmission date and time |

### 6.3.6 Advice Response

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | response_code | String |  | Response code |
| 4 | response_text | String |  | Response text |
| 5 | data.time_stamp | String | 23 | Response time stamp |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 8 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 9 | registration_number | String | 13 |  |
| 10 | transaction_type | String | 25 |  |
| 11 | registration_date | String | 8 |  |
| 12 | registration_expire_date | String | 8 |  |
| 13 | customer_id | String | 12 | Customer ID |
| 14 | customer_name | String | 25 | Customer Name |
| 15 | reference_id | String | 32 |  |
| 16 | ba_reference_number | String | 20 | Biller aggregator reference number |
| 17 | total_amount_100 | Decimal | 17 |  |
| 18 | amount_100 | Decimal | 17 |  |
| 19 | admin_fee_100 | Decimal | 10 |  |
| 20 | payment_date | String | 8 | Payment date |
| 21 | payment_time | String | 6 | Payment date |
| 22 | data.amount | Decimal |  | Amount |

### 6.3.7 Reversal Request

**Message Sample**

**Field Description**

| No | Type | Variable | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | String | command |  | Transaction command |
| 2 | Numeric, can begin with 0 | product_code |  | Product code |
| 3 | String | data.date_time | 23 | Transmission date and time in GMT (Format: yyyy-MM-d’T’HH:mm:ss.SSS’Z’) |
| 4 | String | data.reference_number | 32 | Reference number |
| 5 | String | data.registration_number | 13 | Registration number |
| 6 | String | data.fwd_reference_number | 32 | Forwarding reference number received from inquiry |
| 7 | String | data.fwd_stan | 6 | Forwarding STAN received from inquiry |
| 8 | String | data.date_time | 24 | Transmission date and time |
| 9 | String | data.registration_number | 13 | Registration number |
| 10 | String | data.transaction_type | 25 | Transaction type |
| 11 | String | data.registration_date | 8 | Registration date |
| 12 | String | data.registration_expire_date | 8 | Registration expiration date |
| 12 | String | data.customer_id | 12 | Customer ID |
| 13 | String | data.customer_name | 25 | Customer Name |
| 14 | String | data.ba_reference_number | 32 | Biller aggregator reference number |
| 15 | String | data.reference_id | 20 | Reference ID |
| 16 | Decimal | data.total_amount_100 | 17 |  |
| 17 | Decimal | data.amount_100 | 17 |  |
| 18 | Decimal | data.admin_fee_100 | 10 |  |
| 19 | Decimal | data.amount | 12 | Amount |

### 6.3.8 Reversal Response

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | response_code | String |  | Response code |
| 4 | response_text | String |  | Response text |
| 5 | data.time_stamp | String | 23 | Response time stamp |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.registration_number | String | 13 | Registration number |
| 8 | data.fwd_reference_number | String | 32 | Forwarding reference number received from inquiry |
| 9 | data.fwd_stan | String | 6 | Forwarding STAN received from inquiry |
| 10 | data.date_time | String | 24 | Transmission date and time |
| 11 | data.registration_number | String | 13 | Registration number |
| 12 | data.transaction_type | String | 25 | Transaction type |
| 13 | data.registration_date | String | 8 | Registration date |
| 14 | data.registration_expire_date | String | 8 | Registration expiration date |
| 15 | data.customer_id | String | 12 | Customer ID |
| 16 | data.customer_name | String | 25 | Customer name |
| 17 | data.ba_reference_number | String | 32 | Biller aggregator reference number |
| 18 | data.reference_id | String | 20 | Rererence ID |
| 19 | data.total_amount_100 | Decimal | 17 |  |
| 20 | data.amount_100 | Decimal | 17 |  |
| 21 | data.admin_fee_100 | Decimal | 10 |  |
| 22 | data.payment_date | String | 8 | Payment date |
| 23 | data.payment_time | String | 6 | Payment date |
| 24 | data.amount | Decimal | 12 | Amount |

## 6.4 Prepaid Cell Phone Credit

### Payment Request

**Message Sample**
```http
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

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | data.date_time | String | 23 | Response time stamp |
| 4 | data.reference_number | String | 32 | Reference number |
| 5 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 6 | data.fwd_stan | Numeric |  | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 7 | data.customer_id | String | 12 | Customer ID |
| 8 | data.amount | Decimal | 12 | Amount |
| 9 | data.merchant_type | Numeric | 4 | Merchant type of the channel where customer pay the bill |
| 10 | data.locket_code | String | 32 | Locket code where customer pay |
| 11 | data.locket_name | String | 30 | Locket name where customer pay |
| 12 | data.locket_address | String | 50 | Locket address where customer pay |
| 13 | data.locket_phone | String | 18 | Locket phone where customer pay |

### Payment Response

**Message Sample**

```http
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

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | response_code | String |  | Response code |
| 4 | response_text | String |  | Response text |
| 5 | data.date_time | String | 23 | Response time stamp |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 8 | data.fwd_stan | Numeric |  | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 9 | data.customer_id | String | 12 | Customer ID |
| 10 | data.customer_name | String | 40 | Customer Name |
| 11 | data.amount | Decimal | 12 | Amount |

## 6.5 Postpaid Cell Phone

### Inquiry Request

**Message Sample**

```http
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
	"command": "inquiry",
	"process_category": "1008",
	"data": {
		"reference_number": "1566985456",
		"date_time": "2019-08-28T16:44:16.000Z",
		"customer_id": "0811000000"
	}
}
```

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String | | Transaction command |
| 2 | product_code | Numeric, can begin with 0 | | Product code |
| 3 | data.date_time | String | 23 | Response time stamp |
| 4 | data.reference_number | String | 32 | Reference number |
| 5 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 6 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 7 | data.customer_id | String | 12 | Customer ID |

### Inquiry Response

**Message Sample**

```http
HTTP/1.1 200 OK
Content-type: application/json
Content-encoding: identity
Date: Thu, 8 Aug 2019 08:08:08 GMT
Content-length: 363

{
	"response_code": "00",
	"biller_message": "Sukses",
	"data": {
		"amount": "235000",
		"time_stamp": "2019-08-28T09:44:18.650Z",
		"fwd_stan": "000025",
		"customer_name": "Kamshory",
		"fwd_reference_number": "00000000869",
		"customer_id": "0811000000",
		"reference_number": "1566985456"
	},
	"biller_response": "Sukses",
	"response_text": "Sukses",
	"product_code": "00800080001",
	"command": "inquiry"
}

```

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 | | Product code |
| 3 | response_code | String | | Response code |
| 4 | response_text | String | | Response text |
| 5 | data.date_time | String | 23 | Response time stamp |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 8 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 9 | data.customer_id | String | 12 | Customer ID |
| 10 | data.customer_name | String | 40 | Customer Name |
| 11 | data.amount | Decimal | 12 | Amount |

### Payment Request

**Message Sample**

```http
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
	"command": "payment",
	"process_category": "1008",
	"data": {
		"locket_address": "Jalan Anggrek Neli Murni",
		"currency_code": "IDR",
		"locket_name": "ALTO",
		"merchant_type": "6021",
		"locket_code": "123",
		"reference_number": "1566985456",
		"locket_phone": "02199999",
		"date_time": "2019-08-28T16:46:08.000Z",
		"amount": "235000",
		"time_stamp": "2019-08-28T09:44:18.650Z",
		"fwd_stan": "000025",
		"customer_name": "Kamshory",
		"fwd_reference_number": "00000000869",
		"customer_id": "0811000000"
	}
}
```

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | data.date_time | String | 23 | Response time stamp |
| 4 | data.reference_number | String | 32 | Reference number |
| 5 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 6 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 7 | data.customer_id | String | 12 | Customer ID |
| 8 | data.customer_name | String | 40 | Customer Name |
| 9 | data.amount | Decimal | 12 | Amount |
| 10 | data.merchant_type | Numeric | 4 | Merchant type of the channel where customer pay the bill |
| 11 | data.locket_code | String | 32 | Locket code where customer pay |
| 12 | data.locket_name | String | 30 | Locket name where customer pay |
| 13 | data.locket_address | String | 50 | Locket address where customer pay |
| 14 | data.locket_phone | String | 18 | Locket phone where customer pay |

### Payment Response

**Message Sample**

```http
HTTP/1.1 200 OK
Content-type: application/json
Content-encoding: identity
Date: Thu, 8 Aug 2019 08:08:08 GMT
Content-length: 363

{
	"response_code": "00",
	"biller_message": "Sukses",
	"data": {
		"amount": "235000",
		"time_stamp": "2019-08-28T09:46:09.458Z",
		"fwd_stan": "000025",
		"customer_name": "Kamshory",
		"fwd_reference_number": "00000000869",
		"customer_id": "0811000000",
		"reference_number": "1566985456"
	},
	"biller_response": "Sukses",
	"response_text": "Sukses",
	"product_code": "00800080001",
	"command": "payment"
}
```

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 | | Product code |
| 3 | response_code | String | | Response code |
| 4 | response_text | String | | Response text |
| 5 | data.date_time | String | 23 | Response time stamp |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 8 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 9 | data.customer_id | String | 12 | Customer ID |
| 10 | data.customer_name | String | 40 | Customer Name |
| 11 | data.amount | Decimal | 12 | Amount |

## 6.6 Moble Data

### Payment Request

**Message Sample**
```http
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
	"process_category":"1011",
	"data":{
		"date_time":"2019-10T23:56:59.987Z",
		"reference_number":"1234567889",
		"customer_id":"08123456789",
		"amount":"100000"
	}
}
```

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | data.date_time | String | 23 | Response time stamp |
| 4 | data.reference_number | String | 32 | Reference number |
| 5 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 6 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 7 | data.customer_id | String | 12 | Customer ID |
| 8 | data.amount | Decimal | 12 | Amount |
| 9 | data.merchant_type | Numeric | 4 | Merchant type of the channel where customer pay the bill |
| 10 | data.locket_code | String | 32 | Locket code where customer pay |
| 11 | data.locket_name | String | 30 | Locket name where customer pay |
| 12 | data.locket_address | String | 50 | Locket address where customer pay |
| 13 | data.locket_phone | String | 18 | Locket phone where customer pay |

### Payment Response

**Message Sample**

```http
HTTP/1.1 200 OK
Content-type: application/json
Content-encoding: identity
Date: Thu, 8 Aug 2019 08:08:08 GMT
Content-length: 336

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
		"amount":"100000"
	}
}
```

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | response_code | String |  | Response code |
| 4 | response_text | String |  | Response text |
| 5 | data.date_time | String | 23 | Response time stamp |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 8 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 9 | data.customer_id | String | 12 | Customer ID |
| 10 | data.customer_name | String | 40 | Customer Name |
| 11 | data.amount | Decimal | 12 | Amount |

## 6.7 Water

Drinking Water Company is a company domiciled in each region. Each company has a different policy that causes the message format is also different. Each water company will form a separate product.

If the customer does not enter manatory data in both the inquiry and payment process, the biller might assume the transaction is invalid and will cause the transaction to fail. For this reason, every mandatory parameter must be filled in as appropriate.

### 6.7.1 Inquiry Request

**Message Sample**

```http
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
	"product_code":"99999999",
	"data":{
		"date_time":"2019-10T23:56:59.987Z",
		"reference_number":"1234567889",
		"customer_id":"081234567"
	}
}
```

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 | | Product code |
| 3 | data.date_time | String | 23 | Response time stamp |
| 4 | data.reference_number | String | 32 | Reference number |
| 5 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 6 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 7 | data.customer_id | String | 12 | Customer ID |

### 6.7.2 Inquiry Response

### 6.7.3 Payment Request

### 6.7.4 Payment Response

### 6.7.5 Advice Request

### 6.7.6 Advice Response

### 6.7.5 Reversal Request

### 6.7.8 Reversal Response



## 6.8 General Bill

### 6.8.1 Inquiry Request

**Message Sample**

```http
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

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 | | Product code |
| 3 | data.date_time | String | 23 | Response time stamp |
| 4 | data.reference_number | String | 32 | Reference number |
| 5 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 6 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 7 | data.customer_id | String | 12 | Customer ID |

### 6.8.2 Inquiry Response

**Message Sample**

```http
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

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 | | Product code |
| 3 | response_code | String | | Response code |
| 4 | response_text | String | | Response text |
| 5 | data.date_time | String | 23 | Response time stamp |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 8 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 9 | data.customer_id | String | 12 | Customer ID |
| 10 | data.customer_name | String | 40 | Customer Name |
| 11 | data.amount | Decimal | 12 | Amount |

### 6.8.3 Payment Request

**Message Sample**

```http
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

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 |  | Product code |
| 3 | data.date_time | String | 23 | Response time stamp |
| 4 | data.reference_number | String | 32 | Reference number |
| 5 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 6 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 7 | data.customer_id | String | 12 | Customer ID |
| 8 | data.customer_name | String | 40 | Customer Name |
| 9 | data.amount | Decimal | 12 | Amount |
| 10 | data.merchant_type | Numeric | 4 | Merchant type of the channel where customer pay the bill |
| 11 | data.locket_code | String | 32 | Locket code where customer pay |
| 12 | data.locket_name | String | 30 | Locket name where customer pay |
| 13 | data.locket_address | String | 50 | Locket address where customer pay |
| 14 | data.locket_phone | String | 18 | Locket phone where customer pay |

### 6.8.4 Payment Response

**Message Sample**

```http
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

**Field Description**

| No | Parameter | Type | Length | Description |
| -- | -- | -- | -- | -- |
| 1 | command | String |  | Transaction command |
| 2 | product_code | Numeric, can begin with 0 | | Product code |
| 3 | response_code | String | | Response code |
| 4 | response_text | String | | Response text |
| 5 | data.time_stamp | String | 23 | Response time stamp |
| 6 | data.reference_number | String | 32 | Reference number |
| 7 | data.fwd_reference_number | String | 32 | For several product, fwd_reference_number must be same with fwd_reference_number on inquiry response |
| 8 | data.fwd_stan | Numeric | 6 | For several product, fwd_stan must be same with fwd_stan on inquiry response |
| 9 | data.customer_id | String | 12 | Customer ID |
| 10 | data.customer_name | String | 40 | Customer Name |
| 11 | data.amount | Decimal | 12 | Amount |
