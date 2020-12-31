# coinbene-usdt-rest-contract-openapi-rest-interface-description)
   * [coinbene-usdt-rest-contract-openapi-rest-interface-description)](#coinbene-usdt-rest-contract-openapi-rest-interface-description)
      * [Basic Information](#basic-information)
      * [restriction of visit](#restriction-of-visit)
      * [Interface Type](#interface-type)
      * [Signature method](#signature-method)
      * [Interface Specification](#interface-specification)
         * [Public interface - Get orderBook](#public-interface---get-orderbook)
         * [Public interface - Get all ticker information](#public-interface---get-all-ticker-information)
         * [Public interface - Get K line data](#public-interface---get-k-line-data)
         * [Public Interface - Get the latest filled orders](#public-interface---get-the-latest-filled-orders)
      * [Error Code Summary](#error-code-summary)

 

## Basic Information
- This article lists the baseurl of the REST interface: https://openapi-contract.coinbene.com
- Our server is deployed in the US West, and it is recommended that the api robot server be deployed in the US West to reduce network latency.
- It is recommended to add the own server export IP after the API is created to further enhance the API security check.
- The response of all interfaces is in JSON format.
- All time and timestamps are UNIX time in milliseconds.
- The HTTP 4XX error code is used to indicate the content, behavior, and format of the error.
- HTTP 418 indicates that access was continued after receiving 429, and was blocked.
- The HTTP 5XX error code is used to indicate problems on the Coinbene service side.
- Each interface may throw an exception. The format of the exception response is as follows:

```
{
  "code": xxxx,
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
- requestPath is the request interface path
- Body is the string of the request body. The GET request has no body information to omit; the POST request has a body information JSON string, such as {"instrument_id": "BTC-SWAP", "order_id": "7440"}
- The secret is generated when the user applies for the API
- **Do not disclose secret to others or transfer them to the server at any time**



## Interface Specification
### Public interface - Get orderBook
```
Get a deep list of contracts
Speed limit rule: 10 times per 1 seconds
HTTP GET /api/usdt/v2/market/orderBook?symbol=BTC-SWAP
```

Request parameters:

Name | Type | Required | Description |
---------|--------|---------|--------|
symbol | string | yes | contract name, such as BTC-SWAP |
size | string | No | Depth stall, values are 5, 10, 50, 100. Default value 10|

Return field description:

Name | Type | Description |
---------|---------|---------|
asks | array | seller depth, [gear price, quantity, the depth consists of several orders]|
bid | array | buyer depth, [gear price, quantity, the depth consists of several orders]|
symbol      | string |  contract name
timestamp      | string |  timestamp International time



```
Request:
Url: http://domain/api/usdt/v2/market/orderBook?symbol=BTC-SWAP&size=10
Method: GET
Headers: 
	Accept: application/json
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-21T11:10:28.464ZGET/api/usdt/v2/market/orderBook?symbol=BTC-SWAP&size=10


Response:
{
  "code": 200, 
  "data": {
    "symbol": "BTC-SWAP", 
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
Speed limit rule: 10 times per 1 seconds
HTTP GET /api/usdt/v2/market/tickers
```
Request parameters: none

Return field description:

Name | Type | Description
---------|---------|---------|
symbol | string | contract name, such as BTC-SWAP
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
timestamp      | string |  timestamp UTC International time

```
Request:
Url: http://domain/api/usdt/v2/market/tickers
Method: GET
Headers: 
	Accept: application/json
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-21T11:14:16.161ZGET/api/usdt/v2/market/tickers

Response:
{
  "code": 200, 
  "data": {
    "ETH-SWAP": {
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
      "timestamp":"2019-09-18T02:41:08.016Z"
    }, 
    "BTC-SWAP": {
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
      "timestamp":"2019-09-18T02:41:08.016Z"
    }
  }
}
```
### Public interface - Get K line data

```
Obtain contract K line data. K-line data can be obtained up to 2000.
Speed ​​limit rule: 10 times per 1 seconds
HTTP GET /api/usdt/v2/market/klines
```
Request parameters:

Name | Type | Required | Description |
---------|---------|---------|---------|
symbol | string | yes | contract name, such as BTC-SWAP |
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
timestamp | string | trade time
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
Url: http://domain/api/usdt/v2/market/klines?symbol=
&resolution=1&startTime=1557425760&endTime=1557425820
Method: GET
Headers: 
	Accept: application/json
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-21T11:16:20.521ZGET/api/usdt/v2/market/klines?symbol=BTC-SWAP&resolution=1&startTime=1557425760&endTime=1557425820

Response:
Format description:[time,open,high,low,close,volume,turnover,buyVolume,buyTurnover]
{
  "code": 200, 
  "data": [
    [
      "2019-09-18T02:41:08.016Z", 
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
      "2019-09-18T02:41:08.016Z", 
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
      "2019-09-18T02:41:08.016Z", 
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
Speed limit rule: 10 times per 1 seconds
HTTP GET /api/usdt/v2/market/trades
```
Request parameters:

Name | Type | Required | Description
---------|---------|---------|---------|
symbol | string | yes | contract name, such as BTC-SWAP
limit | string | no | return the number of records, default 10, maximum 100

Return field description:

the data is array ,Parse the data in the following sequence

Name | Type | Description
---------|---------|---------|
price | string | transaction price
side | string | direction of the transaction, s = main sale, b = main purchase
volume | string | Volume (number of sheets)
timestamp | string | closing time


```
Request:
Url: http://domain/api/usdt/v2/market/trades?symbol=BTC-SWAP&limit=1
Method: GET
Headers: 
	Accept: application/json
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-21T11:19:52.303ZGET/api/usdt/v2/market/trades?symbol=BTC-SWAP&limit=10

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
10010 | API authentication failed
10011 | Invalid sign
10012 |api verification failed
