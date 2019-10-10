# CoinBene-Contract-WebSocket API

[中文](openapi-usdt-swap-websocket.md)

- [CoinBene-Contract-WebSocket API](#coinbene-contract-websocket-api)
  * [Overview](#overview)
  * [Command Format](#command-format)
    + [Subscription](#subscription)
    + [Unsubscription](#unsubscription)
  * [Login](#login)
  * [Connect limit](#connect-limit)
    + [Heartbeats](#heartbeats)
    + [The limits](#the-limits)
  * [Topics](#topics)
    + [Public Topics](#public-topics)
      - [OrderBook](#orderbook)
      - [Trade List](#trade-list)
      - [Ticker](#ticker)
      - [Kline](#kline)
    + [Private Topics](#private-topics)
      - [User Account](#user-account)
      - [User Position](#user-position)
      - [User Order](#user-order)
  * [Error Codes](#error-codes)

## Overview

WebSocket protocol is a new HTML5 protocol, which provides full-duplex communication between web browsers and web servers. Connection can be established after one handshake. Web server can then push business logic data to web browsers.Advantages:

Request header is small in size (around 2 bytes) during communication Since there is no need to create and delete TCP connection repeatedly, it saves resources . 

**We strongly recommend developers to use Websocket API to access market related information and trading depth.**

The connection will break automatically when a network problem occurs

url: *wss://ws-contract.coinbene.vip/usdt/openapi/ws*

## Command Format

Send format:

```json
{"op":"<value>","args": ["<value1>","<value2>"]}
```

The `op` support the following values

* `subscribe` 
* `unsubscribe` 
* `login` 

The `args` array,.

Successful response format:

```json
{"event": "<value>","topic":"<value>"}
{"topic":"<value>","data":"[<value1>,<value2>]"}
```

In the orderBook topic, the return formats for distinguishing between the first full amount and the subsequent incremental are:

```json
{"topic":"<value>","action":"<value>","data":["<value1>","<value2>"]}
```

Failure response format:

```json
{"event":"<value>","message":"<errorMessage>","code":"<errorCode>"}
```



### Subscription

Users may subscribe to one or more topics

The format:

```json
{"op":"subscribe", "args":["<topic>"]}
```

the value of `op` is `subscribe`

`args` the array content is topic names

Example:

```json
// send
{"op":"subscribe","args":["orderBook.BTC-SWAP.10","tradeList.BTC-SWAP"]}

// response
{"event":"subscribe","topic":"orderBook.BTC-SWAP.10"}
{"event":"subscribe","topic":"tradeList.BTC-SWAP"}
```

### Unsubscription

Users can cancel one or more topics

The format:

```json
{"op":"unsubscribe","args": ["<topic>"]}
```

Example:

```json
// send
{"op":"unsubscribe","args":["orderBook.BTC-SWAP.10","tradeList.BTC-SWAP"]}

// response
{"event":"unsubscribe","topic":"orderBook.BTC-SWAP.10"}
{"event":"unsubscribe","topic":"tradeList.BTC-SWAP"}
```



## Login

When  subscribe private topic, users need to log in first.

The format:

```json
{"op":"login","args":["<apiKey>","<expires>","<signature>"]}
```

| Parameters | Description                                                  | Generating rules                                             |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| apiKey     | API key                                                      | Obtained from the system                                     |
| expires    | The login expiration time expires after the set time expires. Please ensure that the effective time is greater than five minutes | ISO8601 ISO8601 time format。 Ex: 2019-06-18T01:51:51Z       |
| signature  | Signature                                                    | signature = hex(HMAC_SHA256(apiSecret, expires + method + requestPath))      method="GET"  requestPath="/login" |

> * apiKey applies in API management of CoinBene system
> * apiSecret will be generated when applying for apiKey, please keep it in a safe place
> * After the login authorization expires, the server will automatically cancel the private Topic subscribed by the client. The client should pay attention to re-login to extend the license period.
> *  When login please set method="GET"  requestPath="/login"

Signature demo code

```java
/*              demo（Java）：              */

// source：2019-07-04T02:19:08ZGET/login
// secret：9daf13ebd76c4f358fc885ca6ede5e27
// signature：3ded9d0113133c9f06cfa50ce99618e6d983a534f5a2219ebbe3ffb02b6fbe16

   public static void main(String[] args) {
        String secret = "9daf13ebd76c4f358fc885ca6ede5e27";
        String method = "GET"; // <1>
        String requestPath = "/login"; // <2>
        final DateTimeFormatter utcFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ssXXX");
        String timestamp = utcFormatter.format(ZonedDateTime.now(ZoneOffset.UTC).plusMinutes(10));
        String signStr = signForWebSocketApi(timestamp, method, requestPath, secret);
        System.out.println(signStr);
    }

    /**
     * generate signature
     *
     * @param method      
     * @param requestPath requestPath
     * @param secret     
     */
    private static String signForWebSocketApi(String timestamp, String method, String requestPath, String secret) {
        String shaResource = timestamp + method + requestPath;
        System.out.println(shaResource);
        String signStr = sha256_HMAC(shaResource, secret);
        return signStr;
    }
    
  /**
   * sha256_HMAC
   *
   * @param resource
   * @param secret   
   * @return signature
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
   * byte array to hex string
   *
   * @param bytes 
   * @return hex string
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
```

Example:

```json
// send
{"op":"login","args":["0d999f8ba827adfc494931d9e1539df9","2019-07-04T02:19:08Z","3ded9d0113133c9f06cfa50ce99618e6d983a534f5a2219ebbe3ffb02b6fbe16"]}

// response
{"event":"login","success":true}
```

## Connect limit

### Heartbeats

After connected to CoinBene's Websocket server, the server will send heartbeat periodically (currently at 5s interval) , The literal string `'ping'`. When client receives this heartbeat message, it should response with a literal string `'pong'`.

> After the server sent two consecutive heartbeat messages without receiving at least one  "pong" response from a client, then right before server sends the next "ping" heartbeat, the server will disconnect this client

### The limits

**Connection limit**：1 times/s

**Subscription limit**：240 times/hour

## Topics

Public topic list (don't require login)

| Description             | Format                     | Example              |
| ----------------------- | -------------------------- | -------------------- |
| depth information       | orderBook.{symbol}.{depth} | orderBook.BTC-SWAP.10 |
| trade information       | tradeList.{symbol}         | tradeList.BTC-SWAP    |
| ticker information      | ticker.{symbol}            | ticker.BTC-SWAP       |
| 1mins kline information | kline.{symbol}             | kline.BTC-SWAP        |

Prive topic list (require login)

| Description                 | Name          |
| --------------------------- | ------------- |
| user's account information  | user.account  |
| user's position information | user.position |
| user's order information    | user.order    |



### Public Topics

#### OrderBook

Format: `orderBook.{symbol}.{depth}`

*`symbol`*: contract name
*`depth`* : support 5、10、50、100

Example:

```json
// send 
{"op": "subscribe", "args": ["orderBook.BTC-SWAP.10"]}

//response
// full data
{
    "topic": "orderBook.BTC-SWAP", 
    "action": "insert",
    "data": [{
        "asks": [
            ["5621.7", "58", "2"], 
            ["5621.8", "125", "5"],
            ["5621.9", "100", "9"],
            ["5622", "84", "20"],
            ["5623.5", "90", "12"],
            ["5624.2", "1540", "15"],
            ["5625.1", "300",  "20"],
            ["5625.9", "350", "1"],
            ["5629.3", "200", "1"],
            ["5650", "1000", "8"]
        ],
        "bids": [
            ["5621.3", "287","8"],
            ["5621.2", "41","1"],
            ["5621.1", "2","1"],
            ["5621", "26","2"],
            ["5620.8", "194","2"],
            ["5620", "2", "1"],
            ["5618.8", "204","2"],
            ["5618.4", "30", "9"],
            ["5617.2", "2","1"],
            ["5609.9", "100", "12"]
        ],
        "version":1,
        "timestamp": "2019-07-04T02:21:08Z"
    }]
 }
 // incremental data
{
    "topic": "orderBook.BTC-SWAP", 
    "action": "update", 
    "data": [{
        "asks": [
            ["5621.7", "50", "2"],
            ["5621.8", "0", "0"],
            ["5621.9", "30", "5"]
        ],
        "bids": [
            ["5621.3", "10","1"],
            ["5621.2", "20","1"],
            ["5621.1", "80","5"],
            ["5621", "0","0"],
            ["5620.8", "10","1"]
        ],
        "version":2,
        "timestamp": "2019-07-04T02:21:09Z"
    }]
 }
```

> Full data `action = insert`   incremental data `action = update`
> ["5621.3", "10","1"] [String,String,String] 5621.3 is the price, 10 is the quantity of the price, 1 is the number of orders of the price.
> version   The data version is strictly incremented. The client can judge whether the data is continuous according to the version.

| Parameter | Parameter Type | Description       |
| --------- | -------------- | ----------------- |
| symbol    | string         | Contract name     |
| asks      | string         | Selling side      |
| bids      | string         | Buying side       |
| version   | number         | Data version      |
| timestamp | string         | System time stamp |

* manage a local order book correctly 

  > 1. subscribe `orderBook.{symbol}.{depth}` get full data first.
  > 2. receive incremental data resolve as following rule.
  >     * If price level not exists insert into order book
  >     * If price level exists and new quantity not equals 0, replace old.
  >     * If the quantity is 0, remove the price level

#### Trade List

topic format: `tradeList.{symbol}`

Example

```json
// send
{"op": "subscribe", "args": ["tradeList.BTC-SWAP"]}

// response
{
    "topic": "tradeList.BTC-SWAP",
    "data": [  
      [
        "8600.0000", 
        "s", 
        "100", 
        "2019-05-21T08:25:22.735Z"
      ]
    ]
 }
// array [price,side,quantity,timestamp]
```

返回参数

| Parameter | Parameter Type | Description                             |
| --------- | -------------- | --------------------------------------- |
| price     | string         | Filled price                            |
| quantity  | string         | Filled quantity                         |
| side      | string         | Filled side (b/s)    s: taker sell base |
| timestamp | string         | Filled time                             |



#### Ticker

To capture the latest traded price, best-bid price, best-ask price, and 24-hour trading volume of  contracts on the platform.

topic format: `ticker.{symbol}`

Example

```json
// send
{"op": "subscribe", "args": ["ticker.BTC-SWAP","ticker.ETHUSDT"]}

// response
{
    "topic": "ticker.BTC-SWAP",
    "data": [
        {
          "symbol": "BTC-SWAP",
          "lastPrice": "8548.0", 
          "markPrice": "8548.0", 
          "bestAskPrice": "8601.0", 
          "bestBidPrice": "8600.0",
          "bestAskVolume": "1222", 
          "bestBidVolume": "56505",
          "high24h": "8600.0000", 
          "low24h": "242.4500", 
          "volume24h": "4994", 
          "timestamp": "2019-05-06T06:45:56.716Z"
        }
    ]
 }
```



| Parameter    | Parameter Type | Description            |
| ------------ | -------------- | ---------------------- |
| symbol       | string         | Contract name          |
| bestAskPrice | string         | Best ask price         |
| bestAskSize  | string         | Best ask quantity      |
| bestBidPrice | string         | Best bid price         |
| bestBidSize  | string         | Best bid quantity      |
| lastPrice    | string         | Last traded price      |
| markPrice    | string         | Mark price             |
| high24h      | string         | 24h最高价              |
| low24h       | string         | 24 hour low price      |
| volume24h    | string         | 24 hour trading volume |

#### Kline

1mins kline information

topic format: `kline.{symbol}`

Example:

```json
// send
{"op": "subscribe", "args": ["kline.BTC-SWAP","kline.ETHUSDT"]}

// response
{
    "topic": "kline.BTC-SWAP",
    "data": [
        [
          "BTC-SWAP",
          "1557428280",
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
//array [symbol,time,open,close,high,low,volume,turnover,buyVolume,buyTurnover]
```



| Parameter   | Parameter Type | Description                            |
| ----------- | -------------- | -------------------------------------- |
| symbol      | string         | Contract name                          |
| time        | string         | Create Time                            |
| open        | string         | Open price                             |
| close       | string         | Close price                            |
| high        | string         | Highest price                          |
| low         | string         | Lowest price                           |
| volume      | string         | Trading quantity                       |
| turnover    | string         | The trading volume in a specific token |
| buyVolume   | string         | Taker buy base trading quantity        |
| buyTurnover | string         | Taker buy base trading volume          |



### Private Topics

When  subscribe following topics, users need to log in first.

#### User Account

Get the user's account information , require login.

topic name: `user.account`

Example:

```json
// send
{"op": "subscribe", "args": ["user.account"]}

// response
{
    "topic": "user.account",
    "data": [{
        "asset": "BTC",
        "availableBalance": "20.3859", 
        "frozenBalance": "0.7413",
        "balance": "21.1272", 
        "timestamp": "2019-05-22T03:11:22.0Z"
    }]
}
```



| Parameter        | Parameter Type | Description       |
| ---------------- | -------------- | ----------------- |
| asset            | string         | Asset name        |
| availableBalance | string         | Avaliable balance |
| frozenBalance    | string         | Frozen balance    |
| balance          | string         | Total balance     |



#### User Position

Get the information of holding positions of a contract. require login
topic name: `user.position`

Example:

```json
// send
{"op": "subscribe", "args": ["user.position"]}

// response
{
    "topic": "user.position",
    "data": [{
      "availableQuantity": "100", 
      "avgPrice": "7778.1", 
      "leverage": "20", 
      "liquidationPrice": "5441.0", 
      "markPrice": "8086.5", 
      "positionMargin": "0.0285",  
      "quantity": "507", 
      "realisedPnl": "0.0069", 
      "side": "long", 
      "symbol": "BTC-SWAP",
      "marginMode": "1",
      "createTime": "2019-05-22T03:11:22.0Z"
    }]
}
```



| Parameter         | Parameter Type | Description                    |
| ----------------- | -------------- | ------------------------------ |
| availableQuantity | string         | Avaliable close quantity       |
| avgPrice          | string         | Average open price             |
| leverage          | string         | Leverage                       |
| liquidationPrice  | string         | Liquidation price              |
| markPrice         | string         | Mark price                     |
| postionMargin     | string         | Margin                         |
| quantity          | string         | Total quantity                 |
| realisedPnl       | string         | Realized PnL                   |
| side              | string         | Side  long or short            |
| symbol            | string         | Contract name                  |
| marginMode        | string         | Margin mode  1:crossed 0:fixed |
| createTime        | string         | Create time                    |



#### User Order

Get user's order information , require login
Name: `user.order`

Example:

```json
// send
{"op": "subscribe", "args": ["user.order"]}

// response
{
    "topic": "user.order",
    "data": [{
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
      "status": "new",
      "lastFillQuantity": "0",
      "lastFillPrice": "0",
      "lastFillTime": ""
    }]
}
```

| Parameter        | Parameter  Type | Description                                       |
| ---------------- | --------------- | ------------------------------------------------- |
| orderId          | string          | order id                                          |
| direction        | string          | Direction                                         |
| leverage         | string          | Leverage                                          |
| orderType        | string          | order type   limit or market                      |
| quantity         | string          | Quantity                                          |
| orderPrice       | string          | order price                                       |
| orderValue       | string          | Order value                                       |
| fee              | string          | Fees                                              |
| filledQuantity   | string          | Filled quantity                                   |
| averagePrice     | string          | Average price                                     |
| orderTime        | string          | order Time                                        |
| lastFillPrice    | string          | Last transaction price (if none, push 0)）        |
| lastFillQuantity | string          | Last transaction amount (if none, push 0)         |
| lastFillTime     | string          | Last transaction time (if none, push "")          |
| status           | string          | order status  new,filled,canceled,partiallyFilled |

## Error Codes

| Code  | Description             |
| ----- | :---------------------- |
| 429   | API rate limit exceeded |
| 10501 | ping check timeout      |
| 10502 | unrecognized request    |
| 10503 | unsupported topic       |
| 10504 | user not logged in      |
| 10505 | invalid signature       |
| 10506 | invalid args            |
| 10507 | invalid expires         |
| 10508 | Invalid apiKey          |
| 10509 | unsupported depth       |
| 10500 | Internal system error   |
