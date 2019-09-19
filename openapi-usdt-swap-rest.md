# coinbene-swap-rest 合约openapi rest接口说明
# [English](https://github.com/Coinbene/API-SWAP-Documents/blob/master/openapi-swap-rest-en.md)
# [WebSocket](https://github.com/Coinbene/API-SWAP-Documents/blob/master/openapi-swap-websocket.md)
* [coinbene-swap-rest行情与交易接口](#coinbene-swap-rest行情与交易接口)
  * [基本信息](#基本信息)
  * [访问限制](#访问限制)
  * [接口类型](#接口类型)
  * [签名方式](#签名方式)
  * [接口规范](#接口规范)
  	* [公共接口-获取深度](#公共接口-获取深度)
  	* [公共接口-获取全部ticker信息](#公共接口-获取全部ticker信息)
  	* [公共接口-获取K线数据](#公共接口-获取k线数据)
  	* [公共接口-查询最新成交信息](#公共接口-查询最新成交信息)
	* [公共接口-获取当前合约最新资金费率](#公共接口-获取当前合约最新资金费率)
  	* [私有接口-查询合约账户信息](#私有接口-查询合约账户信息)
  	* [私有接口-合约持仓信息](#私有接口-合约持仓信息)
  	* [私有接口-下单](#私有接口-下单)
  	* [私有接口-指定撤单](#私有接口-指定撤单)
  	* [私有接口-查询所有当前委托单列表](#私有接口-查询所有当前委托单列表)
	* [私有接口-按订单ID分页查询当前委托单列表](#私有接口-按订单ID分页查询当前委托单列表)
 	* [私有接口-获取指定订单信息](#私有接口-获取指定订单信息)
  	* [私有接口-查询历史订单](#私有接口-查询历史订单)
	* [私有接口-按订单ID分页查询历史订单](#私有接口-按订单ID分页查询历史订单)
  	* [私有接口-批量撤单](#私有接口-批量撤单)
	* [私有接口-获取指定订单成交明细](#私有接口-获取指定订单成交明细)
	* [私有接口-获取资金费率列表](#私有接口-获取资金费率列表)
  * [错误代码汇总](#错误代码汇总)

## 基本信息
- 本篇列出REST接口的baseurl http://openapi-contract.coinbene.com
- 需要科学上网，国内用户建议机器绑定host，104.16.127.19 openapi-contract.coinbene.com
- 我们服务器部署在美西，建议api机器人服务器也部署在美西减少网络延迟
- 建议创建完API后，修改添加上自己服务器出口IP，进一步增强API安全性校验
- 所有接口的响应都是JSON格式
- 所有时间、时间戳均为UNIX时间，单位为毫秒
- HTTP 4XX 错误码用于指示错误的请求内容、行为、格式。
- HTTP 429 错误码表示警告访问频次超限，即将被封IP
- HTTP 418 表示收到429后继续访问，于是被封了。
- HTTP 5XX 错误码用于指示Coinbene服务侧的问题。
- HTTP 504 表示API服务端已经向业务核心提交了请求但未能获取响应，特别需要注意的是504代码不代表请求失败，而是未知。很可能已经得到了执行，也有可能执行失败，需要做进一步确认。
- 每个接口都有可能抛出异常，异常响应格式如下：

```
{
  "code": 10001,
  "msg": "Invalid Paramater."
}
```
- 具体的错误码及其解释在错误代码汇总。
- GET方法的接口, 参数必须在query string中发送。
- POST方法的接口, 参数在 request body中发送(content type application/json)。
- 对参数的顺序不做要求。
## 访问限制
- 当访问接口超过频率限制时，将返回429状态：请求太频繁。
- 限制规则，如果传入有效的API key则用user id限速；如果没有则拿用户的公网IP限速。各个接口上有单独的说明。
## 接口类型
- 主要为两类接口，公共接口和私有接口。
- 公共接口无需认证即可调用。
- 私有接口用户订单和账户。每个私有请求必须使用规范的验证形式进行签名。私有接口需要使用您的API key进行验证。
## 签名方式
所有接口请求头必须包含以下内容：
- ACCESS-KEY 字符串类型的API key
- ACCESS-SIGN 使用Hex生成字符串
- ACCESS-TIMESTAMP 发起请求的时间戳
- 所有请求都应该含有application/json类型内容，并且是有效的JSON。

ACCESS-SIGN的值生成规则：
- 按照timestamp + method + requestPath + body字符串（+表示字符串连接），以及secret，使用HMAC SHA256方法加密，最后把加密串的字节数组转成字符串返回。
- 其中，timestamp的值与ACCESS-TIMESTAMP请求头相同，必须是UTC时区Unix时间戳的十进制秒数或ISO8601标准的时间格式，精确到毫秒。
- Method是请求方法，字母全部大写：GET/POST
- requestPath是请求接口路径，例如：/api/swap/v2/market/orderBook
- body是指请求主体的字符串。GET请求没有body信息可省略；POST请求有body信息JSON串，例如{"symbol":"BTCUSDT","order_id":"7440"}
- secret为用户申请API时所生成的
- 任何时候都请不要把secret透露给其他人或传输到服务器端

接口请求样例：
- GET协议接口两种情况: 
```
1. 不带参数：
preHash String：2019-03-08T10:59:25.789ZGET/account/list
2. 带参数：
preHash String：2019-03-08T10:59:25.789ZGET/account/list?symbol=BTCUSDT
```


```
Url: http://域名/api/swap/v2/market/tickers
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
Url: http://域名/api/swap/v2/market/orderBook?symbol=ETHUSDT&size=10
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


- POST协议接口情况：
```
preHash String：2019-03-08T10:59:25.789ZPOST/account/add{"symbol":"BTCUSDT","quantity":"70"}
```


```
Url: http://域名/api/swap/v2/order/place
Method: POST
Headers: 
	Accept: application/json
	ACCESS-KEY: E65791902180E9EF4510DB6A77F6EBAE
	ACCESS-SIGN: hhd/F4LFj/YAA5SC7x0gtSHxI0U9+VqD+orR1VMdofg=
	ACCESS-TIMESTAMP: 2019-05-22T03:33:53.562Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: {"symbol":"ETHUSDT","orderType":"limit","leverage":"20","orderPrice":"147.7","quantity":"7","direction":"openLong","clientId":"1558496033481"}
preHash: 2019-05-22T03:33:53.562ZPOST/api/swap/v2/order/place{"symbol":"ETHUSDT","orderType":"limit","leverage":"20","orderPrice":"147.7","quantity":"7","direction":"openLong","clientId":"1558496033481"}
```
- 签名算法验证：


```
源串：2019-05-25T03:20:30.362ZGET/api/swap/v2/account/info
secret：9daf13ebd76c4f358fc885ca6ede5e27
生成sign串：a02a6428bb44ad338d020c55acee9dd40bbcb3d96cbe3e48dd6185e51e232aa2

样例代码（Java版本）：
**
   * 生成签名
   *
   * @param method      请求方法：POST或者GET
   * @param requestUrl  url
   * @param requestBody 请求内容，没有传null
   * @param secret      密钥
   */
  private String signForContractOpenApi(String method, String requestUrl, String requestBody, String secret) {
    final DateTimeFormatter utcFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'");
    String timestamp = utcFormatter.format(ZonedDateTime.now(ZoneOffset.UTC));
    //NEVER use ZonedDateTime.now(ZoneOffset.UTC).toString(), 
    //which may produce timestamp like 'yyyy-MM-dd'T'HH:mm:ss.SSSSSS'Z', but it depends.
    String shaResource = timeStamp + method + requestUrl + (requestBody == null ? "" : requestBody);
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

样例代码（Python版本）：

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



样例代码（Kotlin版本）：
private fun ByteArray.toHex() = this.joinToString(separator = "") { it.toInt().and(0xff).toString(16).padStart(2, '0') }
private fun String.sha256(secretKey: String): ByteArray = HmacUtils.hmacSha256(secretKey, this)
private val utcFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'")
**
   * 生成签名
   *
   * @param method      请求方法：POST或者GET
   * @param requestUrl  url
   * @param requestBody 请求内容，没有传null
   * @param secret      密钥
   */
fun signForContractOpenApi(method: String, requestUrl: String, requestBody: String, secret:String) {
    //NEVER use ZonedDateTime.now(ZoneOffset.UTC).toString(), 
    //which may produce timestamp like 'yyyy-MM-dd'T'HH:mm:ss.SSSSSS'Z', but it depends.
    val timestamp = utcFormatter.format(ZonedDateTime.now(ZoneOffset.UTC))
    retrun "$timestamp${method.toUpperCase()}$requestUrl${body ?: ""}"
                .sha256(apiSecret)
                .toHex()
}
```


## 接口规范
### 公共接口-获取深度
```
获取合约的深度列表
限速规则：20次/2秒
HTTP GET /api/swap/v2/market/orderBook?symbol=BTCUSDT
```

请求参数：

名称  | 类型  | 是否必填  | 说明 |
---------|--------|---------|--------|
symbol | string | 是 | 合约名称，如BTCUSDT |
size   | string | 否 | 深度档位，值有5、10、50、100。默认值10|

返回字段说明：

名称   | 类型  | 说明 |
---------|---------|---------|
asks   | array | 卖方深度，[档位价格，数量，该深度由几笔订单组成]|
bids   | array | 买方深度，[档位价格，数量，该深度由几笔订单组成]|
symbol      | string |  合约名称
time      | string |  时间戳，国际时间



```
Request:
Url: http://域名/api/swap/v2/market/orderBook?symbol=ETHUSDT&size=10
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
### 公共接口-获取全部ticker信息

```
获取平台全部合约的最新成交价、买一价、卖一价和24交易量
限速规则：20次/2秒
HTTP GET /api/swap/v2/market/tickers
```
请求参数：无

返回字段说明：

名称   | 类型  | 说明
---------|---------|---------|
symbol         | string | 合约名称，如BTCUSDT
bestAskPrice   | string | 卖一价
bestAskSize    | string | 卖一量
bestBidPrice   | string | 买一价
bestBidSize    | string | 买一量
lastPrice    | string | 最新价
markPrice      | string | 标记价格
high24h        | string | 24h最高价
low24h         | string | 24h最低价
volume24h      | string | 24h成交量USDT
turnover      | string |  
time      | string |  时间戳，国际时间


```
Request:
Url: http://域名/api/swap/v2/market/tickers
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
      "turnover": "9988", 
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
### 公共接口-获取K线数据

```
获取合约K线数据。K线数据最多可获取2000条。
限速规则：20次/2秒
HTTP GET /api/swap/v2/market/klines
```
请求参数：

名称  | 类型  | 是否必填  | 说明 |
---------|---------|---------|---------|
symbol      | string | 是 | 合约名称，如BTCUSDT |
startTime   | string | 是 | 开始时间，ISO8601格式时间戳 到秒 |
endTime     | string | 是 |截止时间，ISO8601格式时间戳 到秒|
resolution  | string | 是 | Kline粒度，取值范围参考说明|


```
resolution的值只能取["1", "3", "5", "15", "30",
"60", "120", "240", "360", "720", "D", "W", "M"]，否则请求将会被拒绝，
分别对应 [1min, 3min, 5min, 15min, 30min, 
1hour, 2hour, 4hour, 6hour, 12hour, 1day, 1week, 1month]的时间段
```


返回字段说明：

名称   | 类型  | 说明
---------|---------|---------|
time   | string | 生成时间
open   | string | 开盘价格
close   | string | 收盘价格
high   | string | 最高价格
low   | string | 最低价格
volume   | string | 成交量（张）
turnover   | string | 交易额
buyVolume   | string | 主买量（张）
buyTurnover   | string | 主买额


```
Request:
Url: http://域名/api/swap/v2/market/klines?symbol=BTCUSDT&resolution=1&startTime=1557425760&endTime=1557425820
Method: GET
Headers: 
	Accept: application/json
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-21T11:16:20.521ZGET/api/swap/v2/market/klines?symbol=BTCUSDT&resolution=1&startTime=1557425760&endTime=1557425820

Response:
格式说明:[time,open,close,high,low,volume,turnover,buyVolume,buyTurnover]
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
### 公共接口-查询最新成交信息

```
获取合约的最新成交信息
限速规则：20次/2秒
HTTP GET /api/swap/v2/market/trades
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---------|---------|---------|---------|
symbol      | string | 是 | 合约名称，如BTCUSDT
limit  | string | 否 | 返回记录数，默认10，最大100

返回字段说明：

名称   | 类型  | 说明
---------|---------|---------|
price   | string | 成交价格
side   | string | 成交方向，s=主卖，b=主买
volume   | string | 成交量（张）
time   | string | 成交时间


```
Request:
Url: http://域名/api/swap/v2/market/trades?symbol=BTCUSDT&limit=1
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

### 公共接口-获取当前合约最新资金费率

```
获取当前合约最新资金费率
限速规则：6次/1秒
HTTP GET /api/swap/v2/market/fundingRate
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---------|---------|---------|---------|
symbol      | string | 是 | 合约名称，如BTCUSDT

返回字段说明：无


```
Request:
Url: http://域名/api/swap/v2/market/fundingRate?symbol=BTCUSDT
Method: GET
Headers: 
	Accept: application/json
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-21T11:19:52.303ZGET/api/swap/v2/market/fundingRate?symbol=BTCUSDT

Response:
{
  "code": 200, 
  "data": "0.00375"
}
```

### 私有接口-查询合约账户信息

```
获取用户币种合约的账户信息
限速次数：10次/2秒
HTTP GET /api/swap/v2/account/info
```

请求参数
无

返回结果参数

名称   | 类型  | 说明
---------|---------|---------|
availableBalance   | string | 可用余额
frozenBalance   | string | 冻结资产
balance   | string | 账户余额
marginRatio   | string | 保证金率
unrealisedPnl   | string | 未实现盈亏


```
Request:
Url: http://域名/api/swap/v2/account/info
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
### 私有接口-合约持仓信息


```
获取所有合约的持仓信息
限速规则：10次/2秒
HTTP GET /api/swap/v2/position/list
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---------|---------|---------|---------|
symbol      | string | 否 | 合约名称，如BTCUSDT

返回字段说明：

名称   | 类型  | 说明
---------|---------|---------|
availableQuantity   | string | 可平仓数量
averagePrice   | string | 开仓均价
createTime   | string | 仓位创建时间
deleveragePercentile | string  | 减仓队列，值越大也排名越靠前 
leverage   | string | 杠杆倍数
liquidationPrice   | string | 强平价格
markPrice   | string | 标记价格
postionMargin   | string | 仓位保证金
positionValue   | string | 仓位的BTC价值
quantity   | string | 合约的持仓数量
realisedPnl   | string | 已实现盈亏
roe   | string | 回报率
side   | string | 方向
symbol   | string | 合约名称
unrealisedPnl   | string | 未实现盈亏


```
Request:
Url: http://域名/api/swap/v2/position/list
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

### 私有接口-下单

```
按用户输入进行下单操作
限速规则：20次/2秒
HTTP POST/api/swap/v2/order/place
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---------|---------|---------|---------|
symbol      | string | 是 | 合约名称，如BTCUSDT
direction      | string | 是 | 方向
leverage      | string | 是 | 杠杆倍数
orderType      | string | 是 | 订单类型，默认为limit，其他枚举值：postOnly（只做maker）、fok（Fill or Kill）、ioc（Immediate Or Cancel）
orderPrice      | string | 是 | 下单价格
quantity      | string | 是 | 买入或卖出合约数量（张数）
marginMode      | string | 否 | 仓位模式，默认值crossed， fixed逐仓 crossed全仓
clientId      | string | 否 | 用户请求id，透传返回给用户


```
1.direction取值说明：openLong=开多 2.openShort=开空closeLong=平多 closeShort=平空
3.leverage取值范围：2, 3, 5, 10, 20
4.orderPrice精度说明：BTCUSDT价格精度是0.5；ETHUSDT价格精度是0.05
```

返回字段说明：

名称   | 类型  | 说明
---------|---------|---------|
orderId   | string | 生成的订单id
clientId   | string | 用户请求的clientId


```
Request:
Url: http://域名/api/swap/v2/order/place
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

### 私有接口-指定撤单

```
按用户输入进行撤单操作
限速规则：20次/2秒
HTTP POST/api/swap/v2/order/cancel
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---------|---------|---------|---------|
orderId      | string | 是 | 订单ID

返回字段说明：

名称   | 类型  | 说明
---------|---------|---------|
data   | string | 撤销的订单Id

```
Request:
Url: http://域名/api/swap/v2/order/cancel
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
### 私有接口-查询所有当前委托单列表


```
按用户请求进行订单列表查询，
限速规则：5次/2秒
HTTP GET/api/swap/v2/order/openOrders
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---------|---------|---------|---------|
symbol      | string | 否 | 合约名称，如BTCUSDT
pageNum      | string | 否 | 页码，默认第1页
pageSize      | string | 否 | 单页记录数，默认10


```
说明
1.最大支持同时下50个订单
2.假定一共下了50个订单，并且这些订单都尚未成交，使用参数pageNum=5, pageSize=10，则返回[41, 50]范围内的订单
```


返回字段说明：

名称   | 类型  | 说明
---------|---------|---------|
orderId   | string | 订单Id
direction   | string | 方向
leverage   | string | 杠杆倍数
orderType   | string | 订单类型, 限价=limit 市价=market
quantity   | string | 成交量（张）
orderPrice   | string | 订单价格
orderValue   | string | 订单价值
fee   | string | 手续费
filledQuantity   | string | 成交量（张）
averagePrice   | string | 平均成交价格
orderTime   | string | 订单创建时间
status   | string | 订单状态(new:挂单中,filled:完成成交,canceled:完全撤单,partiallyCanceled:部分撤单）


```
Request:
Url: http://域名/api/swap/v2/order/openOrders?symbol=ETHUSDT&pageNum=1&pageSize=3
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

### 私有接口-按订单ID分页查询当前委托单列表


```
按用户请求进行订单列表查询，
限速规则：5次/2秒
HTTP GET/api/swap/v2/order/openOrdersByPage
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---------|---------|---------|---------|
symbol      | string | 否 | 合约名称，如BTCUSDT
latestOrderId      | string | 否 | 订单ID。默认为空，返回最新20条数据记录

返回字段说明：

名称   | 类型  | 说明
---------|---------|---------|
orderId   | string | 订单Id
direction   | string | 方向
leverage   | string | 杠杆倍数
orderType   | string | 订单类型, 限价=limit 市价=market
quantity   | string | 成交量（张）
orderPrice   | string | 订单价格
orderValue   | string | 订单价值
fee   | string | 手续费
filledQuantity   | string | 成交量（张）
averagePrice   | string | 平均成交价格
orderTime   | string | 订单创建时间
status   | string | 订单状态(new:挂单中,filled:完成成交,canceled:完全撤单,partiallyFilled:部分撤单）


```
Request:
Url: http://域名/api/swap/v2/order/openOrdersByPage
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

### 私有接口-获取指定订单信息

```
通过订单ID获取单个订单信息
限速规则：10次/2秒
HTTP GET/api/swap/v2/order/info
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---------|---------|---------|---------|
orderId      | string | 是 | 订单id

返回字段说明：

名称   | 类型  | 说明
---------|---------|---------|
orderId   | string | 订单Id
direction   | string | 方向
leverage   | string | 杠杆倍数
orderType   | string | 订单类型，限价=limit 市价=market
quantity   | string | 成交量（张）
orderPrice   | string | 订单价格
orderValue   | string | 订单价值
fee   | string | 手续费
filledQuantity   | string | 成交量（张）
averagePrice   | string | 平均成交价格
orderTime   | string | 订单创建时间
status   | string | 订单状态(new:挂单中,filled:完成成交,canceled:完全撤单,partiallyCanceled:部分撤单）



```
Request:
Url: http://域名/api/swap/v2/order/info?orderId=580721369818955776
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

### 私有接口-查询历史订单

```
按用户输入进行查询操作
限速规则：5次/2秒
HTTP GET/api/swap/v2/order/closedOrders
```

请求参数：

名称  | 类型  | 是否必填  | 说明
---------|---------|---------|---------|
beginTime      | string | 否 | 开始时间，毫秒级时间戳
endTime      | string | 否 | 结束时间，毫秒级时间戳
symbol      | string | 是 | 合约名称，如BTCUSDT
pageNum      | string | 否 | 页码，默认值 1
pageSize      | string | 否 | 单页条数，默认值 10
direction      | string | 否 | openLong=开多 openShort=开空
orderType      | string | 否 | 订单类型


返回字段说明：

名称   | 类型  | 说明
---------|---------|---------|
orderId   | string | 订单Id
direction   | string | 方向
leverage   | string | 杠杆倍数
orderType   | string | 订单类型，限价=limit 市价=market
quantity   | string | 成交量（张）
orderPrice   | string | 订单价格
orderValue   | string | 订单价值
fee   | string | 手续费
filledQuantity   | string | 成交量（张）
averagePrice   | string | 平均成交价格
orderTime   | string | 订单创建时间
status   | string | 订单状态(filled:完成成交,canceled:完全撤单,partiallyCanceled:部分撤单）


```
Request:
Url: http://域名/api/swap/v2/order/closedOrders?symbol=ETHUSDT&pageNum=1&pageSize=10
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

### 私有接口-按订单id分页查询历史订单

```
按用户输入进行查询操作
限速规则：5次/2秒
HTTP GET/api/swap/v2/order/closedOrdersByPage
```

请求参数：

名称  | 类型  | 是否必填  | 说明
---------|---------|---------|---------|
beginTime      | string | 否 | 开始时间，毫秒级时间戳
endTime      | string | 否 | 结束时间，毫秒级时间戳
symbol      | string | 否 | 合约名称，如BTCUSDT
status   | string | 订单状态(filled:完成成交,canceled:完全撤单,partiallyCanceled:部分撤单）
latestOrderId      | string | 否 | 订单id。默认为空，返回最新20条数据记录


返回字段说明：

名称   | 类型  | 说明
---------|---------|---------|
orderId   | string | 订单Id
direction   | string | 方向
leverage   | string | 杠杆倍数
orderType   | string | 订单类型，限价=limit 市价=market
quantity   | string | 成交量（张）
orderPrice   | string | 订单价格
orderValue   | string | 订单价值
fee   | string | 手续费
filledQuantity   | string | 成交量（张）
averagePrice   | string | 平均成交价格
orderTime   | string | 订单创建时间
status   | string | 订单状态(new:挂单中,filled:完成成交,canceled:完全撤单,partiallyCanceled:部分撤单）


```
Request:
Url: http://域名/api/swap/v2/order/closedOrdersByPage?symbol=&beginTime=1560658928499
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: a342c5b9a9e0e39d33101bf81e99b6edf8cc42b2bb07fe4969e76f1f6e235994
	ACCESS-TIMESTAMP: 2019-06-26T04:22:08.518Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-06-26T04:22:08.518ZGET/api/swap/v2/order/closedOrdersByPage?symbol=&beginTime=1560658928499

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

### 私有接口-批量撤单

```
按用户输入进行撤单操作
限速规则：5次/2秒
HTTP POST/api/swap/v2/order/batchCancel
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---------|---------|---------|---------|
orderIds      | list<string> | 是 | 订单Id列表，每次最多10个订单id

返回字段说明：

名称   | 类型  | 说明
---------|---------|---------|
orderId   | string | 撤销的订单Id


```
Request:
Url: http://域名/api/swap/v2/order/batchCancel
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

### 私有接口-获取指定订单成交明细

```
按用户输入进行撤单操作
限速规则：10次/2秒
HTTP GET/api/swap/v2/order/fills
```

请求参数：

名称  | 类型  | 是否必填  | 说明
---------|---------|---------|---------|
symbol      | string | 是 | 合约名称，如BTCUSDT
orderId      | string | 是 | 订单Id
lastTradeId      | string | 否 | 成交Id，分页使用，每页固定20条，默认0返回最新20条

返回字段说明：

名称   | 类型  | 说明
---------|---------|---------|
orderId   | string | 订单Id
direction   | string | 订单方向
leverage   | string | 杠杆倍数
symbol   | string | 合约名称，如BTCUSDT
orderType   | string | 订单类型，限价=limit 市价=market
quantity   | string | 订单数量（张）
orderPrice   | string | 订单价格
orderValue   | string | 订单价值
fee   | string | 手续费
filledQuantity   | string | 订单已成交数量（张）
averagePrice   | string | 订单平均价格
orderTime   | string | 订单时间
status   | string | 订单状态(new:挂单中,filled:完成成交,canceled:完全撤单,partiallyCanceled:部分撤单）

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

### 私有接口-获取资金费率列表

```
按用户输入进行撤单操作
限速规则：10次/2秒
HTTP GET/api/swap/v2/position/feeRate
```

请求参数：

名称  | 类型  | 是否必填  | 说明
---------|---------|---------|---------|
pageNum      | string | 否 | 默认值1，表示第一页开始
pageSize      | string | 否 | 默认值10，每页返回记录条数

返回字段说明：

名称   | 类型  | 说明
---------|---------|---------|
symbol   | string | 合约名称，如BTCUSDT
side   | string | 方向
markPrice   | string | 标记价格
positionValue   | string | 仓位价值
fee   | string | 手续费
feeRate   | string | 费率
time   | string | 收取时间
leverage   | string | 杠杆倍数

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


## 错误代码汇总

错误代码     |  message
---|:---
429 |请求太频繁
10001 | "ACCESS_KEY"不能为空
10002 | "ACCESS_SIGN"不能为空
10003 | "ACCESS_TIMESTAMP"不能为空
10005 | 无效的ACCESS_TIMESTAMP
10006 | 无效的ACCESS_KEY
10007 | 无效的Content_Type，请使用“application/json”格式
10008 | 请求时间戳过期
10009 | 系统错误
100011 |无效的sign
100012 |api 校验失败
11000 |{0}必填参数为空
11001 |{0}参数值不合法
11002 |{0}参数值超过最大值限制
10200 | 余额不足
10201 |   账户出错了
10202 |   参数错误
10204 |   账户不存在
10300 |   下单失败
10301 |   参数错误
10302 |   最小下单量是{0}
10303 |   最大下单量是{0}
10304 |   最小价格变化是{0}
10305 |   一个合约下进行中委托数量上限为{0}
10306 |   已存在使用的杠杆{0}
10307 |   账户余额不足
10308 |   委托价格限价在{0}将导致仓位被强平，请调整委托价格
10309 |   撤单失败
10310 |   已成交撤单失败
10311 |   平仓数量不能大于剩余可平仓位
10312 |   平仓委托价格不能突破强平价格
10313 |   当前市价开仓将导致持仓风险过高，容易被强平，请使用限价委托
10314 |  查询失败
10315 |   委托价格跟当前标记价格偏差不能超过50%
10316 |   合约不存在
10317 |   用户不存在
10318 |   杠杆倍数超过允许范围
10318 |   超过最大撤单数量
10319 |   用户被冻结
10320 |   订单类型不合适
10321 |   订单方向错误
10321|  订单类型不合适
10322|  订单方向错误
10323|  必传参数为空
10324|  仓位不存在
10325|  订单不存在
10326|  限价价格不能为空
10329|  委托在{0}成交将导致仓位保证金亏损至最低维持仓位保证金水平 请调整委托价格或者降低杠杆
10332|  保证金模式错误
10333|  频繁下小于10张的订单，禁用下单功能10分钟

