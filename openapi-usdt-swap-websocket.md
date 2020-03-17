# CoinBene-Contract-WebSocket接口说明

[English](openapi-usdt-swap-websocket-en.md)

- [CoinBene-Contract-WebSocket接口说明](#CoinBene-Contract-WebSocket接口说明)
  * [概述](#概述)
  * [指令格式](#指令格式)
    * [订阅](#订阅)
    * [取消订阅](#取消订阅)
  * [登录](#登录)
  * [连接限制](#连接限制)
    + [心跳消息](#心跳消息)
    + [限频规则](#限频规则)
  * [Topic说明](#Topic说明)
    + [公共Topic](#公共Topic)
      - [深度列表](#深度列表)
      - [最新成交](#最新成交)
      - [Ticker信息](#Ticker信息)
      - [K线数据](#K线数据)
    + [私有Topic](#私有Topic)
      - [用户账户](#用户账户)
      - [用户持仓](#用户持仓)
      - [用户交易](#用户交易)
  * [错误代码汇总](#错误代码汇总)

## 概述

WebSocket是HTML5一种新的协议(Protocol)。它实现了客户端与服务器全双工通信， 使得数据可以快速地双向传播。通过一次简单的握手就可以建立客户端和服务器连接， 服务器根据业务规则可以主动推送信息给客户端。其优点如下：

* 客户端和服务器进行数据传输时，请求头信息比较小，大概2个字节。
* 客户端和服务器皆可以主动地发送数据给对方。
* 不需要多次创建TCP请求和销毁，节约宽带和服务器的资源。

*强烈建议开发者使用WebSocket API获取市场行情和买卖深度等信息。*

.地址
`wss://ws.coinbene.vip/stream/ws`

> * 访问地址需要具备科学上网环境
> * 连接上ws后需主动处理服务端的`ping`检测消息,用于连接保活
> * CoinBene服务器部署在美西，为最大限度地减少API访问延迟，建议您使用与美西通讯通畅的服务器

## 指令格式

请求

```json
{"op":"<value>","args": ["<value1>","<value2>"]}
```

其中`op`可取值

* `subscribe` 订阅
* `unsubscribe` 取消订阅
* `login` 登录

`args`为执行`op`操作的参数,视`op`不同而不同.

成功响应

```json
{"event": "<value>","topic":"<value>"}
{"topic":"<value>","data":"[<value1>,<value2>]"}
```

其中为了区分是首次全量和后续增量返回格式将会是

```json
{"topic":"<value>","action":"<value>","data":["<value1>","<value2>"]}
```

失败响应

```json
{"event":"<value>","message":"<errorMessage>","code":"<errorCode>"}
```

### 订阅

用户可以同时订阅一个或者多个Topic

指令格式

```json
{"op":"subscribe", "args":["<topic>"]}
```

订阅时`op`取值`subscribe`

`args`为topic数组

示例

```json
// send
{"op":"subscribe","args":["usdt/orderBook.BTC-SWAP.10","usdt/tradeList.BTC-SWAP"]}

// response
{"event":"subscribe","topic":"usdt/orderBook.BTC-SWAP.10"}
{"event":"subscribe","topic":"usdt/tradeList.BTC-SWAP"}
```

### 取消订阅

可以同时取消一个或者多个Topic

指令格式

```json
{"op":"unsubscribe","args": ["<topic>"]}
```

示例

```json
// send
{"op":"unsubscribe","args":["usdt/orderBook.BTC-SWAP.10","usdt/tradeList.BTC-SWAP"]}

// response
{"event":"unsubscribe","topic":"usdt/orderBook.BTC-SWAP.10"}
{"event":"unsubscribe","topic":"usdt/tradeList.BTC-SWAP"}
```



## 登录

如果您希望订阅私有Topic，则必须先进行登录。

指令格式

```json
{"op":"login","args":["<apiKey>","<expires>","<signature>"]}
```

| 参数      | 描述                                                         | 生成规则                                                     |
| --------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| apiKey    | API key                                                      | 系统中申请获得                                               |
| expires   | 登录失效时间,超过设定的时间后本次登录失效. 请保证有效时长大于五分钟 | ISO8601标准时间格式，精确到秒。 例如 2019-06-18T01:51:51Z    |
| signature | 签名                                                         | hex(HMAC_SHA256(apiSecret, expires + method + requestPath))  此处method="GET"  requestPath="/login" |

> * apiKey在CoinBene系统的API管理中申请
> * 申请apiKey的同时会生成apiSecret,请妥善保管.任何时候都不要透露给其他人或传输到服务器端
> * 登录授权失效后,服务器会自动取消客户端订阅的私有Topic,客户端应注意重新登录来延长授权期限.
> *  method和requestPath为固定值 method="GET"  requestPath="/login"

签名算法验证(参考)

```java
/*              样例代码（Java版本）：              */

// 源串：2019-07-04T02:19:08ZGET/login
// secret：9daf13ebd76c4f358fc885ca6ede5e27
// 生成sign串：3ded9d0113133c9f06cfa50ce99618e6d983a534f5a2219ebbe3ffb02b6fbe16

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
     * 生成签名
     *
     * @param method      请求方法：POST或者GET
     * @param requestPath requestPath
     * @param secret      密钥
     */
    private static String signForWebSocketApi(String timestamp, String method, String requestPath, String secret) {
        String shaResource = timestamp + method + requestPath;
        System.out.println(shaResource);
        String signStr = sha256_HMAC(shaResource, secret);
        return signStr;
    }
    
  /**
   * sha256_HMAC加密
   *
   * @param resource 签名源字符串
   * @param secret   秘钥
   * @return 加密后字符串
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
   * 将加密后的字节数组转换成字符串
   *
   * @param bytes 字节数组
   * @return 字符串
   */
  private static String byteArrayToHexString(byte[] bytes) {
    StringBuffer buffer = new StringBuffer();
    String stmp;
    for (int index = 0; bytes != null && index < bytes.length; index++) {
      stmp = Integer.toHexString(bytes[index] & 0XFF);
      if (stmp.length() == 1) {
        // 一位补零
        buffer.append('0');
      }
      buffer.append(stmp);
    }
    return buffer.toString().toLowerCase();
  }
```

示例

```json
// send
{"op":"login","args":["0d999f8ba827adfc494931d9e1539df9","2019-07-04T02:19:08Z","3ded9d0113133c9f06cfa50ce99618e6d983a534f5a2219ebbe3ffb02b6fbe16"]}

// response
{"event":"login","success":true}
```

## 连接限制

### 心跳消息

当用户的WebSocket客户端连接到CoinBene服务器后，服务器会定期（当前为5秒）向其发送字符串`'ping'`来检测连接.WebSocket客户端接收到此心跳消息后，应返回文字字符串`'pong'`作为回应.

> 当WebSocket服务器连续两次发送了`ping`消息却没有收到任何一次`pong`消息返回后，服务器将主动断开与此客户端的连接。

### 限频规则

*连接限制*：1次/s

*订阅限制*：每小时240次

## Topic说明

公共Topic(无需登录)

| 名称       | 格式                       | 示例                 |
| ---------- | -------------------------- | -------------------- |
| 深度列表   | usdt/orderBook.{symbol}.{depth} | usdt/orderBook.BTC-SWAP.10 |
| 最新成交   | usdt/tradeList.{symbol}         | usdt/tradeList.BTC-SWAP    |
| Ticker信息 | usdt/ticker.{symbol}            | usdt/ticker.BTC-SWAP       |
| K线数据    | usdt/kline.{symbol}             | usdt/kline.BTC-SWAP        |

私有Topic(需登录)

| 名称     | 格式          |
| -------- | ------------- |
| 用户账户 | usdt/user.account  |
| 用户持仓 | usdt/user.position |
| 用户交易 | usdt/user.order    |



### 公共Topic

#### 深度列表

topic格式: `usdt/orderBook.{symbol}.{depth}`

*`symbol`*: 交易所支持的任一交易对
*`depth`* : 目前支持深度5、10、20、50、100

订阅时将`symbol`和`depth`设置成目标值即可.
首次返回指定深度(depth)的委托列表，后续为增量

示例

```json
// send  获取BTC-SWAP交易对10挡的深度列表
{"op": "subscribe", "args": ["usdt/orderBook.BTC-SWAP.10"]}

//response
// 首次10挡
{
    "topic": "usdt/orderBook.BTC-SWAP", 
    "action": "insert",
    "data": [{
        "asks": [
            ["5621.7", "58"], 
            ["5621.8", "125"],
            ["5621.9", "100"],
            ["5622", "84"],
            ["5623.5", "90"],
            ["5624.2", "1540"],
            ["5625.1", "300"],
            ["5625.9", "350"],
            ["5629.3", "200"],
            ["5650", "1000"]
        ],
        "bids": [
            ["5621.3", "287"],
            ["5621.2", "41"],
            ["5621.1", "2"],
            ["5621", "26"],
            ["5620.8", "194"],
            ["5620", "2"],
            ["5618.8", "204"],
            ["5618.4", "30"],
            ["5617.2", "2"],
            ["5609.9", "100"]
        ],
        "version":1,
        "timestamp": 1584412740809
    }]
 }
 //后续增量
{
    "topic": "usdt/orderBook.BTC-SWAP", 
    "action": "update", 
    "data": [{
        "asks": [
            ["5621.7", "50"],
            ["5621.8", "0"],
            ["5621.9", "30"]
        ],
        "bids": [
            ["5621.3", "10"],
            ["5621.2", "20"],
            ["5621.1", "80"],
            ["5621", "0"],
            ["5620.8", "10"]
        ],
        "version":2,
        "timestamp": 1584412740809
    }]
 }
```

> 首次`action = insert` 后续增量 `action = update`
> 一个挂单项是一个[price，size]的数组 例:["5621.7", "58"]  5621.7为深度价格，58为此价格数量
> version  数据版本严格递增 client端可以根据version来判断数据是否连续

| 参数名    | 参数类型 | 描述     |
| --------- | -------- | -------- |
| symbol    | string   | 交易对   |
| asks      | string   | 卖方深度 |
| bids      | string   | 买方深度 |
| version   | number   | 数据版本 |
| timestamp | number   | 时间戳(毫秒)   |

* 本地维护orderBook副本(参考)

  > 1. 订阅`orderBook.{symbol}.{depth}`首次获取到{depth}深度orderBook副本
  > 2. 对后续的增量推送,以price为key按如下规则处理
  >     * 如果出现新的price，说明有用户按新的价格挂单，此时需要将挂单项插入到挂单列表中,同时保证列表顺序
  >     * price已存在，并且最新size值不为0，说明该价格的挂单刚刚进行了交易，此时需要用新的size值替换旧的size值
  >     * price已存在，并且size值为0，说明该价位的挂单已经撤单或者被吃，应该移除这个价位

#### 最新成交

topic格式: `usdt/tradeList.{symbol}`

*`symbol`*: 交易所支持的任一交易对

示例

```json
// send
{"op": "subscribe", "args": ["usdt/tradeList.BTC-SWAP"]}

// response
{
    "topic": "usdt/tradeList.BTC-SWAP",
    "data": [  
      [
        "8600.0000", 
        "s", 
        "100", 
        1584412740809
      ]
    ]
 }
// 一个数据项是一个[price,side,volume,timestamp]的数组
```

返回参数

| 参数名    | 参数类型 | 描述                     |
| --------- | -------- | ------------------------ |
| price     | string   | 成交价格                 |
| volume    | string   | 成交数量                 |
| side      | string   | 成交方向，s=主卖，b=主买 |
| timestamp | number   | 成交时间                 |



#### Ticker信息

获取平台合约的最新成交价、买一价、卖一价和24交易量

topic格式: `usdt/ticker.{symbol}`

*`symbol`:* 交易所支持的任一交易对

示例

```json
// send
{"op": "subscribe", "args": ["usdt/ticker.BTC-SWAP","usdt/ticker.ETH-SWAP"]}

// response
{
    "topic": "usdt/ticker.BTC-SWAP",
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
          "timestamp": 1584412736365
        }
    ]
 }
```

返回参数

| 参数名       | 参数类型 | 描述                |
| ------------ | -------- | ------------------- |
| symbol       | string   | 合约名称，如BTC-SWAP |
| bestAskPrice | string   | 卖一价              |
| bestAskSize  | string   | 卖一量              |
| bestBidPrice | string   | 买一价              |
| bestBidSize  | string   | 买一量              |
| lastPrice    | string   | 最新价              |
| markPrice    | string   | 标记价格            |
| high24h      | string   | 24h最高价           |
| low24h       | string   | 24h最低价           |
| volume24h    | string   | 24h成交量USDT       |

#### K线数据

获取平台合约的K线数据

topic格式: `usdt/kline.{symbol}`

*`symbol:`*交易所支持的任一交易对

示例

```json
// send
{"op": "subscribe", "args": ["usdt/kline.BTC-SWAP","usdt/kline.ETH-SWAP"]}

// response
{
    "topic": "usdt/kline.BTC-SWAP",
    "data": [
        {
          "c": 7513.01,
          "h": 7513.37,
          "l": 7510.02,
          "o": 7510.24,
          "m": 7512.03,
          "v": 60.5929,
          "t": 1578278880
  			 }
    ]
 }
```

返回参数

| 参数名      | 参数类型 | 描述                |
| ----------- | -------- | ------------------- |
| c      | number   | 收盘价格            |
| h        | number   | 最高价格            |
| l        | number   | 最低价格            |
| o        | number   | 开盘价格            |
| v      | number   | 成交量（张）        |
| t        | number   | 时间戳            |



### 私有Topic

如下私有Topic在订阅时需要先进行登录操作.

#### 用户账户

获取账户信息，需先登录

topic格式: `usdt/user.account`

示例

```json
// send
{"op": "subscribe", "args": ["usdt/user.account"]}

// response
{
    "topic": "usdt/user.account",
    "data": [{
        "asset": "BTC",
        "availableBalance": "20.3859", 
        "frozenBalance": "0.7413",
        "balance": "21.1272", 
        "timestamp": "2019-05-22T03:11:22.0Z"
    }]
}
```

返回参数

| 参数名           | 参数类型 | 描述     |
| ---------------- | -------- | -------- |
| asset            | string   | 资产     |
| availableBalance | string   | 可用余额 |
| frozenBalance    | string   | 冻结额度 |
| balance          | string   | 账户余额 |



#### 用户持仓

获取用户持仓信息，需先登录
topic格式: `usdt/user.position`

示例

```json
// send
{"op": "subscribe", "args": ["usdt/user.position"]}

// response
{
    "topic": "usdt/user.position",
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

返回参数

| 参数名            | 参数类型 | 描述                    |
| ----------------- | -------- | ----------------------- |
| availableQuantity | string   | 可平仓数量              |
| avgPrice          | string   | 开仓均价                |
| leverage          | string   | 杠杆倍数                |
| liquidationPrice  | string   | 强平价格                |
| markPrice         | string   | 标记价格                |
| postionMargin     | string   | 仓位保证金              |
| quantity          | string   | 合约的持仓数量          |
| realisedPnl       | string   | 已实现盈亏              |
| side              | string   | 方向                    |
| symbol            | string   | 合约名称                |
| marginMode        | string   | 保证金模式  1全仓 0逐仓 |
| createTime        | string   | 仓位创建时间            |



#### 用户交易

获取用户交易信息，需先登录
topic格式: `usdt/user.order`

示例

```json
// send
{"op": "subscribe", "args": ["usdt/user.order"]}

// response
{
    "topic": "user.order",
    "data": [{
      "orderId": "580721369818955776", 
      "direction": "openLong", 
      "leverage": "20", 
      "symbol": "ETH-SWAP", 
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

| 参数名           | 参数类型 | 描述                                                         |
| ---------------- | -------- | ------------------------------------------------------------ |
| orderId          | string   | 订单Id                                                       |
| direction        | string   | 方向                                                         |
| leverage         | string   | 杠杆倍数                                                     |
| orderType        | string   | 订单类型, 限价=limit 市价=market                             |
| quantity         | string   | 委托量（张）                                                 |
| orderPrice       | string   | 订单价格                                                     |
| orderValue       | string   | 订单价值                                                     |
| fee              | string   | 手续费                                                       |
| filledQuantity   | string   | 成交量（张）                                                 |
| averagePrice     | string   | 平均成交价格                                                 |
| orderTime        | string   | 订单创建时间                                                 |
| lastFillPrice    | string   | 最新成交价格 （如果没有，推0）                               |
| lastFillQuantity | string   | 最新成交数量 （如果没有，推0）                               |
| lastFillTime     | string   | 最新成交时间 （如果没有，推 ""）                             |
| status           | string   | 订单状态(new:挂单中,filled:完成成交,canceled:完全撤单,partiallyFilled:部分撤单 |

## 错误代码汇总

| 错误代码 | 描述                  |
| -------- | :-------------------- |
| 429      | 请求太频繁            |
| 10501    | ping检测超时          |
| 10502    | 请求体错误            |
| 10503    | Topic不支持           |
| 10504    | 未登录                |
| 10505    | 签名错误              |
| 10506    | 参数错误              |
| 10507    | expires错误           |
| 10508    | App Id 错误           |
| 10509    | 不支持的orderBook深度 |
| 10500    | 系统错误              |
