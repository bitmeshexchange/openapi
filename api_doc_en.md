# BitMesh-API

<a name="Introduction"></a>
# Introduction
Welcome to the BitMesh developer documentation, you can use the API to get current market data, trade, and manage your account.<br />If you encounter any problems during use, please add the [Telegram](https://t.me/bitmeshchat)  group, or the WeChat customer service  (bitmesh2) , we will try our best to answer them for you.

<a name="bf7f33cb"></a>
# Market maker Project
Welcome users with excellent Maker strategy  to participate in the  market Maker project. For more information, please send an email to :business@bitmesh.com, or [Submit a ticker](https://help.bitmesh.com/hc/zh-cn/requests/new)

<a name="dfbcb1f3"></a>
# Access instructions
BitMesh currently provides two access methods: **rest** and **websocket**;<br />BitMesh API currently provides two types: "Public Https API" and "Private Https API;<br />The Public API can be accessed directly. The Private API needs to use the api key and sign it through sha256;<br />All Websocket requests are encoded using [msgpack](https://msgpack.org/);<br />All Rest APIs of BitMesh can be called through Websocket, And the Websocket API can also subscribe to data. When the response data is generated, the server will actively push (refer to the [Websocket API ](https://www.yuque.com/thirteen/kb/vgrxoe#adb18e42)below ).

<a name="d8ae4cd2"></a>
## API KEY
If you do not  have a BitMesh account, please [register ](https://bitmesh.com/?#/register)first<br />After registering a BitMesh account, you can go to the personal center and [add API](https://bitmesh.com/#/user) (currently only 1 API is allowed for each BitMesh account, query and trade permissions are enabled by default ), api key can be generated after the Account Verification (never disclose your api key. If you suspect that it has been leaked, please delete and generate a new KEY immediately)

<a name="d82a2fa7"></a>
## API request format
The uniform URL path for all API requests is **https://api.bitmesh.com** and the parameter format is as follows:

| Field | Type | Description |
| --- | --- | --- |
| API | string | Requested API name |
| params | json object | Parameters required by the API (optional) |
| t | string | Current timestamp, obtained by Date.now() |
| accessKey | string | API KEY provided by BitMesh |
| signature | string | Sha256 signature |

Note: Only the following three parameters are required when requesting the Private API


<a name="a455473f"></a>
## API return format
All API return data is in JSON format.<br />At the top of JSON, there are several fields that represent the Request status and attributes: **"success"**, and **"data "**;<br />The content returned by the actual interface is in the "data" field, "success" means the request result, "true" means the request is successful, and "false" means the request fails.

**Request Error**<br />When the request encounters an error, the error message will be returned via "mssage" in "data", as shown below

```json
{
	"success": true,
	"data": {
		"success": false,
		"message": "market is required"
	}
}
```

<a name="39c9c036"></a>
## Public API
The Public API allows you to access Public market data.

| API | Description |
| --- | --- |
| market.list   | get the list of all  pairs supported by BitMesh |
| market.ticker  | get all ticker of markets |
| market.depth    | query the depth of one market |
| market.statistics | query detail of one market |
| market.tradeHistory | query the latest trade history |

Public API Request Parameters and return data details, please refer to the specific [Public Rest API ](#bf822c64)example below

<a name="23661669"></a>
## Private API
The Private API allows for queries and transactions on your Private account.

| Field | Description |
| --- | --- |
| account.balance  | Check account balance |
| order.put  | make a limit order |
| order.batchPut | make multiple order |
| order.cancel | cancel a order |
| order.batchCancel  | cancel multiple order |
| order.cancelAll | cancel all order |
| order.pending | query pending orders |
| order.latestDeal | query order history |
| order.detail  | query order details |

Private API Request Parameters and return data details, please refer to the specific [Private Rest API ](#7e1dbffe)example below

**Note:**<br />In order to prevent data from being intercepted and tampered  during network transmission, whether it is Rest or Websocket, when requesting a Private API, you need to pass the three parameters of** "t", "acceskey" and "signature"** at the same time.

<a name="25c27b2e"></a>
## Rest call step

1. First, please normalize the params parameter in standard format (because different formats will cause different HMAC signatures), skip if the requested api does not require params

```javascript
paramsStr = Object.keys(params)
  .sort()
  .map(key => { return `${key}=${params[key]}`;})
  .join('&');
```

2. generate signature according to the following methods. Signature is not required if request Public API

```javascript
const Signature = sha256(accessKey + api + paramsStr + timestamp + accessSecret).toString('hex').toLowerCase();
```

3. Request API<br />Please refer to the specific Rest API call example below

<a name="64e4aa7a"></a>
## Websocket call steps
All rest APIs of BitMesh can be called through websocket, And the websocket interface can also subscribe to data (please refer to the example for subscription function)

If you are requesting the Public API, you can request it directly (refer to the [Websocket API ](#adb18e42)example below)<br />If you are requesting a Private API, please follow these steps

1. First of all, please generate signature according to the following methods.

```javascript
const Signature = sha256([accessKey, accessSecret, timestamp].join('\n'));
```

2. Call the auth API by the following method.

```javascript
ws.send(msgpack.encode([
  Date.now(),
  'auth',
  {
    accessKey, timestamp, signature
  }
]));
```

3. Call Private API<br />Please refer to the specific websocket API call example below

<a name="80e24b72"></a>
# Example of Public Rest API
<a name="e4b1a609"></a>
## market.list  
Get the list of tading pairs supported by BitMesh

**params**
None

**Response Data**

Array of trading pairs currently supported on BitMesh

| Field | Type | Description |
| --- | --- | --- |
| name | string | the trading pair to query |
| funds | string | Quote currency |
| stock | string | Base currency |

**Example request**

```shell
https://api.bitmesh.com/?api=market.list
```

**Example response**
```
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
Get the latest ticker for all markets

**params**
None

**Response Data**

| Field | Type | Description |
| --- | --- | --- |
| ID | json array | array of pairs |
| name | string | pairs name (Quote currency_Base currency） |
| price | string | latest price |
| volume | string | 24H volume（Base currency） |
| value | string | 24H value（Quote currency） |
| max | string | 24H max |
| min | string | 24H min |
| change | string | 24H change (%) |

**Example request**
```shell
https://api.bitmesh.com/?api=market.ticker
```
**Example response**
```
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
Market depth of one market

**params**

| Field | Type | Description |
| --- | --- | --- |
| market | string | the trading pair to query |
| limit | integer | The number of market depth to return on each side |
| group | integer | Market depth aggregation level, 8 means 0.00000001 |

**Response Data**<br />Response Data is an array of selling orders and paying bills

| Field | Type | Description |
| --- | --- | --- |
| bids | array | array of bids（price，amount，Cumulative amount） |
| asks | array | aarray of asks（price，amount，Cumulative amount） |

**Example request**
```shell
https://api.bitmesh.com/?api=market.depth&params={"market":"btc_grin","limit":2,"group":8}
```

**Example response**

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
Query transaction details

**params**

| Field | Type | Description |
| --- | --- | --- |
| market | string | the trading pair to query |

**Response Data**

| Field | Type | Description |
| --- | --- | --- |
| stock | string | Base currency |
| funds | string | Quote currency |
| bid | array | First bid (price, amount) |
| ask | array | First ask (price, amount) |
| price | string | Latest Price |
| max | string | 24H Max |
| min | string | 24H Min |
| volume | string | 24H Volume (Base currency) |
| value | string | 24H Value (Quote currency) |
| change | string | 24H change (%) (%) |

**Example request**
```shell
https://api.bitmesh.com/?api=market.statistics&params={"market":"btc_grin"}
```

**Example response**
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
Check the last 50 transaction records in the market<br />**params**

| Field | Type | Description |
| --- | --- | --- |
| market | string | the trading pair to query |

**Response Data**
Return 50 records of the recent transaction in the form of an array, followed by: **transaction time stamp (milliseconds)**, **Transaction direction**, **transaction price**, **transaction amount**<br />Direction of transaction: 1 indicates that Taker is Seller and 2 indicates that Taker is Buyer

**Example request**
```shell
https://api.bitmesh.com/?api=market.tradeHistory&params={"market":"btc_grin"}
```

**Example response**
```
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


<a name="f323d8d8"></a>
# Private Rest API example
All Private API requests need to pass three parameters** "t" "acceskey" **and** "signature"** at the same time. Please see [# Full example for specific use](https://www.yuque.com/thirteen/kb/vgrxoe#39c58f5e)

<a name="c904cfdf"></a>
## account.balance 
Query account balance

**params**
None

**Response Data**
Response to an array of asset

| Field | Type | Description |
| --- | --- | --- |
| name | string | token name |
| amount | string | total |
| frozen | string | in order |
| available | string | avaliable |

**Example response**<br />**
```
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
Make a Limit Order

**params**

| Field | Type | Description |
| --- | --- | --- |
| market | string | the trading pair to query |
| price | double | price |
| amount | double | amount |
| side | int | order type（1 buy，2 sell） |

**Response Data**
Returns the order ID if the delegate is successful and the error message if it fails

**Example response**
```
{ success: true, data: '1552117835346623381206561830' }
```


<a name="1e5bf990"></a>
## order.batchPut 
Place orders in batches

**params**<br />The parameter is the same as order. put. The difference is that needs an Array

**Response Data**
Returns an order ID array if the delegate is successful, and an error message if it fails

**Example response**
```
{ 
  success: true, 
  data: [
    '1552117835346623381206561830','1552117835346623381206561831',……
  ] 
}
```

<a name="order.cancel"></a>
## order.cancel
Cancel an order

**params**

| Field | Type | Description |
| --- | --- | --- |
| market | string | the trading pair to query |
| id | strin | order id |

**Response Data**
Data returns true if the cancellation is successful, and error message if it fails

**Example response**
```
{ success: true, data: true }
```

<a name="4a080cfe"></a>
## order.batchCancel 
Cancel Multiple Orders

**params**
The parameter is the same as order. cancel. The difference is that needs an Array

**Response Data**
Data returns true if the cancellation is successful, and error message if it fails

**Example response**
```
{ 
  success: true, 
  data: [
    true, true, true ...
  ] }
```


<a name="order.cancelAll"></a>
## order.cancelAll
Cancel all orders

**params**
None

**Response Data**
Data returns true if the cancellation is successful, and error message if it fails

**Example response**
```
{ success: true, data: true }
```

<a name="order.pending"></a>
## order.pending
Query pending orders

**params**

| Field | Type | Description |
| --- | --- | --- |
| market | string | the trading pair to query |

**Response Data**
order array if there are pending orders

| Field | Type | Description |
| --- | --- | --- |
| id | string | order id |
| createtime | int64 | order creattime |
| price | string | price |
| userid | string | UID |
| side | string | order side（1 is buy，2 is sell） |
| leftamount | string | left amount（Base currency） |
| leftfunds | string | left amount（Quote currency） |
| market | string | trading pair |
| type | string | order type（1 is limit，2 is market） |
| filledamount | string | filled amount（Base currency） |
| filledfunds | string | filled amount（Quote currency） |

**Example response**

```
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
Query the completed order

**params**

| Field | Type | Description |
| --- | --- | --- |
| market | string | the trading pair to query |

**Response Data**

| Field | Type | Description |
| --- | --- | --- |
| id | string | order id |
| createtime | int64 | order createtime |
| side | string | order side（1 is buy，2 is sell） |
| leftamount | string | left amount（Base currency） |
| leftfunds | string | left amount（Quote currency） |
| market | string | trading pair |
| type | string | order type（1 is limit，2 is market） |
| filledamount | string | filled amount（Base currency） |
| filledfunds | string | filled amount（Quote currency） |

**Example response**
```
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
Query Order Details

**params**

| Field | Type | Description |
| --- | --- | --- |
| market | string | the trading pair to query |
| id | strin | order id |

**Response Data**

| Field | Type | Description |
| --- | --- | --- |
| id | tring | order ID |
| createtime | int64 | order createtime |
| price | string | order price |
| leftamount | string | left amount（Base currency） |
| filledamount | string | filled amount（Base currency） |
| leftfunds | string | left amount（Quote currency） |
| filledfunds | string | filled amount（Quote currency） |
| side | string | order side（1 is buy，2 is sell） |

**Example response**
```
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
All rest APIs of BitMesh can be called through websocket, And the websocket interface can also subscribe to data, and the server will actively push when the response data is generated. Websocket obtains data in two ways: request and subscription.

<a name="Request"></a>
## Request
The data transmission between the client and the server is in the msgpack encoding format. The Calling method is as follows.

```
ws.send(msgpack.encode([
  id,
  api,
  params
]));
```

Interpretation of parameters

| Field | Type | Description |
| --- | --- | --- |
| id | int | The id used to identify this request, and the server response one-to-one correspondence |
| API | string | The name of the API to call, such as maket.depth |
| params | json | The parameters required to call the API (optional) |

**Example：query market depth  (Public API)**

Make  request

```
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

Server Data response

```
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
The first element of the array 123 represents the id of this request, and the second element is the specific data.

Private API call method please follow the[ Websocket call step](https://www.yuque.com/thirteen/kb/vgrxoe), first generate signature and call auth. api, the following steps are the same.

<a name="Subscription"></a>
## Subscription
At present, websocket has integrated the subscription of user transaction data by default. When the user's order is completed, you will actively receive the message pushed by the server. The message format is as follows.

```
[
  "deal",
  {
    "m": "btc_grin", // market
    "s": "2", // side 1 means taker is the seller，2 means taker is the buyer
    "d": "1787947298347827349243", //deal id
    "o": "13123132189812938", // order id
    "p": "0.00021321", // deal price
    "a": "0.1", // deal amount
    "t": "1552038284130843600" // deal time 
  }
]
```



<a name="59dacad9"></a>
# Complete example of Rest Private API request

<a name="4b1a23db"></a>
## 1. Get API KEY
After registering a BitMesh account, you can add an API in the personal center to generate acceskey and accessSecret<br />Example (for example use only ):

```javascript
accessKey = 'ErKy6TT813WE1czJxFrhimMgMM5pmidC787811';
accessSecret = '12ff34e0dce1f74d78bf8837f5551e49c24e8302f51b8d24e658b902a20903f2e1';
```

<a name="2a10f187"></a>
## 2. Prepare all parameters
Example: query api of "order. put"

```javascript
api:'order.put',
params: JSON.stringify({
  market:'btc_eth', 
  price: '0.1', 
  amount:'0.2',
  })
t: 1551323157987, // Date.now()
accessKey: 'ErKy6TT813WE1czJxFrhimMgMM5pmidC787811',
```

<a name="c6363f1c"></a>
## 3. Normalize params
Normalize parameters using the following methods

```javascript
paramsStr = Object.keys(params)
  .sort()
  .map(key => { return `${key}=${params[key]}`;})
  .join('&');
```
The method will get results:  'amount=0.2&market=btc_eth&price=0.1'

<a name="ddd21646"></a>
## 4. generate signature
Generate signature using the following methods

```javascript
const Signature = sha256(accessKey + api + paramsStr + timestamp + accessSecret).toString('hex').toLowerCase();
```

The method will get

```javascript
sha256('ErKy6TT813WE1czJxFrhimMgMM5pmidC787811' + 'order.put' + 'amount=0.2&market=btc_eth&price=0.1' + '1551323157987' + '12ff34e0dce1f74d78bf8837f5551e49c24e8302f51b8d24e658b902a20903f2e1').toString('hex').toLowerCase();

// 5a848575009e99d71d25c259b99f7ba3894dd94e8f3a60bc363fed142bce2dfa//
```

<a name="68596581"></a>
## 5. submit request

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


