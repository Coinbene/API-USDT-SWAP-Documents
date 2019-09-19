# coinbene-swap-rest-contract-openapi-rest-interface-description)
[中文版本](https://github.com/Coinbene/API-SWAP-Documents/blob/master/openapi-swap-rest.md)
* [Basic Information](#basic-information)
* [restriction of visit](#restriction-of-visit)
* [Interface Type](#interface-type)
* [Signature method](#signature-method)
* [Interface Specification](#interface-specification)
  * [Public interface - Get orderBook](#public-interface---get-orderbook)
  * [Public interface - Get all ticker information](#public-interface---get-all-ticker-information)
  * [Public interface - Get K line data](#public-interface---get-k-line-data)
  * [Public Interface - Get the latest filled orders](#public-interface---get-the-latest-filled-orders)
  * [Private Interface - Query Contract Account Information](#private-interface---query-contract-account-information)
  * [Private Interface - Get Position Information](#private-interface---get-position-information)
  * [Private Interface - Place an Order](#private-interface---place-an-order)
  * [Private Interface - Cancel an Order](#private-interface---cancel-an-order)
  * [Private Interface - Get  open orders](#private-interface---get--open-orders)
  * [Private Interface - Paging the current list of delegation orders by order ID](#private-interface---paging-the-current-list-of-delegation-orders-by-order-id)
  * [Private Interface - Get  order information](#private-interface---get--order-information)
  * [Private Interface - Query History Order](#private-interface---query-history-order)
  * [Private Interface - Query historical orders by order ID Pagination](#private-interface---query-historical-orders-by-order-id-pagination)
  * [Private Interface - Cancel multiple Orders](#private-interface---cancel-multiple-orders)
  * [Private Interface - Get the specified order transaction details](#private-interface---get-the-specified-order-transaction-details)
  * [Private Interface - Get a list of funding rates](#private-interface---get-a-list-of-funding-rates)
* [Error Code Summary](#error-code-summary)

## Basic Information
- This article lists the baseurl of the REST interface: http://openapi-contract.coinbene.com
- Need science online, Chinese domestic users recommend machine binding host：104.16.127.19 openapi-contract.coinbene.com
- Our server is deployed in the US West, and it is recommended that the api robot server be deployed in the US West to reduce network latency.
- It is recommended to add the own server export IP after the API is created to further enhance the API security check.
- The response of all interfaces is in JSON format.
- All time and timestamps are UNIX time in milliseconds.
- The HTTP 4XX error code is used to indicate the content, behavior, and format of the error.
- HTTP 418 indicates that access was continued after receiving 429, and was blocked.
- The HTTP 5XX error code is used to indicate problems on the Coinbene service side.
- HTTP 504 indicates that the API server has submitted a request to the business core but failed to get a response. It is important to note that the 504 code does not represent a request failure, but is unknown. It is very likely that it has already been executed, and it is possible that the execution will fail and further confirmation is needed.
- Each interface may throw an exception. The format of the exception response is as follows:

```
{
  "code": 10001,
  "msg": "Invalid Paramater."
}
```
- The specific error code and its explanation are summarized in the error code.
- The interface of the GET method, the parameter must be sent in the query string.
- The interface of the POST method, the parameter is sent in the request body (content type application/json).- No order is required for the order of the parameters.
## restriction of visit
- When the access interface exceeds the frequency limit, it will return a 429 status: the request is too frequent.
- Limit the rule. If a valid API key is passed, the user id is used to limit the rate; if not, the user's public network IP is used to limit the rate. There are separate instructions on each interface.
## Interface Type
- Mainly two types of interfaces, public interfaces and private interfaces.
- The public interface can be called without authentication.
- Private interface user orders and accounts. Each private request must be signed using a canonical form of authentication. The private interface needs to be verified with your API key.
## Signature method
All interface request headers must contain the following:
- ACCESS-KEY string type API key.
- ACCESS-SIGN uses Hex to generate strings.
- ACCESS-TIMESTAMP The timestamp of the request.
- All requests should contain application/json type content and be valid JSON.

ACCESS-SIGN value generation rules:
- According to timestamp + method + requestPath + body string (+ indicates string concatenation), and secret, use HMAC SHA256 method to encrypt, and finally convert the byte array of the encrypted string into a string to return.
- The timestamp value is the same as the ACCESS-TIMESTAMP request header, and must be the decimal time of the UTC time zone Unix timestamp or the time format of the ISO8601 standard, which is accurate to the millisecond.
- Method is the request method, all letters are capitalized: GET/POST
- requestPath is the request interface path, for example: /api/swap/v2/market/orderBook
- Body is the string of the request body. The GET request has no body information to omit; the POST request has a body information JSON string, such as {"symbol": "BTCUSDT", "order_id": "7440"}
- The secret is generated when the user applies for the API
- **Do not disclose secret to others or transfer them to the server at any time**

Sample interface request:
- Two cases of GET protocol interface:
```
1. Without parameters:
preHash String：2019-05-21T11:14:16.161ZGET/api/swap/v2/market/tickers
2. With parameters:
preHash String：2019-05-21T11:10:28.464ZGET/api/swap/v2/market/orderBook?symbol=ETHUSDT&size=10
```


```
Url: http://domain/api/swap/v2/market/tickers
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: E65791902180E9EF4510DB6A77F6EBAE
	ACCESS-SIGN: SswsZUURIjFke/rBHohJ1IFw0J7f67R08WpGZGGFaYI=
	ACCESS-TIMESTAMP: 2019-05-21T11:14:16.161Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-21T11:14:16.161ZGET/api/swap/v2/market/tickers
```


```
Url: http://domain/api/swap/v2/market/orderBook?symbol=ETHUSDT&size=10
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: E65791902180E9EF4510DB6A77F6EBAE
	ACCESS-SIGN: U3de14fFE9uOhpnBgNHxLtXMBCV7813K/VonpKDWqZE=
	ACCESS-TIMESTAMP: 2019-05-21T11:10:28.464Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-21T11:10:28.464ZGET/api/swap/v2/market/orderBook?symbol=ETHUSDT&size=10
```


- POST protocol interface situation:
```
preHash String：2019-03-08T10:59:25.789ZPOST/account/add{"symbol":"BTCUSDT","quantity":"70"}
```


```
Url: http://domain/api/swap/v2/order/place
Method: POST
Headers: 
	Accept: application/json
	ACCESS-KEY: E65791902180E9EF4510DB6A77F6EBAE
	ACCESS-SIGN: hhd/F4LFj/YAA5SC7x0gtSHxI0U9+VqD+orR1VMdofg=
	ACCESS-TIMESTAMP: 2019-05-22T03:33:53.562Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: {"symbol":"ETHUSDT","orderType":"limit","leverage":"20","orderPrice":"147.7","quantity":"7","direction":"openLong","marginMode":"fixed","clientId":"1558496033481"}
preHash: 2019-05-22T03:33:53.562ZPOST/api/swap/v2/order/place{"symbol":"ETHUSDT","orderType":"limit","leverage":"20","orderPrice":"147.7","quantity":"7","direction":"openLong","marginMode":"fixed","clientId":"1558496033481"}
```
- Signature algorithm verification:


```
Source string: 2019-05-25T03:20:30.362ZGET/api/swap/v2/account/info
secret：9daf13ebd76c4f358fc885ca6ede5e27
Generate a sign string: a02a6428bb44ad338d020c55acee9dd40bbcb3d96cbe3e48dd6185e51e232aa2

Sample code (Java version):
/**
    * Generate signature
    *
    * @param timeStamp timestamp
    * @param method Request method: POST or GET
    * @param requestUrl url
    * @param requestBody request content, no null passed
    * @param secret key
    */
  private String signForContractOpenApi(String timeStamp, String method, String requestUrl, String requestBody, String secret) {
    String shaResource = timeStamp + method + requestUrl + (requestBody == null ? "" : requestBody);
    System.out.println(shaResource);
    String signStr = sha256_HMAC(shaResource, secret);
    return signStr;
  }

  /**
    * sha256_HMAC encryption
    *
    * @param resource signature source string
    * @param secret key
    * @return Encrypted string
    */
  private static String sha256_HMAC(String resource, String secret) {
    String hash = "";
    try {
      Mac sha256_HMAC = Mac.getInstance("HmacSHA256");
      SecretKeySpec secret_key = new SecretKeySpec(secret.getBytes("UTF-8"), "HmacSHA256");
      sha256_HMAC.init(secret_key);
      byte[] bytes = sha256_HMAC.doFinal(resource.getBytes("UTF-8"));
      hash = byteArrayToHexString(bytes);
    } catch (Exception e) {
      System.out.println("Error HmacSHA256 ===========" + e.getMessage());
    }
    return hash;
  }

  /**
    * Convert the encrypted byte array to a string
    *
    * @param bytes byte array
    * @return string
    */
  private static String byteArrayToHexString(byte[] bytes) {
    StringBuffer buffer = new StringBuffer();
    String stmp;
    for (int index = 0; bytes != null && index < bytes.length; index++) {
      stmp = Integer.toHexString(bytes[index] & 0XFF);
      if (stmp.length() == 1) {
         buffer.append('0');
      }
      buffer.append(stmp);
    }
    return buffer.toString().toLowerCase();
  }

Sample code (Python version):

import hashlib
import hmac
import unittest

def sign(message, secret):
    """
    gen sign
    :param message: message wait sign
    :param secret:  secret key
    :return:
    """
    secret = secret.encode('utf-8')
    message = message.encode('utf-8')
    sign = hmac.new(secret, message, digestmod=hashlib.sha256).hexdigest()
    return sign

class TestUtil(unittest.TestCase):
    def test_sign(self):
        sn = sign("2019-05-25T03:20:30.362ZGET/api/swap/v2/account/info", "9daf13ebd76c4f358fc885ca6ede5e27")
        self.assertEqual(sn, "a02a6428bb44ad338d020c55acee9dd40bbcb3d96cbe3e48dd6185e51e232aa2")


```


## Interface Specification
### Public interface - Get orderBook
```
Get a deep list of contracts
Speed limit rule: 20 times per 2 seconds
HTTP GET /api/swap/v2/market/orderBook?symbol=BTCUSDT
```

Request parameters:

Name | Type | Required | Description |
---------|--------|---------|--------|
symbol | string | yes | contract name, such as BTCUSDT |
size | string | No | Depth stall, values are 5, 10, 50, 100. Default value 10|

Return field description:

Name | Type | Description |
---------|---------|---------|
asks | array | seller depth, [gear price, quantity, the depth consists of several orders]|
bid | array | buyer depth, [gear price, quantity, the depth consists of several orders]|
symbol      | string |  contract name
time      | string |  timestamp International time



```
Request:
Url: http://domain/api/swap/v2/market/orderBook?symbol=ETHUSDT&size=10
Method: GET
Headers: 
	Accept: application/json
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-21T11:10:28.464ZGET/api/swap/v2/market/orderBook?symbol=ETHUSDT&size=10


Response:
{
  "code": 200, 
  "data": {
    "symbol": "BTCUSDT", 
    "asks": [
      [
        "7863.0", 
        "8306", 
        "1"
      ], 
      [
        "7864.0", 
        "830", 
        "1"
      ], 
      [
        "7865.0", 
        "780", 
        "2"
      ], 
      [
        "7866.0", 
        "50", 
        "1"
      ], 
      [
        "7868.0", 
        "83", 
        "10"
      ]
    ], 
    "bids": [
      [
        "7863.0", 
        "8306", 
        "1"
      ], 
      [
        "7862.0", 
        "8306", 
        "1"
      ], 
      [
        "7859.0", 
        "8306", 
        "1"
      ], 
      [
        "7858.0", 
        "8306", 
        "2"
      ], 
      [
        "7857.0", 
        "8306", 
        "1"
      ]
    ],
    "time":"2019-09-18T02:41:08.016Z"
  }
}
```
### Public interface - Get all ticker information

```
Get the latest transaction price, buy one price, sell one price and 24 trading volume of all the contracts of the platform.
Speed limit rule: 20 times per 2 seconds
HTTP GET /api/swap/v2/market/tickers
```
Request parameters: none

Return field description:

Name | Type | Description
---------|---------|---------|
symbol | string | contract name, such as BTCUSDT
bestAskPrice | string | Sell one price
bestAskSize | string | Sell a quantity
bestBidPrice | string | Buy one price
bestBidSize | string | Buy one
lastPrice | string | Latest price
markPrice | string | tag price
high24h | string | 24h highest price
low24h | string | 24h lowest price
volume24h | string | 24h volume USDT
turnover | string | 
time      | string |  timestamp International time

```
Request:
Url: http://domain/api/swap/v2/market/tickers
Method: GET
Headers: 
	Accept: application/json
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-21T11:14:16.161ZGET/api/swap/v2/market/tickers

Response:
{
  "code": 200, 
  "data": {
    "ETHUSDT": {
      "lastPrice": "242.46", 
      "markPrice": "242.46", 
      "bestAskPrice": "243.20", 
      "bestBidPrice": "242.45", 
      "high24h": "8600.0000", 
      "low24h": "242.4500", 
      "volume24h": "4994", 
      "turnover": "9984", 
      "bestAskVolume": "2222", 
      "bestBidVolume": "5312",
      "time":"2019-09-18T02:41:08.016Z"
    }, 
    "BTCUSDT": {
      "lastPrice": "8548.0", 
      "markPrice": "8548.0", 
      "bestAskPrice": "8601.0", 
      "bestBidPrice": "8600.0", 
      "high24h": "8600.0000", 
      "low24h": "242.4500", 
      "volume24h": "4994", 
      "turnover": "4994", 
      "bestAskVolume": "1222", 
      "bestBidVolume": "56505",
      "time":"2019-09-18T02:41:08.016Z"
    }
  }
}
```
### Public interface - Get K line data

```
Obtain contract K line data. K-line data can be obtained up to 2000.
Speed ​​limit rule: 20 times per 2 seconds
HTTP GET /api/swap/v2/market/klines
```
Request parameters:

Name | Type | Required | Description |
---------|---------|---------|---------|
symbol | string | yes | contract name, such as BTCUSDT |
startTime | string | yes | start time, ISO8601 format timestamp to seconds |
endTime | string | yes | deadline, ISO8601 format timestamp to seconds |
resolution | string | yes | Kline granularity, range of values ​​reference description|


```
The value of resolution can only take ["1", "3", "5", "15", "30",
"60", "120", "240", "360", "720", "D", "W", "M"], otherwise the request will be rejected.
Corresponding to [1min, 3min, 5min, 15min, 30min,
Time period of 1hour, 2hour, 4hour, 6hour, 12hour, 1day, 1week, 1month]
```

Return field description:

the data is array ,Parse the data in the following sequence

Name | Type | Description
---------|---------|---------|
time | string | generation time
open | string | opening price
close | string | Closing price
high | string | highest price
low | string | Lowest price
volume | string | Volume (number of sheets)
turnover | string | transaction amount
buyVolume | string | Main Buy (number of sheets)
buyTurnover | string | main purchase amount


```
Request:
Url: http://domain/api/swap/v2/market/klines?symbol=BTCUSDT&resolution=1&startTime=1557425760&endTime=1557425820
Method: GET
Headers: 
	Accept: application/json
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-21T11:16:20.521ZGET/api/swap/v2/market/klines?symbol=BTCUSDT&resolution=1&startTime=1557425760&endTime=1557425820

Response:
Format description:[time,open,close,high,low,volume,turnover,buyVolume,buyTurnover]
{
  "code": 200, 
  "data": [
    [
      "1557428280", 
      "5794", 
      "5794", 
      "5794", 
      "5794", 
      "0", 
      "0", 
      "0", 
      "0"
    ], 
    [
      "1557426180", 
      "5794", 
      "5794", 
      "5794", 
      "5794", 
      "0", 
      "0", 
      "0", 
      "0"
    ], 
    [
      "1557427440", 
      "5794", 
      "5794", 
      "5794", 
      "5794", 
      "0", 
      "0", 
      "0", 
      "0"
    ]
  ]
}
```
### Public Interface - Get the latest filled orders

```
Get the latest filled orders information for the contract
Speed limit rule: 20 times per 2 seconds
HTTP GET /api/swap/v2/market/trades
```
Request parameters:

Name | Type | Required | Description
---------|---------|---------|---------|
symbol | string | yes | contract name, such as BTCUSDT
limit | string | no | return the number of records, default 10, maximum 100

Return field description:

the data is array ,Parse the data in the following sequence

Name | Type | Description
---------|---------|---------|
price | string | transaction price
side | string | direction of the transaction, s = main sale, b = main purchase
volume | string | Volume (number of sheets)
time | string | closing time


```
Request:
Url: http://domain/api/swap/v2/market/trades?symbol=BTCUSDT&limit=1
Method: GET
Headers: 
	Accept: application/json
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-21T11:19:52.303ZGET/api/swap/v2/market/trades?symbol=BTCUSDT&limit=10

Response:
{
  "code": 200, 
  "data": [
    [
      "8600.0000", 
      "s", 
      "100", 
      "2019-05-21T08:25:22.735Z"
    ], 
    [
      "8601.0000", 
      "s", 
      "10", 
      "2019-05-21T08:25:22.735Z"
    ]
  ]
}
```

### Private Interface - Query Contract Account Information

```
Get account information for user currency contracts
Speed limit: 10 times per 2 seconds
HTTP GET /api/swap/v2/account/info
```

Request parameter
no

Return result parameter

Name | Type | Description
---------|---------|---------|
availableBalance | string | Available balance
frozenBalance | string | Freeze assets
balance | string | account balance
marginRatio | string | margin rate
unrealisedPnl | string | Unrealized profit and loss


```
Request:
Url: http://domain/api/swap/v2/account/info
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: E65791902180E9EF4510DB6A77F6EBAE
	ACCESS-SIGN: LPwZXxu5vF2bAQK705QT9xk0n07Kpfco6rBP5iHs7dI=
	ACCESS-TIMESTAMP: 2019-05-21T11:23:34.403Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-21T11:23:34.403ZGET/api/swap/v2/account/info

Response:
{
  "code": 200, 
  "data": {
    "availableBalance": "20.3859", 
    "frozenBalance": "0.7413", 
    "marginBalance": "21.2127", 
    "marginRate": "0.0299", 
    "balance": "21.1272", 
    "unrealisedPnl": "0.0854"
  }
}
```
### Private Interface - Get Position Information


```
Get position information for all contracts
Speed ​​limit rule: 10 times per 2 seconds
HTTP GET /api/swap/v2/position/list
```
Request parameters:

Name | Type | Required | Description
---------|---------|---------|---------|
symbol | string | no | contract name, such as BTCUSDT

Return field description:

Name | Type | Description
---------|---------|---------|
availableQuantity | string | Number of positions that can be closed
averagePrice | string | Opening average price
createTime | string | position creation time
deleveragePercentile | string | Lighten up the queue, the higher the value, the higher the ranking
Leverage | string | leverage multiple
liquidationPrice | string | Strong price
markPrice | string | tag price
postionMargin | string | Position Margin
positionValue | string | BTC value of the position
quantity | string | Number of positions in the contract
realisedPnl | string | realized profit and loss
roe | string | rate of return
side | string | direction
symbol | string | contract name
unrealisedPnl | string | Unrealized profit and loss


```
Request:
Url: http://domain/api/swap/v2/position/list
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: E65791902180E9EF4510DB6A77F6EBAE
	ACCESS-SIGN: ZbrtcHKjRKSixSDabppcwsFcDvXAgQ2ak7mLXb3dXKY=
	ACCESS-TIMESTAMP: 2019-05-22T03:20:36.021Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-22T03:20:36.021ZGET/api/swap/v2/position/list

Response:
{
  "code": 200, 
  "data": [
    {
      "availableQuantity": "0", 
      "avgPrice": "7778.1", 
      "createTime": "2019-05-22T03:11:22.0Z", 
      "deleveragePercentile": "37", 
      "leverage": "20", 
      "liquidationPrice": "5441.0", 
      "markPrice": "8086.5", 
      "positionMargin": "0.0285", 
      "positionValue": "0.0627", 
      "quantity": "507", 
      "realisedPnl": "0.0069", 
      "roe": "0.0872", 
      "side": "long", 
      "symbol": "BTCUSDT", 
      "unrealisedPnl": "0.0024"
    }, 
    {
      "availableQuantity": "1000", 
      "avgPrice": "8000.0", 
      "createTime": "2019-05-22T03:00:04.0Z", 
      "deleveragePercentile": "0", 
      "leverage": "20", 
      "liquidationPrice": "8379.0", 
      "markPrice": "8086.5", 
      "positionMargin": "0.0063", 
      "positionValue": "0.1237", 
      "quantity": "1000", 
      "realisedPnl": "0.0004", 
      "roe": "-0.2139", 
      "side": "short", 
      "symbol": "BTCUSDT", 
      "unrealisedPnl": "-0.0013"
    }
  ]
}
```

### Private Interface - Place an Order

```
Place an order,Trading only supports limit orders.
Speed limit rule: 20 times per 2 seconds
HTTP POST/api/swap/v2/order/place
```
Request parameters:

Name | Type | Required | Description
---------|---------|---------|---------|
symbol | string | yes | contract name, such as BTCUSDT
direction | string | yes | direction
leverage | string | yes | leverage multiple
orderType | string | is the | order type, default to limit, and other enumeration values: postOnly (make only), FOK (Fill or Kill), IOC (Immediate Or Cancel)
orderPrice | string | yes | order price
quantity | string | yes | number of contracts bought or sold (number of sheets)
marginMode | string | no | warehouse mode, default value crossed, fixed warehouse-by-warehouse crossed full warehouse
clientId | string | no | user request id, transparently returned to the user


```
1.direction value description:
openLong = open long ; openShort=open short ;closeLong= close long; closeShort=close short
3.leverage value range: 2, 3, 5, 10, 20
4.orderPrice accuracy description: BTCUSDT price accuracy is 0.5; ETHUSDT price accuracy is 0.05
```

Return field description:

Name | Type | Description
---------|---------|---------|
orderId | string | generated order id
clientId | string | clientId requested by client


```
Request:
Url: http://domain/api/swap/v2/order/place
Method: POST
Headers: 
	Accept: application/json
	ACCESS-KEY: E65791902180E9EF4510DB6A77F6EBAE
	ACCESS-SIGN: hhd/F4LFj/YAA5SC7x0gtSHxI0U9+VqD+orR1VMdofg=
	ACCESS-TIMESTAMP: 2019-05-22T03:33:53.562Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: {"symbol":"ETHUSDT","orderType":"limit","leverage":"20","orderPrice":"147.7","quantity":"7","direction":"openLong","marginMode":"fixed","clientId":"1558496033481"}
preHash: 2019-05-22T03:33:53.562ZPOST/api/swap/v2/order/place{"symbol":"ETHUSDT","orderType":"limit","leverage":"20","orderPrice":"147.7","quantity":"7","direction":"openLong","marginMode":"fixed","clientId":"1558496033481"}

Response:
{
  "code": 200, 
  "data": {
    "orderId": "580719990266232832", 
    "clientId": "1558496033481"
  }
}
```

### Private Interface - Cancel an Order

```
Cancelling an unfilled order
Speed limit rule: 20 times per 2 seconds
HTTP POST/api/swap/v2/order/cancel
```
Request parameters:

Name | Type | Required | Description
---------|---------|---------|---------|
orderId | string | yes | order ID

Return field description:

Name | Type | Description
---------|---------|---------|
data | string | Undo Order Id

```
Request:
Url: http://domain/api/swap/v2/order/cancel
Method: POST
Headers: 
	Accept: application/json
	ACCESS-KEY: E65791902180E9EF4510DB6A77F6EBAE
	ACCESS-SIGN: zQafkYJgk514N6FQAeyYcdLRON1vvpCw5fy2BiZaIAM=
	ACCESS-TIMESTAMP: 2019-05-22T03:36:33.251Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: {"orderId":"580719990266232832"}
preHash: 2019-05-22T03:36:33.251ZPOST/api/swap/v2/order/cancel{"orderId":"580719990266232832"}

Response:
{
  "code": 200, 
  "data": "580719990266232832"
}
```
### Private Interface - Get  open orders 


```
Order list query by user request,
Speed limit rule: 5 times per 2 seconds
HTTP GET/api/swap/v2/order/openOrders
```
Request parameters:

Name | Type | Required | Description
---------|---------|---------|---------|
symbol | string | no | contract name, such as BTCUSDT
pageNum | string | no | page number, default page 1
pageSize | string | no | single page number, default 10


```
Description
1. Maximum support for 50 orders at the same time
2. Assume that a total of 50 orders have been placed, and these orders have not been filled. If the parameter pageNum=5, pageSize=10, the order within [41, 50] is returned.
```


Return field description:

Name | Type | Description
---------|---------|---------|
orderId | string | Order Id
direction | string | direction
leverage | string | leverage multiple
orderType | string | order type, limit price = limit market price = market
quantity | string | Volume (number of sheets)
orderPrice | string | order price
orderValue | string | order value
fee | string | handling fee
fillQuantity | string | Volume (number of sheets)
averagePrice | string | Average transaction price
orderTime | string | order creation time
status | string | Order Status(new,filled,canceled,partiallyCanceled）


```
Request:
Url: http://domain/api/swap/v2/order/openOrders?symbol=ETHUSDT&pageNum=1&pageSize=3
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: E65791902180E9EF4510DB6A77F6EBAE
	ACCESS-SIGN: U4pKbDuOXqOqbQiqbHLAptEjTSpG7UYUrsbKZclJ5dU=
	ACCESS-TIMESTAMP: 2019-05-22T03:40:14.396Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-22T03:40:14.396ZGET/api/swap/v2/order/openOrders?symbol=ETHUSDT&pageNum=1&pageSize=3

Response:
{
  "code": 200, 
  "data": [
    {
      "orderId": "580721369818955776", 
      "direction": "openLong", 
      "leverage": "20", 
      "symbol": "ETHUSDT", 
      "orderType": "limit", 
      "quantity": "7", 
      "orderPrice": "146.30", 
      "orderValue": "0.0010", 
      "fee": "0.0000", 
      "filledQuantity": "0", 
      "averagePrice": "0.00", 
      "orderTime": "2019-05-22T03:39:24.0Z", 
      "status": "new"
    }, 
    {
      "orderId": "580721368082513920", 
      "direction": "openLong", 
      "leverage": "20", 
      "symbol": "ETHUSDT", 
      "orderType": "limit", 
      "quantity": "6", 
      "orderPrice": "145.90", 
      "orderValue": "0.0009", 
      "fee": "0.0000", 
      "filledQuantity": "0", 
      "averagePrice": "0.00", 
      "orderTime": "2019-05-22T03:39:23.0Z", 
      "status": "new"
    }
  ]
}
```

### Private Interface - Paging the current list of delegation orders by order ID 

```
Order list query by user request,
Speed limit rule: 5 times per 2 seconds
HTTP GET/api/swap/v2/order/openOrdersByPage
```
Request parameters:

Name | Type | Required | Description
---------|---------|---------|---------|
symbol | string | no | contract name, such as BTCUSDT
latestOrderId | string | no | Order id, paging. The default is empty, returning the latest 20 data records


Return field description:

Name | Type | Description
---------|---------|---------|
orderId | string | Order Id
direction | string | direction
leverage | string | leverage multiple
orderType | string | order type, limit price = limit market price = market
quantity | string | Volume (number of sheets)
orderPrice | string | order price
orderValue | string | order value
fee | string | handling fee
fillQuantity | string | Volume (number of sheets)
averagePrice | string | Average transaction price
orderTime | string | order creation time
status | string | Order Status(new,filled,canceled,partiallyCanceled）


```
Request:
Url: http://domain/api/swap/v2/order/openOrdersByPage
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: E65791902180E9EF4510DB6A77F6EBAE
	ACCESS-SIGN: U4pKbDuOXqOqbQiqbHLAptEjTSpG7UYUrsbKZclJ5dU=
	ACCESS-TIMESTAMP: 2019-05-22T03:40:14.396Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-22T03:40:14.396ZGET/api/swap/v2/order/openOrdersByPage

Response:
{
  "code": 200, 
  "data": [
    {
      "orderId": "580721369818955776", 
      "direction": "openLong", 
      "leverage": "20", 
      "symbol": "ETHUSDT", 
      "orderType": "limit", 
      "quantity": "7", 
      "orderPrice": "146.30", 
      "orderValue": "0.0010", 
      "fee": "0.0000", 
      "filledQuantity": "0", 
      "averagePrice": "0.00", 
      "orderTime": "2019-05-22T03:39:24.0Z", 
      "status": "new"
    }, 
    {
      "orderId": "580721368082513920", 
      "direction": "openLong", 
      "leverage": "20", 
      "symbol": "ETHUSDT", 
      "orderType": "limit", 
      "quantity": "6", 
      "orderPrice": "145.90", 
      "orderValue": "0.0009", 
      "fee": "0.0000", 
      "filledQuantity": "0", 
      "averagePrice": "0.00", 
      "orderTime": "2019-05-22T03:39:23.0Z", 
      "status": "new"
    }
  ]
}
```

### Private Interface - Get  order information

```
Get  order information by order_id
Speed limit rule: 10 times per 2 seconds
HTTP GET/api/swap/v2/order/info
```
Request parameters:

Name | Type | Required | Description
---------|---------|---------|---------|
orderId | string | yes | order id

Return field description:

Name | Type | Description
---------|---------|---------|
orderId | string | Order Id
direction | string | direction
leverage | string | leverage multiple
orderType | string | order type, limit price = limit market price = market
quantity | string | Volume (number of sheets)
orderPrice | string | order price
orderValue | string | order value
fee | string | handling fee
fillQuantity | string | Volume (number of sheets)
averagePrice | string | Average transaction price
orderTime | string | order creation time
status | string | Order Status(new,filled,canceled,partiallyFilled）



```
Request:
Url: http://domain/api/swap/v2/order/info?orderId=580721369818955776
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: E65791902180E9EF4510DB6A77F6EBAE
	ACCESS-SIGN: aH/1blAlzwKnyaHHPKt6evg9naD7IBzeT0oBz4dKk7A=
	ACCESS-TIMESTAMP: 2019-05-22T03:47:41.653Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-22T03:47:41.653ZGET/api/swap/v2/order/info?orderId=580721369818955776

Response:
{
  "code": 200, 
  "data": {
    "orderId": "580721369818955776", 
    "direction": "openLong", 
    "leverage": "20", 
    "symbol": "ETHUSDT", 
    "orderType": "limit", 
    "quantity": "7", 
    "orderPrice": "146.30", 
    "orderValue": "0.0010", 
    "fee": "0.0000", 
    "filledQuantity": "0", 
    "averagePrice": "0.00", 
    "orderTime": "2019-05-22T03:39:24.0Z ", 
    "status": "new"
  }
}
```

### Private Interface - Query History Order

```
Query by user input
Speed ​​limit rule: 5 times per 2 seconds
HTTP GET/api/swap/v2/order/closedOrders
```

Request parameters:

Name | Type | Required | Description
---------|---------|---------|---------|
beginTime | string | no | start time, millisecond timestamp
endTime | string | no | end time, millisecond timestamp
symbol | string | yes | contract name, such as BTCUSDT
pageNum | string | no | page number, default value 1
pageSize | string | no | single page number, default 10
direction | string | no | openLong=open more openShort=open
orderType | string | no | order type


Return field description:

Name | Type | Description
---------|---------|---------|
orderId | string | Order Id
direction | string | direction
leverage | string | leverage multiple
orderType | string | order type, limit price = limit market price = market
quantity | string | Volume (number of sheets)
orderPrice | string | order price
orderValue | string | order value
fee | string | handling fee
fillQuantity | string | Volume (number of sheets)
averagePrice | string | Average transaction price
orderTime | string | order creation time
status | string | Order Status(filled,canceled,partiallyCanceled）


```
Request:
Url: http://domain/api/swap/v2/order/closedOrders?symbol=ETHUSDT&pageNum=1&pageSize=10
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: E65791902180E9EF4510DB6A77F6EBAE
	ACCESS-SIGN: cA+ci0WdlF3fJMJlI+M4YtR2mYZSuBa+jgh+nPTe0AQ=
	ACCESS-TIMESTAMP: 2019-05-22T04:03:41.607Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-22T04:03:41.607ZGET/api/swap/v2/order/closedOrders?symbol=ETHUSDT&pageNum=1&pageSize=10

Response:
{
  "code": 200, 
  "data": [
    {
      "orderId": "580719990266232832", 
      "direction": "openLong", 
      "leverage": "20", 
      "symbol": "ETHUSDT", 
      "orderType": "limit", 
      "quantity": "7", 
      "orderPrice": "147.70", 
      "orderValue": "0.0010", 
      "fee": "0.0000", 
      "filledQuantity": "0", 
      "averagePrice": "0.00", 
      "orderTime": "2019-05-22T03:33:55.0Z", 
      "status": "canceled"
    }, 
    {
      "orderId": "580719596848906240", 
      "direction": "openLong", 
      "leverage": "20", 
      "symbol": "ETHUSDT", 
      "orderType": "limit", 
      "quantity": "2", 
      "orderPrice": "146.05", 
      "orderValue": "0.0003", 
      "fee": "0.0000", 
      "filledQuantity": "0", 
      "averagePrice": "0.00", 
      "orderTime": "2019-05-22T03:32:21.0Z", 
      "status": "canceled"
    }
  ]
}
```

### Private Interface - Query historical orders by order ID Pagination

```
Query by user input
Speed limit rule: 5 times per 2 seconds
HTTP GET/api/swap/v2/order/closedOrdersByPage
```

Request parameters:

Name | Type | Required | Description
---------|---------|---------|---------|
beginTime | string | no | start time, millisecond timestamp
endTime | string | no | end time, millisecond timestamp
symbol | string | no | contract name, such as BTCUSDT
status | string | Order Status(filled,canceled,partiallyFilled）
latestOrderId | string | no | Order id, paging. The default is empty, returning the latest 20 data records


Return field description:

Name | Type | Description
---------|---------|---------|
orderId | string | Order Id
direction | string | direction
leverage | string | leverage multiple
orderType | string | order type, limit price = limit market price = market
quantity | string | Volume (number of sheets)
orderPrice | string | order price
orderValue | string | order value
fee | string | handling fee
fillQuantity | string | Volume (number of sheets)
averagePrice | string | Average transaction price
orderTime | string | order creation time
status | string | Order Status(new,filled,canceled,partiallyCanceled）


```
Request:
Url: http://domain/api/swap/v2/order/closedOrdersByPage
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: E65791902180E9EF4510DB6A77F6EBAE
	ACCESS-SIGN: cA+ci0WdlF3fJMJlI+M4YtR2mYZSuBa+jgh+nPTe0AQ=
	ACCESS-TIMESTAMP: 2019-05-22T04:03:41.607Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-22T04:03:41.607ZGET/api/swap/v2/order/closedOrdersByPage

Response:
{
  "code": 200, 
  "data": [
    {
      "orderId": "580719990266232832", 
      "direction": "openLong", 
      "leverage": "20", 
      "symbol": "ETHUSDT", 
      "orderType": "limit", 
      "quantity": "7", 
      "orderPrice": "147.70", 
      "orderValue": "0.0010", 
      "fee": "0.0000", 
      "filledQuantity": "0", 
      "averagePrice": "0.00", 
      "orderTime": "2019-05-22T03:33:55.0Z", 
      "status": "canceled"
    }, 
    {
      "orderId": "580719596848906240", 
      "direction": "openLong", 
      "leverage": "20", 
      "symbol": "ETHUSDT", 
      "orderType": "limit", 
      "quantity": "2", 
      "orderPrice": "146.05", 
      "orderValue": "0.0003", 
      "fee": "0.0000", 
      "filledQuantity": "0", 
      "averagePrice": "0.00", 
      "orderTime": "2019-05-22T03:32:21.0Z", 
      "status": "canceled"
    }
  ]
}
```

### Private Interface - Cancel multiple Orders

```
Cancel multiple Orders
Speed limit rule: 5 times per 2 seconds
HTTP POST/api/swap/v2/order/batchCancel
```
Request parameters:

Name | Type | Required | Description
---------|---------|---------|---------|
orderIds | list<string> | Yes | Order Id List，Up to 10 order ids at a time

Return field description:

Name | Type | Description
---------|---------|---------|
orderId | string | Undo Order Id


```
Request:
Url: http://domain/api/swap/v2/order/batchCancel
Method: POST
Headers: 
	Accept: application/json
	ACCESS-KEY: E65791902180E9EF4510DB6A77F6EBAE
	ACCESS-SIGN: h+b1jbtIlxBIqW3oZjM2xigZa+xGFdHqe7lkPTGqNck=
	ACCESS-TIMESTAMP: 2019-05-22T04:10:50.176Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: {"orderIds":["578639816552972288","578639902896914432"]}
preHash: 2019-05-22T04:10:50.176ZPOST/api/swap/v2/order/batchCancel{"orderIds":["578639816552972288","578639902896914432"]}

Response:		
{
  "code": 200, 
  "data": [
    {
      "orderId": "578639816552972288", 
      "code": "200"
    }, 
    {
      "orderId": "578639902896914432", 
      "code": "10325", 
      "message": "Order does not exist "
    }
  ]
}
```


### Private Interface - Get the specified order transaction details

```
Cancel the order by user input
Speed limit rule: 10 times per 2 seconds
HTTP GET/api/swap/v2/order/fills
```
Request parameters:

Name | Type | Required | Description
---------|---------|---------|---------|
Symbol | string | yes | contract name, such as BTCUSDT
orderId | string | yes | orderId
lastTradeId | string | no | deal Id, page break, fixed 20 per page, default 0 returns the latest 20

Return field description:

Name | Type | Description
---------|---------|---------|
orderId | string | Order Id
Direction | string | order direction
Leverage | string | leverage multiple
Symbol | string | contract name, such as BTCUSDT
orderType | string | order type, limit price = limit market price = market
Quantity | string | Order Quantity (Zhang)
orderPrice | string | order price
orderValue | string | order value
Fee | string | handling fee
FillQuantity | string | Order Quantity (Zhang)
averagePrice | string | Average order price
orderTime | string | order time
Status | string | Order status (new: pending order, filled: completed transaction, canceled: complete withdrawal, partiallyCanceled: partial withdrawal)

```
Request:
Url: http://172.20.20.156:9320/api/swap/v2/order/fills?symbol=ETHUSDT&lastTradeId=0&orderId=586149733106667520
Method: GET
Headers:
Accept: application/json
ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
ACCESS-SIGN: d279f3c59f38cf2c4b25862aa43bf81aa4b5783fab193b2a1683d5be81c3823a
ACCESS-TIMESTAMP: 2019-06-09T04:14:18.213Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=zh_CN
Body:
preHash: 2019-06-09T04:14:18.213ZGET/api/swap/v2/order/fills?symbol=ETHUSDT&lastTradeId=0&orderId=586149733106667520

Response:
{
  "code": 200,
  "data": [
    {
      "orderId": "580719990266232832",
      "direction": "openLong",
      "leverage": "20",
      "symbol": "ETHUSDT",
      "orderType": "limit",
      "quantity": "7",
      "orderPrice": "147.70",
      "orderValue": "0.0010",
      "fee": "0.0000",
      "filledQuantity": "0",
      "averagePrice": "0.00",
      "orderTime": "2019-05-22T03:33:55.0Z",
      "status": "canceled"
    },
    {
      "orderId": "580719596848906240",
      "direction": "openLong",
      "leverage": "20",
      "symbol": "ETHUSDT",
      "orderType": "limit",
      "quantity": "2",
      "orderPrice": "146.05",
      "orderValue": "0.0003",
      "fee": "0.0000",
      "filledQuantity": "0",
      "averagePrice": "0.00",
      "orderTime": "2019-05-22T03:32:21.0Z",
      "status": "canceled"
    }
  ]
}
```

### Private Interface - Get a list of funding rates

```
Cancel the order by user input
Speed limit rule: 10 times per 2 seconds
HTTP GET/api/swap/v2/position/feeRate
```
Request parameters:

Name | Type | Required | Description
---------|---------|---------|---------|
pageNum | string | no | default value of 1, indicating the first page begins
pageSize | string | no | default value 10, number of records returned per page

Return field description:

Name | Type | Description
---------|---------|---------|
Symbol | string | contract name, such as BTCUSDT
Side | string | direction
markPrice | string | tag price
positionValue | string | position value
Fee | string | handling fee
feeRate | string | rate
Time | string | charge time
Leverage | string | leverage multiple

```
Request:
Url: http://172.20.20.156:9320/api/swap/v2/position/feeRate?pageNum=1&pageSize=2
Method: GET
Headers:
Accept: application/json
ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
ACCESS-SIGN: 8a0fc6ee1d39c95100c00ea42ba2033d8a33a644d53150e1851e1fff04e1f536
ACCESS-TIMESTAMP: 2019-06-09T04:22:06.355Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=zh_CN
Body:
preHash: 2019-06-09T04:22:06.355ZGET/api/swap/v2/position/feeRate?pageNum=1&pageSize=2

Response:
{
  "code": 200,
  "data": [
    {
      "symbol": "BTCUSDT",
      "side": "short",
      "markPrice": "8086.5000",
      "positionValue": "0.1237",
      "fee": "0.0004637358",
      "feeRate": "0.003750",
      "time": "2019-05-31T09:00:00.0Z",
      "leverage": "20"
    },
    {
      "symbol": "BTCUSDT",
      "side": "long",
      "markPrice": "8086.5000",
      "positionValue": "0.1237",
      "fee": "-0.0004637359",
      "feeRate": "0.003750",
      "time": "2019-05-31T09:00:00.0Z",
      "leverage": "20"
    }
  ]
}
```


## Error Code Summary

Error code | message
---|:---
429 | Requests are too frequent
10001 | "ACCESS_KEY" cannot be empty
10002 | "ACCESS_SIGN" cannot be empty
10003 | "ACCESS_TIMESTAMP" cannot be empty
10005 | Invalid ACCESS_TIMESTAMP
10006 | Invalid ACCESS_KEY
10007 | Invalid Content_Type, please use the "application/json" format
10008 | Request timestamp expired
10009 | System error
100011 | Invalid sign
100012 |api verification failed
11000 |{0}Required parameter is empty
11001 | {0} parameter value is invalid
11002 |{0} parameter value exceeds maximum limit
10200 | Insufficient balance
10201 | Account has gone wrong
10202 | Parameter error
10204 | The account does not exist
10300 | Order failed
10301 | Parameter error
10302 | The minimum order quantity is {0}
10303 | The maximum order quantity is {0}
10304 | The minimum price change is {0}
10305 | The maximum number of in-progress orders under a contract is {0}
10306 | Leverage used{0}
10307 | Insufficient account balance
10308 | Entrusted price limit at {0} will result in a position being forced, please adjust the commission price
10309 | Cancellation failed
10310 | Failed to cancel the transaction
10311 | The number of closed positions cannot be greater than the remaining open positions
10312 | Closing the commission price can not break the strong price
10313 | Opening the current market price will result in a risk of holding a position is too high, it is easy to be flat, please use the limit order
10314 | Query failed
10315 | The entrusted price cannot deviate by more than 50% from the current mark price
10316 | Contract does not exist
10317 | User does not exist
10318 | Lever multiple exceeds allowable range
10318 | Exceeded the maximum number of withdrawals
10319 | User is frozen
10320 | Order type is not suitable
10321 | Order direction error
10321| Incorrect order type
10322| Order direction error
10323| Passive parameters are empty
10324| Position does not exist
10325| Order does not exist
10326| Limit price cannot be empty
10329| Entrusted to deal in {0} will result in a position margin loss to the minimum maintenance position margin level Please adjust the commission price or reduce the leverage
10332 | Margin model error
10333 | Frequently order less than 10 sheets, disable order function for 10 minutes
