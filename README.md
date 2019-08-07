
# Chapter 1 – Introduction

## Protocol

AltoPay Biller API used the HTTP protocol version 1.1. The method used for the request is **POST** or **PUT**.

## Messge Format

The message format used is JavaScript Object Notation (JSON). JSON is sent through the request body. The request body length must be included in the header.

## Request Headers

HTTP headers allow the client and the server to pass additional information with the request or the response. An HTTP header consists of its case-insensitive name followed by a colon '`:`', then by its value (without line breaks). Leading white space before the value is ignored. Some of headers used to make Inquiry Request and Payment are as follows:

1. **X-Timestamp**
X-Timestamp is the time stamp of the transaction with ISO Format in UTC. Time stamp can be used to validate the transaction time.
For example **2019-12-31T23:59:59.999Z**

2. **X-signature**
The signature is required for transaction such as Inquiry and Payment. Signature using **hmac and sha256** with request method, relative path, times tamp, hash of entity body, username and password as parameters.
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
timestamp and apikey sent on request header.

3. **Content-length**
Content length is the length of the request body in bytes. The client must calculate the length of the  request body before sending it to the server. Therefore, the merchant may not send a request with partial content.

4. **Content-type**
The content type is filled with json/application.
5. **Host**
Server address that will be accessed by merchant. This address contains: domain/subdomain, domain/subdomain and port, IP address, IP address and port.

## 1.4 Request Body

Every request body always contains this two properties:

1. **command  (string)**
It is a command that must be execute by the server. This command must be write in lowercase letters. If the command is written incorrectly, the server will send a response with response code “Invalid Transaction” with an error message “Invalid Command”.
2. **product_code (string)**
It is a command that must be execute by the server. This command must be write in lowercase letters. If the command is written incorrectly, the server will send a response with response code “Invalid Transaction” with an error message “Invalid Command”.

2. **data (object)**
The data (object) contains the request message detail. Every command has different data properties.

## 1.5 URL
Merchants must provide at least an URL to accept requests for inquiry and payment virtual accounts. Merchants can unify or separate the URL. If the merchant unites the inquiry and payment URL, the merchant can distinguish between the two requests from the command parameter on the body request received.

# Chapter 2 - Topology

# Chapter 3 – Transaction Flow

# Chapter 4 – General Message Format

## 4.1 Inquiry

Inquiry is an electronic transaction to find out billing information for certain customer ID of the biller.

### Request

#### Header

**Accept-encoding**

Accept-encoding is encoding that can be accepted by AltoPay. Acceptable encoding, namely:

1. identity (without compression)
2. gzip (with GZIP compression)

**X-api-key**

The X-api-key is used to identity the client.

See **Request Headers** section on Chapter 1.

**X-timestamp**

The X-timestamp is one of signature component of the request to guarantee data integrity.

See **Request Headers** section on Chapter 1.

**X-signature**

The X-signature is signature of the request to guarantee data integrity. The parameters of the X-Signature are:

1. metod
2. path
3. body
4. timestamp
5. apikey
6. password

See **Request Headers** section on Chapter 1.

**Content-type**

Content-type is a format for data sent by AltoPay. The format of the data sent is application/json with charset utf-8.

**Accept**

The `Accept` request HTTP header advertises which content types, expressed as MIME types, the client is able to understand. Using content negotiation, the server then selects one of the proposals, uses it and informs the client of its choice with the Content-Type response header.

**Connection**

The `Connection` general header controls whether or not the network connection stays open after the current transaction finishes. If the value sent is `keep-alive`, the connection is persistent and not closed, allowing for subsequent requests to the same server to be done.

**User-agent**

The **User-Agent** request header contains a characteristic string that allows the network protocol peers to identify the application type, operating system, software vendor or software version of the requesting software user agent.  AltoPay will send specific **User-Agent**  **to merchant.**

**Content-Length**

The `Content-Length` entity header indicates the size of the entity-body, in bytes, sent to merchant.

**Host**

Host is the name of the server accessed by AltoPay. From this header, the merchant server can find out how AltoPay is connected to the merchant server.

#### Body

| **No** | **Parameter**|**Type**|**Description**|**Note**|
|---|---|---|---|---|
|1|command|String|Transaction command|Mandatory|
|2|product_code|Numeric, can begin with 0|Product code|Mandatory|
|3|data|Object|Container of the transaction data|Mandatory|
|4|data.customer_id|Numeric, can begin with 0|Customer ID|Mandatory|
|5|data.date_time|String|Transmission date and time in GMT (Format: yyyy-MM-dd’T’HH:mm:ss.SSS’Z’)|Mandatory|
|6|data.reference_number|String (32)|Reference number|Mandatory|

Note: The data length above is the maximum allowed. Shorter will be better.

# Chapter 5 – Product Message Format

Each product has a different message format depending on each biller. AltoPay adjusts the message format for each product according to its needs. Some products have far more data attributes than other products because they are related to biller policy.

Some products that have the same attributes and processes are combined into one product category, namely general billing. Some more products are made with special categories because they have very big differences.

Every message is always listed one of _product_code_ and _category_process. category_process_ is only used for prepaid cellular phone top up and postpaid cell phone bill payments.

## Prepaid Electricity

### Inquiry Request
```json
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

### Inquiry Response
```json
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

### Purchase Request
```json
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

### Purchase Response
```json
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

## Prepaid Cell Phone Credit

### Payment Request

```json
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
```json
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

```json
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

```
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

```json
{
	"command":"payment",
	"process_category":"6666",
	"data":{
		"date_time":"2019-10T23:56:59.987Z",
		"reference_number":"1234567889",
		"customer_id":"081234567",
		"customer_name":"Jono",
		"amount":"100000"
	}
}
```


### Payment Response

```json
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
```json
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
```json
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

```json
{
	"command":"payment",
	"product_code":"482382366",
	"data":{
		"date_time":"2019-10T23:56:59.987Z",
		"reference_number":"1234567889",
		"customer_id":"081234567",
		"customer_name":"Jono",
		"amount":"100000"
	}
}
```


### Payment Response

```json
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
