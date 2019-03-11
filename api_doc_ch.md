# BitMesh-API文档

<a name="68b2581b"></a>
# API简介
**欢迎使用BitMesh开发者文档，你可以使用API获得当前市场行情数据，进行交易，并且管理你的账户。**<br />如果你在使用过程中遇到任何问题，请添加[Telegram讨论群](https://t.me/bitmeshchat)，或微信客服号([bitmesh2]())，我们将尽力为您解答。

<a name="0cb03a07"></a>
# 做市商项目
欢迎有优秀 Maker 策略的用户参与BitMesh做市商项目，需要了解更多信息，请发送邮件至：[business@bitmesh.com]()，或者 [提交工单](https://help.bitmesh.com/hc/zh-cn/requests/new)

<a name="ec97ee8d"></a>
# 接入说明
BitMesh目前提供 **rest** 与 **websocket** 两种接入方式；<br />BitMesh API目前提供“Public Https API”与“Private Https API”两种类型；<br />Public API 可以直接访问，Private API 需要使用 API KEY并通过 SHA256进行签名;<br />BitMesh所有的 Websocket请求都使用 [msgpack](https://msgpack.org/)进行编码；<br />BitMesh所有的 Rest API 都可以通过 Websocket方式调用，同时Websocket API还能订阅数据，当产生响应的数据时服务端会主动推送（参照下文 [Websocket API](https://www.yuque.com/thirteen/kb/vgrxoe#adb18e42)）。

<a name="d8ae4cd2"></a>
## API KEY
如果你还没有BitMesh账户，请先进行 [注册](https://bitmesh.com/?#/register)<br />注册BitMesh账户后，你可以到个人中心 [添加API](https://bitmesh.com/#/user)（目前每个BitMesh账户仅允许添加1个API，默认开启查询与交易权限），通过账户验证后便可生成API KEY（在任何时候请匆泄露您的API KEY，如果你怀疑已经泄露，请立刻删除并生成新的密钥）

<a name="044744e3"></a>
## API 请求格式
所有 API请求统一 URL路径为 https://api.bitmesh.com，参数格式如下：

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| API | string | 请求的API名称 |
| params | json object | API需要的参数(可选) |
| t | string | 当前时间戳，通过Date.now()获取 |
| accessKey | string | BitMesh提供的API KEY |
| signature | string | sha256签名，为防止数据在传输过程中被截取 |

注：只有请求 Private  API 时才需要后面三个参数


<a name="b1762176"></a>
## API 返回格式
所有 API 返回数据都是JSON格式。<br />在JSON最上层有几个表示请求状态和属性的字段："success", 和 "data"；<br />实际接口返回的内容在 "data" 字段中，"success" 表示请求结果，true 表示请求成功，false 表示请求失败。

**请求错误**<br />当请求遇到错误时，错误信息将通过 "data" 中的 "mssage" 返回，示例如下

```javascript
{
	"success": true,
	"data": {
		"success": false,
		"message": "market is required"
	}
}
```
[
      
    ]()
  
    [
      
    ]()

<a name="381df2ec"></a>
## Public  API
Public  API 允许你对公共市场数据进行访问。

| 接口 | 描述 |
| --- | --- |
| market.list   | 获取所有BitMesh支持的交易对列表 |
| market.ticker  | 获取所有交易对的最新 ticker |
| market.depth    | 查询市场深度 |
| market.statistics | 查询交易对详情 |
| market.tradeHistory | 查询市场最新成交记录 |

Public  API 请求参数及返回数据详情，请参照下文具体的 [Public ](https://www.yuque.com/thirteen/kb/vgrxoe#bf822c64)[Rest](https://www.yuque.com/thirteen/kb/vgrxoe#bf822c64)[  API ](https://www.yuque.com/thirteen/kb/vgrxoe#bf822c64)示例

<a name="f590773c"></a>
## Private  API
Private  API 允许对你的私人帐户进行查询和交易。

| 接口 | 描述 |
| --- | --- |
| account.balance  | 查询账户余额 |
| order.put  | 限价下单 |
| order.batchPut | 批量下单 |
| order.cancel | 取消单个订单 |
| order.batchCancel  | 取消多个订单 |
| order.cancelAll | 取消所有订单 |
| order.pending | 查询委托中订单 |
| order.latestDeal | 查询已成交订单 |
| order.detail  | 查询订单详情 |

Private API 请求参数及返回数据详情，请参照下文具体的 [Private Rest API](https://www.yuque.com/thirteen/kb/vgrxoe#7e1dbffe) 示例

**注意：**<br />为防止数据在网络传输过程中被截取篡改，所以**无论是 Rest 方式，还是 Websocket 方式, 请求 Private  API 时都需要同时传递  "t"  "acce****ssKey"  "signature" 三个参数。**

<a name="5cdc4563"></a>
## Rest 调用步骤

1.首先，请以标准格式规范化params参数（因为不同的格式将导致不同的HMAC签名），如果请求的 api不需要params，则跳过

```javascript
paramsStr = Object.keys(params)
  .sort()
  .map(key => { return `${key}=${params[key]}`;})
  .join('&');
```

2.按照以下方法生成 signature。如果是请求 Public API，则不需要 signature

```javascript
const Signature = sha256(accessKey + api + paramsStr + timestamp + accessSecret).toString('hex').toLowerCase();
```

3.请求 API<br />请参照下文具体的 Rest API 调用示例

<a name="6dba2dfc"></a>
## Websocket 调用步骤
BitMesh所有的 rest API 都可以通过websocket方式调用，同时websocket接口还能订阅数据（订阅功能请参照示例）

如果是请求 Public API，则可直接请求（参照下文 [Websocket API](https://www.yuque.com/thirteen/kb/vgrxoe#adb18e42) 示例）<br />如果是请求 Private API，请按以下步骤

**1.**首先，请按照以下方法生成 signature。

```javascript
const Signature = sha256([accessKey, accessSecret, timestamp].join('\n'));
```

**2.**通过以下方法调用 auth API。

```javascript
ws.send(msgpack.encode([
  Date.now(),
  'auth',
  {
    accessKey, timestamp, signature
  }
]));
```

**3.**调用 Private  API<br />请参照下文具体的 websocket API 调用示例


<a name="bf822c64"></a>
# Public Rest API 示例
<a name="e4b1a609"></a>
## market.list  
获取所有 BitMesh支持的交易对列表

**params参数**<br />无

**响应数据**<br />响应为BitMesh当前支持的交易对数组

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| name | string | 交易对名称 |
| funds | string | 计价币种 |
| stock | string | 基础币种 |

**请求示例**<br />**
```shell
https://api.bitmesh.com/?api=market.list
```

**响应示例**<br />**
```javascript
{
	"success": true,
	"data": [
	{
		"name": "btc_beam",
		"funds": "BTC",
		"stock": "BEAM"
	},
	{
		"name": "btc_grin",
		"funds": "BTC",
		"stock": "GRIN"
	},
 ……
```

<a name="8840de62"></a>
## market.ticker 
获取所有交易对的最新 ticker

**params参数**<br />无

**响应数据**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| ID | json array | 交易对数组 |
| name | string | 交易对名称（计价币种_基础币种） |
| price | string | 最新成交价 |
| volume | string | 24小时成交量（基础币种单位） |
| value | string | 24小时成交量（计价币种单位） |
| max | string | 24小时最高价 |
| min | string | 24小时最低价 |
| change | string | 24小时涨幅百分比 |

**请求示例**<br />**
```shell
https://api.bitmesh.com/?api=market.ticker
```
<br /><br />**响应示例**<br />**
```json
{ success: true,
  data:
   { btc_xlm: { name: 'btc_xlm', change: 0 },
     btc_eth:
      { name: 'btc_eth',
        price: '0.26512027',
        volume: '100',
        value: '2.651202',
        max: '0.26512027',
        min: '0.23323451',
        change: "2.85" },
     btc_ltc: { name: 'btc_ltc', change: 0 }
    ……
```

<a name="market.depth"></a>
## market.depth
查询市场深度

**params参数**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| market | string | 交易对名称 |
| limit | integer | 返回深度的数量 |
| group | integer | 深度的价格聚合位数，默认为8，即精确到 0.00000001 |

**响应数据**<br />响应数据为卖单及买单数组

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| bids | array | 买单数组（价格，数量，累计数量） |
| asks | array | 卖单数组（价格，数量，累计数量） |

**请求示例**<br />**
```shell
https://api.bitmesh.com/?api=market.depth&params={"market":"btc_grin","limit":2,"group":8}
```

**响应示例**<br />**
```json
{
	"success": true,
	"data": {
    "bids": [
      [
        0.00080711,
        85.822384,
        85.822384
      ],
      [
        0.00080709,
        144.078429,
        229.900813
      ]
    ],
    "asks": [
      [
        0.000816,
        14.709785,
        14.709785
      ],
      [
        0.00082,
        100,
        114.709785
      ]
    ]
  }
}
```


<a name="market.statistics"></a>
## market.statistics
查询交易对详情

**params参数**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| market | string | 交易对名称 |

**响应数据**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| stock | string | 基础币种 |
| funds | string | 计价币种 |
| bid | array | 买一价（价格，数量） |
| ask | array | 卖一价（价格，数量） |
| price | string | 最新成交价 |
| max | string | 24小时最高价 |
| min | string | 24小时最低价 |
| volume | string | 24小时成交量（基础币种单位） |
| value | string | 24小时成交量（计价币种单位） |
| change | string | 24小时涨幅百分比 |

**请求示例**<br />**
```shell
https://api.bitmesh.com/?api=market.statistics&params={"market":"btc_grin"}
```

**响应示例**<br />**
```json
{
  "success": true,
  "data": {
    "stock": "GRIN",
    "funds": "BTC",
    "bid": [
      0.00080711,
      85.822384
    ],
    "ask": [
      0.000816,
      14.709785
    ],
    "price": "0.00080711",
    "max": "0.0009297",
    "min": "0.00080711",
    "volume": "2035.839466",
    "value": "1.71702187",
    "change": "-1.09"
  }
}
```


<a name="market.tradeHistory"></a>
## market.tradeHistory
查询市场最近50条成交记录

**params参数**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| market | string | 交易对名称 |

**响应数据**<br />以数组形式返回最近成交的50条记录，依次为：**成交时间戳(毫秒)，成交方向，成交价格，成交数量**<br />**成交方向**：1 表示Taker为Seller，2 表示Taker为Buyer

**请求示例**<br />**
```shell
https://api.bitmesh.com/?api=market.tradeHistory&params={"market":"btc_grin"}
```

**响应示例**<br />**
```json
{
  "success": true,
  "data": [
    [
      "1552186342725",
      "1",
      "0.00080711",
      "3.18682"
    ],
    [
      "1552186842725",
      "1",
      "0.00080711",
      "4.467509"
    ],
  ……
```


<a name="7e1dbffe"></a>
# Private Rest  API 示例
所有 Private API 请求都需要同时传递 "t" "accessKey" "signature"三个参数。具体使用请查看 [#完整示例](https://www.yuque.com/thirteen/kb/vgrxoe#39c58f5e)

<a name="c904cfdf"></a>
## account.balance 
查询账户余额

**params参数**<br />无

**响应数据**<br />响应为资产信息数组

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| name | string | 币种名称 |
| amount | string | 总额 |
| frozen | string | 下单冻结数额 |
| available | string | 可用余额 |

**响应示例**<br />**
```json
{
  success: true,
  data:[
    { name: 'BTC',
       amount: '99.99996023',
       frozen: '0.04813444',
       available: '99.95182579' },
     { name: 'ETH',
       amount: '9.99985',
       frozen: 0,
       available: '9.99985' },
     { name: 'USDC', amount: 0, frozen: 0, available: 0 }
    ……
  ] 
}
```

<a name="5536c9dd"></a>
## order.put 
限价下单

**params参数**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| market | string | 交易对名称 |
| price | double | 价格 |
| amount | double | 数量 |
| side | int | 订单方向（1为购买，2为出售） |

**返回数据**<br />如果委托成功，则返回订单ID，如果失败则返回错误message

**返回示例**<br />**
```json
{ success: true, data: '1552117835346623381206561830' }
```


<a name="1e5bf990"></a>
## order.batchPut 
批量下单

**params参数**<br />参数与order.put一样，区别是传入数组

**返回数据**<br />如果委托成功，则返回订单ID数组，如果失败则返回错误message

**返回示例**<br />**
```json
{ 
  success: true, 
  data: [
    '1552117835346623381206561830','1552117835346623381206561831',……
  ] 
}
```

<a name="order.cancel"></a>
## order.cancel
取消一个订单

**params参数**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| market | string | 交易对名称 |
| id | strin | 订单id |

**返回数据**<br />如果取消成功，则data返回为true，如果失败则返回错误message

**返回示例**<br />**
```json
{ success: true, data: true }
```

<a name="4a080cfe"></a>
## order.batchCancel 
取消多个订单

**params参数**<br />参数与order.cancel一样，区别是传入数组

**返回数据**<br />如果取消成功，则data返回为true，如果失败则返回错误message

**返回示例**<br />**
```json
{ 
  success: true, 
  data: [
    true, true, true ...
  ] }
```


<a name="order.cancelAll"></a>
## order.cancelAll
取消全部订单

**params参数**<br />无

**返回数据**<br />如果取消成功，则data返回为true，如果失败则返回错误message

**返回示例**<br />**
```json
{ success: true, data: true }
```

<a name="order.pending"></a>
## order.pending
查询委托中订单

**params参数**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| market | string | 交易对名称 |

**返回数据**<br />如果存在委托中订单，则返回订单数组

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| id | string | 订单号 |
| createtime | int64 | 订单创建时间 |
| price | string | 价格 |
| userid | string | 用户UID |
| side | string | 订单方向（1为购买，2为出售） |
| leftamount | string | 剩余数量（基础币种单位） |
| leftfunds | string | 剩余数量（计价币种单位） |
| market | string | 交易对 |
| type | string | 订单类型（1为限价，2为市价） |
| filledamount | string | 已成交数量（基础币种单位） |
| filledfunds | string | 已成交数量（计价币种单位） |

**返回示例**<br />**
```json
{ success: true,
  data:
   [ { id: '1552117835346623381206561830',
       createtime: 1552117835346,
       price: '0.021375',
       userid: '3pTwHQUJSibYm9GXjnrcFE',
       side: '1',
       leftamount: '10.2',
       leftfunds: '0.218025',
       market: 'btc_eth',
       type: '1',
       filledamount: 0,
       filledfunds: 0 
     },
     { id: '1552117811659591748048861408',
       createtime: 1552117811659,
       price: '0.021376',
       userid: '3pTwHQUJSibYm9GXjnrcFE',
       side: '1',
       leftamount: '10.2',
       leftfunds: '0.218025',
       market: 'btc_eth',
       type: '1',
       filledamount: 0,
       filledfunds: 0 
     } 
   ] 
}
```

<a name="order.latestDeal"></a>
## order.latestDeal
查询已成交订单

**params参数**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| market | string | 交易对名称 |

**返回数据**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| id | string | 订单号 |
| createtime | int64 | 订单创建时间 |
| side | string | 订单方向（1为购买，2为出售） |
| leftamount | string | 剩余数量（基础币种单位） |
| leftfunds | string | 剩余数量（计价币种单位） |
| market | string | 交易对 |
| type | string | 订单类型（1为限价，2为市价） |
| filledamount | string | 已成交数量（基础币种单位） |
| filledfunds | string | 已成交数量（计价币种单位） |

**返回示例**<br />**
```json
{ success: true,
  data:
   [ { id: '1552119988345858152416398117',
       createtime: 1552119988345,
       side: '2',
       type: '1',
       market: 'btc_eth',
       filledamount: '0.1',
       leftamount: 0,
       filledfunds: '0.02651202',
       leftfunds: '0' 
     } 
   ] 
}
```

<a name="b93551ff"></a>
## order.detail 
查询订单详情

**params参数**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| market | string | 交易对名称 |
| id | strin | 订单id |

**返回数据**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| id | tring | 订单ID |
| createtime | int64 | 订单创建时间 |
| price | string | 订单价格 |
| leftamount | string | 剩余数量（基础币种单位） |
| filledamount | string | 已成交数量（基础币种单位） |
| leftfunds | string | 剩余数量（计价币种单位） |
| filledfunds | string | 已成交数量（计价币种单位） |
| side | string | 订单方向（1为购买，2为出售） |

**返回示例**<br />**
```json
{
  success: true,
  data:
   { id: '1552118841532736701258292074',
     createtime: 1552118841532,
     price: '0.26512027',
     leftamount: '0.281557',
     filledamount: 0,
     leftfunds: '0.07464646',
     filledfunds: 0,
     side: '1' 
   } 
}
```


<a name="adb18e42"></a>
# websocket API
BitMesh所有的 rest API 都可以通过websocket方式调用，同时websocket接口还能订阅数据，当产生响应的数据时服务端会主动推送。Websocket 获取数据的方式分为请求和订阅两种。

<a name="75b16081"></a>
## 请求
客户端和服务端的数据传输采用msgpack编码格式，调用方式如下。
```json
ws.send(msgpack.encode([
  id,
  api,
  params
]));
```

**参数解释**

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| id | int | 用于标识本次请求的id，和服务端响应一一对应 |
| API | string | 要调用的API名称，例如maket.depth |
| params | json | 调用该API需要的参数，可选 |

**以调用某个市场深度的API为例（Public API）**

发出调用请求

```json
ws.send(msgpack.encode([
  123,
  'market.depth',
  {
    market: 'btc_eth',
    limit: 1,
    group: 8,
  }
]))
```

服务端数据响应

```json
[
  123,
  {
    "success": true,
    "data": {
      "bids": [
        [
          0.26512027,
          0.181557,
          0.181557
        ]
      ]
    }
  }
]
```
其中数组第一个元素123表示本次请求的id，第二个元素为具体的数据。

Private API调用方式请按照 [Websocket调用步骤]()，首先生成 signature 并调用 auth.api，后续步骤是相同的。

<a name="02daf71f"></a>
## 订阅
目前websocket已默认集成用户成交数据的订阅，当用户的订单被成交时会主动收到服务端推送的消息，消息格式如下。

```json
[
  "deal",
  {
    "m": "btc_grin", // 成交的交易对
    "s": "2", // 成交方向 1表示taker是卖单，2表示taker是买单
    "d": "1787947298347827349243", //本次成交的id
    "o": "13123132189812938", // 用户挂单的订单id
    "p": "0.00021321", // 成交价格
    "a": "0.1", // 成交数量
    "t": "1552038284130843600" // 成交时间 
  }
]
```



<a name="39c58f5e"></a>
# Rest Private API 请求完整示例

<a name="30730294"></a>
## 1.获取API KEY
注册BitMesh账户后可在个人中心添加API，生成 accessKey 和 accessSecret<br />示例（仅做示例使用）：

```javascript
accessKey = 'ErKy6TT813WE1czJxFrhimMgMM5pmidC787811';
accessSecret = '12ff34e0dce1f74d78bf8837f5551e49c24e8302f51b8d24e658b902a20903f2e1';
```

<a name="859f3b91"></a>
## 2.准备所有参数
以请求order.put接口为例

```javascript
api:'order.put',
params: JSON.stringify({
  market:'btc_eth', //市场
  price: '0.1', //价格
  amount:'0.2', //数量
  })
t: 1551323157987, // Date.now()
accessKey: 'ErKy6TT813WE1czJxFrhimMgMM5pmidC787811',
```

<a name="a019ab4d"></a>
## 3.规范化params参数
使用以下方法规范化参数

```javascript
paramsStr = Object.keys(params)
  .sort()
  .map(key => { return `${key}=${params[key]}`;})
  .join('&');
```
该方法将得到结果： 'amount=0.2&market=btc_eth&price=0.1'

<a name="02397ac5"></a>
## 4.生成 signature
使用以下方法生成signature

```javascript
const Signature = sha256(accessKey + api + paramsStr + timestamp + accessSecret).toString('hex').toLowerCase();
```

该方法将得到

```javascript
sha256('ErKy6TT813WE1czJxFrhimMgMM5pmidC787811' + 'order.put' + 'amount=0.2&market=btc_eth&price=0.1' + '1551323157987' + '12ff34e0dce1f74d78bf8837f5551e49c24e8302f51b8d24e658b902a20903f2e1').toString('hex').toLowerCase();

5a848575009e99d71d25c259b99f7ba3894dd94e8f3a60bc363fed142bce2dfa
```

<a name="93d33bf6"></a>
## 5.提交请求

```javascript
http.request({
  url:'https://api.bitmesh.com',
  data: {
    api:'order.put',
    params: JSON.stringify({
      market:'btc_eth',
      price: '0.1',
      amount:'0.2',
    }),
    t: 1551323157987, // Date.now()
    accessKey: 'ErKy6TT813WE1czJxFrhimMgMM5pmidC787811',
    signature：'5a848575009e99d71d25c259b99f7ba3894dd94e8f3a60bc363fed142bce2dfa', 
  }
})
```

