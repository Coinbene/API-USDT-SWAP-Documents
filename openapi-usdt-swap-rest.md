# coinbene-usdt-rest 合约openapi rest接口说明
      
 * [coinbene-usdt-rest 合约openapi rest接口说明](#coinbene-usdt-rest-合约openapi-rest接口说明)
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
      * [错误代码汇总](#错误代码汇总)


## 基本信息
- 本篇列出REST接口的baseurl https://openapi-contract.coinbene.com
- 我们服务器部署在美西，建议api机器人服务器也部署在美西减少网络延迟
- 建议创建完API后，修改添加上自己服务器出口IP，进一步增强API安全性校验
- 所有接口的响应都是JSON格式
- 所有时间、时间戳均为UNIX时间，单位为毫秒
- HTTP 4XX 错误码用于指示错误的请求内容、行为、格式。
- HTTP 429 错误码表示警告访问频次超限，即将被封IP
- HTTP 418 表示收到429后继续访问，于是被封了。
- HTTP 5XX 错误码用于指示Coinbene服务侧的问题。
- 每个接口都有可能抛出异常，异常响应格式如下：

```
{
  "code": xxxx,
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
- requestPath是请求接口路径，例如：/api/usdt/v2/market/orderBook
- body是指请求主体的字符串。GET请求没有body信息可省略；POST请求有body信息JSON串，例如{"symbol":"BTC-SWAP","order_id":"7440"}
- secret为用户申请API时所生成的
- 任何时候都请不要把secret透露给其他人或传输到服务器端

接口请求样例：
- GET协议接口两种情况: 
```
1. 不带参数：
preHash String：2019-03-08T10:59:25.789ZGET/account/list
2. 带参数：
preHash String：2019-03-08T10:59:25.789ZGET/account/list?symbol=BTC-SWAP
```


```
Url: http://域名/api/usdt/v2/market/tickers
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: E65791902180E9EF4510DB6A77F6EBAE
	ACCESS-SIGN: SswsZUURIjFke/rBHohJ1IFw0J7f67R08WpGZGGFaYI=
	ACCESS-TIMESTAMP: 2019-05-21T11:14:16.161Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-21T11:14:16.161ZGET/api/usdt/v2/market/tickers
```


```
Url: http://域名/api/usdt/v2/market/orderBook?symbol=BTC-SWAP&size=10
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: E65791902180E9EF4510DB6A77F6EBAE
	ACCESS-SIGN: U3de14fFE9uOhpnBgNHxLtXMBCV7813K/VonpKDWqZE=
	ACCESS-TIMESTAMP: 2019-05-21T11:10:28.464Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-21T11:10:28.464ZGET/api/usdt/v2/market/orderBook?symbol=ETHBTC-SWAPUSDT&size=10
```


- POST协议接口情况：
```
preHash String：2019-03-08T10:59:25.789ZPOST/account/add{"symbol":"BTC-SWAP","quantity":"70"}
```


```
Url: http://域名/api/usdt/v2/order/place
Method: POST
Headers: 
	Accept: application/json
	ACCESS-KEY: E65791902180E9EF4510DB6A77F6EBAE
	ACCESS-SIGN: hhd/F4LFj/YAA5SC7x0gtSHxI0U9+VqD+orR1VMdofg=
	ACCESS-TIMESTAMP: 2019-05-22T03:33:53.562Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: {"symbol":"BTC-SWAP","orderType":"limit","leverage":"20","orderPrice":"147.7","quantity":"7","direction":"openLong","clientId":"1558496033481"}
preHash: 2019-05-22T03:33:53.562ZPOST/api/usdt/v2/order/place{"symbol":"BTC-SWAP","orderType":"limit","leverage":"20","orderPrice":"147.7","quantity":"7","direction":"openLong","clientId":"1558496033481"}
```
- 签名算法验证：


```
源串：2019-05-25T03:20:30.362ZGET/api/usdt/v2/account/info
secret：9daf13ebd76c4f358fc885ca6ede5e27
生成sign串：9e77c73cba34ec465ebc7cc9dfe448c0c377f0663cdbb7bbe8fd379d1ec2659f

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
限速规则：10次/1秒
HTTP GET /api/usdt/v2/market/orderBook?symbol=BTC-SWAP
```

请求参数：

名称  | 类型  | 是否必填  | 说明 |
---------|--------|---------|--------|
symbol | string | 是 | 合约名称，BTC-SWAP |
size   | string | 否 | 深度档位，值有5、10、50、100。默认值10|

返回字段说明：

名称   | 类型  | 说明 |
---------|---------|---------|
asks   | array | 卖方深度，[档位价格，数量，该深度由几笔订单组成]|
bids   | array | 买方深度，[档位价格，数量，该深度由几笔订单组成]|
symbol      | string |  合约名称
timestamp      | string |  时间戳，国际时间



```
Request:
Url: http://域名/api/usdt/v2/market/orderBook?symbol=BTC-SWAP&size=10
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
### 公共接口-获取全部ticker信息

```
获取平台全部合约的最新成交价、买一价、卖一价和24交易量
限速规则：10次/1秒
HTTP GET /api/usdt/v2/market/tickers
```
请求参数：无

返回字段说明：

名称   | 类型  | 说明
---------|---------|---------|
symbol         | string | 合约名称，如BTC-SWAP
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
timestamp      | string |  时间戳，UTC国际时间


```
Request:
Url: http://域名/api/usdt/v2/market/tickers
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
      "turnover": "9988", 
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
### 公共接口-获取K线数据

```
获取合约K线数据。K线数据最多可获取2000条。
限速规则：10次/1秒
HTTP GET /api/usdt/v2/market/klines
```
请求参数：

名称  | 类型  | 是否必填  | 说明 |
---------|---------|---------|---------|
symbol      | string | 是 | 合约名称，如BTC-SWAP |
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
timestamp   | string | 生成时间
open   | string | 开盘价格
close   | string | 收盘价格
high   | string | 最高价格
low   | string | 最低价格
volume   | string | 成交量（张）
turnover   | string | 交易额
buyVolume   | string | 主买量（张）
buyTurnover   | string | 主卖量（张）


```
Request:
Url: http://域名/api/usdt/v2/market/klines?symbol=BTC-SWAP&resolution=1&startTime=1557425760&endTime=1557425820
Method: GET
Headers: 
	Accept: application/json
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-21T11:16:20.521ZGET/api/usdt/v2/market/klines?symbol=BTC-SWAP&resolution=1&startTime=1557425760&endTime=1557425820

Response:
格式说明:[time,open,high,low,close,volume,turnover,buyVolume,buyTurnover]
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
### 公共接口-查询最新成交信息

```
获取合约的最新成交信息
限速规则：10次/1秒
HTTP GET /api/usdt/v2/market/trades
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---------|---------|---------|---------|
symbol      | string | 是 | 合约名称，如BTC-SWAP
limit  | string | 否 | 返回记录数，默认10，最大100

返回字段说明：

名称   | 类型  | 说明
---------|---------|---------|
price   | string | 成交价格
side   | string | 成交方向，s=主卖，b=主买
volume   | string | 成交量（张）
timestamp   | string | 国际时间 成交时间


```
Request:
Url: http://域名/api/usdt/v2/market/trades?symbol=BTC-SWAP&limit=1
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

### 公共接口-获取当前合约最新资金费率

```
获取当前合约最新资金费率
限速规则：10次/1秒
HTTP GET /api/usdt/v2/market/fundingRate
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---------|---------|---------|---------|
symbol      | string | 是 | 合约名称，如BTC-SWAP

返回字段说明：无


```
Request:
Url: http://域名/api/usdt/v2/market/fundingRate?symbol=BTC-SWAP
Method: GET
Headers: 
	Accept: application/json
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-05-21T11:19:52.303ZGET/api/usdt/v2/market/fundingRate?symbol=BTC-SWAP

Response:
{
  "code": 200, 
  "data": "0.00375"
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
10010 | api 校验失败
10011 |无效的sign
10012 |api 校验失败
11000 |{0}必填参数为空
11001 |{0}参数值不合法
