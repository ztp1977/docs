# Rest API共通情報

本APIを使って、取引プログラムの実装ができます。

APIを使えば以下の機能が使用できます。

- 下記のマーケット情報を取得可能
  + チャート情報
  + デプス (奥行き)
  + リアルタイム約定情報
  + 24時間チャートのモニタリング
- 資産情報の取得
- 注文する、及び注文取り消し操作
- 注文履歴の取得

## セキュリティ認証
### 1. API KEYの申請

* API-KEYの作成と変更について、 [アカウント > API > 秘密鍵の作成] にて行ってください。

keyの種類 | 説明 |
------- | ----- |
AccessKey | アクセスキー |
SecretKey | 署名時に必要な秘密鍵 |

<aside class="info">
秘密キーは申請時のみ表示されますので、ご注意ください。
</aside>

<aside class="warning">
これらの二つのキーはアカウントのセキュリティに関する重要なものなので絶対に他人に開示しないでください。
</aside>

### 2. リクエストの共通仕様

セキュリティ上の理由で、マーケットAPI以外のAPIリクエストには全て署名が必要となります。これから共通の仕様を説明します。


> リクエストの例

```shell
curl -X GET \  
      https://api-cloud.huobi.co.jp/v1/order/orders? \ 
      AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx \ 
      &SignatureMethod=HmacSHA256 \ 
      &SignatureVersion=2 \ 
      &Timestamp=2017-05-11T15%3A19%3A30 \ 
      &order-id=1234567890 \ 
      &Signature=$(some calculated value)
```

### HOST

`https://api-cloud.huobi.co.jp`

### 共通仕様

Header | 説明 |
--------- | -----------
User Agent |  'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.71 Safari/537.36' |
POST Request | Content-Type: application/json | 
GET Request |  Content-Type: application/x-www-form-urlencoded |
Response Format | json |

<aside class="success">
頻度 - 100回/100s
</aside> 

### 認証用共通パラメータ

パラメータ | 詳細説明
---------- | -------
AccessKeyId | アクセスキー
SignatureMethod | 署名の演算時に用いるハッシュベースプロトコル、ここではHmacSHA256を指定します
SignatureVersion | 署名プロトコルのバージョン、ここでは2を指定します
Timestamp | リクエスト時のタイムスタンプ(UTC 時間) 。<br>タイムスタンプをクエリに含めることで、第三者がリクエストを傍受するのを防ぐことができます。例：2017-05-11T16:22:06。タイムスタンプはUTC 時間であることに注意してください。 
Signature | 署名に基づいて計算された値。署名が有効で改ざんされていないことを保証するために使用されます。<br>**必須**、オプションのパラメーター各メソッドには、API呼び出しを定義するための一連の必須、オプションのパラメーターがあります。これらのパラメータとその意味は、各メソッドの説明で確認できます。
 
<aside class="notice">
GETリクエストの場合、各メソッドのパラメータに署名する必要があリます。
</aside>

<aside class="notice">
POSTリクエストに対して、下記のパラメータのみ署名する必要があります。そのほかのパラメータはRequest Bodyに入れてください<br>
　　・AccessKeyId<br>
　　・SignatureMethod<br>
　　・SignatureVersion<br>
　　・Timestamp<br>
</aside>

## 署名処理
### 署名処理手順

```shell
# 元のリクエスト
https://api-cloud.huobi.co.jp/v1/order/orders?
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx
&order-id=1234567890
&SignatureMethod=HmacSHA256
&SignatureVersion=2
&Timestamp=2017-05-11T15:19:30

# 1. 改行入れる
GET\n

# 2. 改行入れる
api-cloud.huobi.co.jp\n

# 3. 改行入れる
/v1/order/orders\n

# 4. パラメータソートする
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx
SignatureMethod=HmacSHA256
SignatureVersion=2
Timestamp=2017-05-11T15%3A19%3A30
order-id=1234567890

# 5. '&'で連結
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&order-id=1234567890

# 6. 署名用文字列
GET\n
api-cloud.huobi.co.jp\n
/v1/order/orders\n
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&order-id=1234567890

# 7. 署名処理 - サンプルコードに参照
SecretKey: b0xxxxxx-c6xxxxxx-94xxxxxx-dxxxx
Signature: 4F65x5A2bLyMWVQj3Aqp+B4w+ivaA7n5Oi2SuYtCJ9o=

# 8. 署名後の文字列をURLに付加する
https://api-cloud.huobi.co.jp/v1/order/orders?AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&order-id=1234567890&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&Signature=4F65x5A2bLyMWVQj3Aqp%2BB4w%2BivaA7n5Oi2SuYtCJ9o%3D
```

<aside class="success">
HMACを使う為に、標準化しないと署名の結果は変わります
</aside>
 
1. リクエスト方法（GET 或いは POST），続けて改行を追加する

2. 小文字のアクセスアドレスに続けて改行を追加する

3. アクセスメソッドへのパス、続けて改行を追加する

4. パラメータ名は、ASCIIコードの順にソートされます（UTF-8エンコーディングとURIエンコーディングを使用すると、16進文字は大文字にする必要があり、 '：'は '％3A'、スペースは'％20'といった具合にエンコードされます）
例えば、エンコード後の要求パラメータの元の順序は次のとおりです

5. 上記の順序で、各パラメーターは文字 '＆'を使用して接続されます

6. 計算のために署名される最後の文字列は次のとおりです

7. 秘密キー(SecretKey)と変換後のリクエスト文字列を使って、署名処理を行い、その結果はBase64エンコードします。<br>上記の値をパラメータSignatureの値としてAPIリクエストに追加します。 このパラメータをリクエストに追加する場合、その値はURIエンコードされている必要があります

8. 最終的に、サーバーに送信するAPIリクエストは次のようになります

# マスター情報関連

## 取引ペア精度 

```shell
curl -X GET \
      "https://api-cloud.huobi.co.jp/v1/common/symbols"
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "data": [
    {
      "base-currency": "ltc",
      "quote-currency": "jpy",
      "price-precision": 4,
      "amount-precision": 4,
      "symbol-partition": "default",
      "symbol": "ltcjpy"
    },
    {
      "base-currency": "xrp",
      "quote-currency": "jpy",
      "price-precision": 4,
      "amount-precision": 2,
      "symbol-partition": "default",
      "symbol": "xrpjpy"
    }
  ]
}
```

このエンドポイントは各取引ペアの精度を返します。

### HTTP Request

`GET /v1/common/symbols`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
 |  | 


### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| status | true | string | リクエスト处理结果,  例えば: "ok","error" 
| data   | true | list | 精度データ 

## 対応取引通貨ペア

```shell
curl -X GET \
     "https://api-cloud.huobi.co.jp//v1/common/currencys"
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "data": [
    "jpy",
    "btc",
    "xrp",
    "eth",
    "ltc",
    "bch",
    "mona"
  ]
}
```


このエンドポイントは各取引通貨ペアを返します。

### HTTP Request

`GET /market/detail/currencys`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
 |  | 


### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| status | true | string | リクエスト处理结果,  例えば: "ok","error" 
| data   | true | list | 精度データ 

## システム時間を調べる

```shell
curl -X GET \
      "https://api-cloud.huobi.co.jp//v1/common/timestamp"
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "data": 1555667124908
}
```


このエンドポイントはシステムのタイムスタンプ(ミリ秒)を返します。

### HTTP Request

`GET /market/detail/timestamp`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
 |  | 


### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| status | true | string | リクエスト处理结果,  例えば: "ok","error" 
| data   | true | int | システムのUNIX Timestamp (ミリ秒) 

# マーケット関連

## KLineデータの取得

```shell
curl -X GET \
      "https://api-cloud.huobi.co.jp/market/history/kline?period=1day&size=200&symbol=btcjpy"
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "ch": "market.btcjpy.kline.1day",
  "ts": 1499223904680,
  "data": [
    {
      "id": 1499184000,
      "amount": 37593.0266,
      "count": 0,
      "open": 1935.2000,
      "close": 1879.0000,
      "low": 1856.0000,
      "high": 1940.0000,
      "vol": 71031537.97866500
    }
  ]
}
```

```shell
curl -X GET \
      https://api-cloud.huobi.co.jp/market/history/kline?period=not-exist&size=200&symbol=ethjpy
```

> 期間エラーのとき、下記のようなレスポンスを返します。

```json
{
  "ts": 1490758171271,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid period"
}
```

```shell
GET /market/history/kline?period=1day&size=not-exist&symbol=ethjpy
```

> サイズエラーのとき、下記のようなレスポンスを返します。

```json
{
  "ts": 1490758221221,
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "invalid size, valid range: [1,2000]"
}
```

```shell
GET /market/history/kline?period=1day&size=200&symbol=not-exist
```

> 取引通貨ペアが存在しない場合は、下記のようなレスポンスを返します。

```json
{
  "ts": 1490758171271,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid symbol"
}
```

### URI

`GET /market/history/kline`

### Request Params

| Params | Required | Type | Description  |
| ------------ | ----- | ------ | --- | 
| symbol | Y | string | 取引ペア - btceth |
| period | Y | string | チャートタイプ |
| size | N | int | サイズ - default=150 max=2000 |

### Response Data

| Field | Type | Description  |
| ------ | ------ | ----------- |
| status | string | リクエスト処理結果 ["ok" , "error"] |
| ts     | number | 生成応答時間点，单位：ミリ秒  |
| tick   | object | KLine データ |
| ch     | string | チャンネル <br> Example: market.$symbol.kline.$period |

## 板データの取得 (Ticker)

```shell
curl -X GET "https://api-cloud.huobi.co.jp/market/detail/merged?symbol=ethjpy"
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "ch": "market.ethjpy.detail.merged",
  "ts": 1499225276950,
  "tick": {
    "id": 1499225271,
    "ts": 1499225271000,
    "close": 1885.0000,
    "open": 1960.0000,
    "high": 1985.0000,
    "low": 1856.0000,
    "amount": 81486.2926,
    "count": 42122,
    "vol": 157052744.85708200, 
    "ask": [ 
      1885.0000, 
      21.8804 
    ],
    "bid": [ 
      1884.0000,
      1.6702 
    ]
  }
}
```

> 取引ペアがないとき、下記のレスポンスになります

```json

{
  "ts": 1490758171271,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid symbol"
}
```

このエンドポイントは集約されたチャート情報を返します。

### HTTP Request

`GET /market/detail/merged`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
symbol | true | トレードペア, 例えば btcjpy


### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| status | true | string | リクエスト处理结果,  例えば: "ok","error" 
| ts     | true | number | 生成応答時間点，单位：ミリ秒 
| tick   | true | object | チャートデータ 
| ch     | true | string | データの所属 channel，例えば： market.btcjpy.detail.merged 

## すべての取引ペアの取引相場

```shell
curl -X GET "https://api-cloud.huobi.co.jp/market/tickers"
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{  
    "status":"ok",
    "ts":1510885463001,
    "data":[  
        {  
            "open":0.044297,      // チャート（日） 始値
            "close":0.042178,     // チャート（日） 終値
            "low":0.040110,       // チャート（日） 最安値
            "high":0.045255,      // チャート（日） 最高値
            "amount":12880.8510,  // 24時間の約定量
            "count":12838,        // 24時間の約定数数
            "vol":563.0388715740, // 24時間の取引高
            "symbol":"ethbtc"     // 通貨ペア
        },
        {  
            "open":0.008545,
            "close":0.008656,
            "low":0.008088,
            "high":0.009388,
            "amount":88056.1860,
            "count":16077,
            "vol":771.7975953754,
            "symbol":"ltcbtc"
        }
    ]
}
```

このエンドポイントはすべての取引ペアの取引相場を取得できます。

### HTTP Request

`GET /market/tickers`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
 |  | 


### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| status | true | string | リクエスト处理结果,  例えば: "ok","error" 
| ts     | true | number | 生成応答時間点，单位：ミリ秒 
| data   | true | object | 取引相場のリスト

<aside class="success">
注意：通貨ペアがまだ取引を生成していない場合、返されたデータは内部にあります `open` `close` `high` `low` `amount` `count` `vol` の値は全て `null`
</aside>

## マーケットデプス (Depth)

```shell
curl -X GET "https://api-cloud.huobi.co.jp/market/depth?symbol=btcjpy&type=step1"
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "ch": "market.btcjpy.depth.step1",
  "ts": 1489472598812,
  "tick": {
    "id": 1489464585407,
    "ts": 1489464585407,
    "bids": [
      [7964, 0.0678], // [価格, 注文数量]
      [7963, 0.9162],
      [7961, 0.1],
      [7960, 12.8898],
      [7958, 1.2],
      [7955, 2.1009],
      [7954, 0.4708],
      [7953, 0.0564],
      [7951, 2.8031],
      [7950, 13.7785],
      [7949, 0.125],
      [7948, 4],
      [7942, 0.4337],
      [7940, 6.1612],
      [7936, 0.02],
      [7935, 1.3575],
      [7933, 2.002],
      [7932, 1.3449],
      [7930, 10.2974],
      [7929, 3.2226]
    ],
    "asks": [
      [7979, 0.0736],
      [7980, 1.0292],
      [7981, 5.5652],
      [7986, 0.2416],
      [7990, 1.9970],
      [7995, 0.88],
      [7996, 0.0212],
      [8000, 9.2609],
      [8002, 0.02],
      [8008, 1],
      [8010, 0.8735],
      [8011, 2.36],
      [8012, 0.02],
      [8014, 0.1067],
      [8015, 12.9118],
      [8016, 2.5206],
      [8017, 0.0166],
      [8018, 1.3218],
      [8019, 0.01],
      [8020, 13.6584]
    ]
  }
}
```

```shell
GET /market/depth?symbol=btcjpy&type=not-exist
```

> 集約レベルがないとき、下記のレスポンスになります

```json
{
  "ts": 1490759358099,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid type"
}
```

このエンドポイントは集約されたチャート情報を返します。

### HTTP Request

`GET /market/depth`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
symbol | true | 取引ペア, 例えば btcjpy
type | true | 集約レベル, [step0, step1, step2, step3, step4, step5]

<aside class="success">
注意 - step0の場合は注文の集約しません。
</aside>

### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| status | true | string | リクエスト处理结果, ["ok","error"] 
| ts     | true | number | 生成応答時間点，单位：ミリ秒 
| tick   | true | object | チャートデータ 
| ch     | true | string | データの所属 channel，例えば： market.btcjpy.detail.merged 

## 取引詳細データの取得

```shell
curl -X GET "https://api-cloud.huobi.co.jp/market/trade?symbol=ethjpy"
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "ch": "market.btcjpy.trade.detail",
  "ts": 1489473346905,
  "tick": {
    "id": 600848670,
    "ts": 1489464451000,
    "data": [
      {
        "id": 600848670,
        "price": 7962.62,
        "amount": 0.0122,
        "direction": "buy",
        "ts": 1489464451000
      }
    ]
  }
}
```

> 取引ペアがないとき、下記のレスポンスになります

```json
{
  "ts": 1490759506429,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid symbol"
}
```

このエンドポイントは集約されたチャート情報を返します。

### HTTP Request

`GET /market/trade`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
symbol | true | トレードペア, 例えば btcjpy


### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| status | true | string | リクエスト处理结果,  例えば: "ok","error" 
| ts     | true | number | 生成応答時間点，单位：ミリ秒 
| tick   | true | object | 取引データ 
| ch     | true | string | データの所属 channel，例えば： market.btcjpy.detail.merged 

## 取引履歴の取得

```shell
curl -X GET \
      "https://api-cloud.huobi.co.jp/market/history/trade?symbol=ethjpy"
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
    "status": "ok",
    "ch": "market.ethjpy.trade.detail",
    "ts": 1502448925216,
    "data": [
        {
            "id": 31459998,
            "ts": 1502448920106,
            "data": [
                {
                    "id": 17592256642623,
                    "amount": 0.04,
                    "price": 1997,
                    "direction": "buy",
                    "ts": 1502448920106
                }
            ]
        }
    ]
}
```

> 取引ペアがないとき、下記のレスポンスになります

```json

{
  "ts": 1490758171271,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid symbol"
}
```

このエンドポイントは集約されたチャート情報を返します。

### HTTP Request

`GET /market/history/trade`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
symbol | true | トレードペア, 例えば btcjpy
size | false | データサイズ, Range: {1, 2000}

### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| status | true | string | リクエスト处理结果,  例えば: "ok","error" 
| ts     | true | number | 生成応答時間点，单位：ミリ秒 
| data   | true | object | 約定記録のリスト 
| ch     | true | string | データの所属 channel，例えば： market.$symbol.trade.detail


# アカウント関連 

## ユーザアカウント 

```shell
curl -X GET "https://api-cloud.huobi.co.jp/v1/account/accounts"
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "data": [
    {
      "id": 100009,
      "type": "spot",
      "state": "working",
      "user-id": 1000
    }
  ]
}
```

このエンドポイントはユーザのアカウント情報を返します。

<aside class="success">
署名認証が必要です。
</aside>

### HTTP Request

`GET /v1/account/accounts`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
 |  | 


### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| id    | true | long | アカウントID
| type  | true | string | アカウントタイプ, spot - スポットアカウント
| state | true | string | アカウントステータス, Range: {"work","lock"} 
| user-id  | true | int | ユーザID

## 残高照合

```shell
curl -X GET "https://api-cloud.huobi.co.jp/v1/account/accounts/{account-id}/balance"
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "data": {
    "id": 100009,
    "type": "spot",
    "state": "working",
    "list": [
      {
        "currency": "jpy",
        "type": "trade",
        "balance": "500009195917.4362872650"
      },
      {
        "currency": "jpy",
        "type": "frozen",
        "balance": "328048.1199920000"
      },
      {
        "currency": "etc",
        "type": "trade",
        "balance": "499999894616.1302471000"
      },
      {
        "currency": "etc",
        "type": "frozen",
        "balance": "9786.6783000000"
      },
      {
        "currency": "eth",
        "type": "trade",
        "balance": "499999894616.1302471000"
      },
      {
        "currency": "eth",
        "type": "frozen",
        "balance": "9786.6783000000"
      }
    ],
    "user-id": 1000
  }
}
```

このエンドポイントはユーザの残高情報を返します。

<aside class="success">
署名認証が必要です。
</aside>

### HTTP Request

`GET /v1/account/accounts/{account-id}/balance`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
　account-id  | true | アカウントID


### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| id    | true | long | アカウントID
| state | true | string | アカウントステータス, Range: {"work","lock"} 
| type  | true | string | アカウントタイプ, spot - スポットアカウント
| list  | true | list | 残高情報

#### Listのデータ構造

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| balance | true | long | 残高
| currency | true | string | 仮想通貨
| type  | true | string | タイプ, trade: トレード残高，frozen: 凍結残高

# 取引関連

## 注文実行

```shell
curl -X POST \
     -H "Content-Type: application/json" \
     "https://api-cloud.huobi.co.jp/v1/order/orders/place" \
     -d \
{
   "account-id": "100009",
   "amount": "10.1",
   "price": "100.1",
   "source": "api",
   "symbol": "ethjpy",
   "type": "buy-limit"
}
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "data": "59378"
}
```


このエンドポイントは注文を出すAPIです。

<aside class="success">
署名認証が必要です。
</aside>

### HTTP Request

`GET /v1/order/orders/place`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
| account-id | true  | アカウント ID，accountsを使用して獲得する方法。仮想通貨ペアのトレードを使用‘spot’アカウントのaccountid 
| amount     | true  | 指値の表示注文数量，成り行きで買う時は幾らで買うかを表示，成り行きで売る時に何枚売るかを表示 
| price      | false | 注文価格，成り行き注文はこのパラメーターを通さない  
| source     | false | 注文のソース    
| symbol     | true  | 取引通貨ペア 
| type       | true  | 注文タイプ <br> buy-market：成り行き買い <br> sell-market：成り行き売り<br>buy-limit：指値買い<br>sell-limit：指値売り<br>buy-ioc：IOC買い注文<br>sell-ioc：IOC売り注文<br>buy-limit-maker<br>sell-limit-maker


<aside class="success">
buy-limit-maker<br>

“注文価格”>=“市場最低売り価格”である場合，注文送信後，システムはこの注文を約定拒否します。<br>

“注文価格”<“市場最低売り価格”である場合，送信成功後，この注文はシステムによって受け入れられます。<br>
</aside>

<aside class="success">
sell-limit-maker<br>

“注文価格”<=“市場最高買い入れ価格” である場合，注文送信後，システムはこの注文を受け入れることを拒否します。<br>

“注文価格”>“市場最高買い入れ価格” である場合， 送信成功後，この注文はシステムによって受け入れられます。
</aside>

### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| data  | false | string | 订单ID

## 未約定注文一覧

```shell
curl -X GET \
    "https://api-cloud.huobi.co.jp/v1/order/openOrders" 
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "data": [
    {
      "id": 5454937,
      "symbol": "ethjpy",
      "account-id": 30925,
      "amount": "1.000000000000000000",
      "price": "0.453000000000000000",
      "created-at": 1530604762277,
      "type": "sell-limit",
      "filled-amount": "0.0",
      "filled-cash-amount": "0.0",
      "filled-fees": "0.0",
      "source": "web",
      "state": "submitted"
    }
  ]
}
```

このエンドポイントは未約定注文一覧を返します。

<aside class="success">
署名認証が必要です。
</aside>

### HTTP Request

`GET /v1/order/openOrders`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
| account-id | true  | アカウントID 
| symbol | true  | 取引通貨ペア 
| side | false | 取引方向, Range: {'buy', 'sell'} 
| size | false | 必要な記録数    

### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| id    | true | long | 注文番号 
| symbol| true | string | 取引通貨ペア 
| price | true | string | 注文価格 
| created-at | true | int | 注文時間（ミリ秒 
| type | true | string | 注文タイプ, Range: {buy-market, sell-market, buy-limit, sell-limit, buy-ioc, sell-ioc}
| filled-amount | true | string | 注文時間(ミリ秒)<br>部分取引注文でない場合、このフィールドは0 になります
| filled-cash-amount | true | string | 約定された部品の注文価格（=約定された注文の数量x注文の価格）<br> 部分取引注文でない場合、このフィールドは0になります|
| filled-fees | true | string | 取引の一部における手数料<br>部分取引注文でない場合、このフィールドは0 になります
| source | true | string | 注文ソース, Range: {sys, web, api, app}
| state | true | string | この注文ステータス, Range: {submitted(約定済), partial-filled(部分約定), cancelling(キャンセル)}

## 注文キャンセル

```shell
curl -X POST \
     -H "Content-Type: application/json" \
     "https://api-cloud.huobi.co.jp/v1/order/orders/{order-id}/submitcancel"
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "data": 59378
}
```

本APIはキャンセルリクエストを送信します。実際のキャンセル結果は注文詳細APIにてご確認してください。

<aside class="success">
署名認証が必要です。
</aside>


### HTTP Request

`GET /v1/order/orders/place`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
| order-id | true  | 注文ID 

### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| status | true | string | リクエストの状態
| data  | false | string | 注文ID

<aside class="success">
status=OK、出金依頼が成功したことを示します。<br>
注文が正常に取り消された場合は、注文状況照会インターフェースを呼び出して注文状況を照会してください。
</aside>

## 注文の一括キャンセル

```shell
curl -X POST \
    -H "Content-Type: application/json" \
    "https://api-cloud.huobi.co.jp/v1/order/orders/batchcancel"
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "data": {
    "success": [
      "1",
      "2",
      "3"
    ]
  }
}
```

> 一部失敗する場合、下記のようなレスポンスが返します。

```json
{
  "status": "ok",
  "data": {
    "success": [
      "1",
      "3"
    ],
    "failed": [
      {
        "err-msg": " Invalid record",
        "order-id": "2",
        "err-code": "base-record-invalid"
      }
    ]
  }
}
```

本APIは注文の一括キャンセルを実行します。

<aside class="success">
署名認証が必要です。
</aside>


### HTTP Request

`POST /v1/order/orders/batchcancel`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
| | | 

### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| status | true | string | リクエストの状態
| data  | true | string | キャンセルの結果

## 条件付き注文の一括キャンセル

```shell
curl -X POST \
     -H "Content-Type: application/json" \
     "https://api-cloud.huobi.co.jp/v1/order/orders/batchCancelOpenOrders"
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "data": {
    "success-count": 2,
    "failed-count": 0,
    "next-id": 5454600
  }
}
```

本APIは条件付き注文を一括でキャンセルする。

<aside class="success">
署名認証が必要です。
</aside>


### HTTP Request

`POST /v1/order/orders/batchCancelOpenOrders`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
| account-id | true  | アカウントID 
| symbol | false | 取引通貨ペア, 単一トレードペアの文字列は、デフォルトでは条件が満たされていない全ての注文を返します 
| side | false | 取引方向 , Range: {“buy”,“sell”}， デフォルトでは、条件が満たされていない全ての注文が返されます。
| size | false | 必要な記録数, default: 100, Range: {0,100}

### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| success-count | true | int | キャンセル注文成功数 
| failed-count | true | int | キャンセル注文失敗数
| next-id | true | long | キャンセル基準を満たす次の注文番号

## 注文の詳細の照会

```shell
curl -X GET \
    "https://api-cloud.huobi.co.jp/v1/order/orders/{order-id}" 
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "data": {
    "id": 59378,
    "symbol": "ethjpy",
    "account-id": 100009,
    "amount": "10.1000000000",
    "price": "100.1000000000",
    "created-at": 1494901162595,
    "type": "buy-limit",
    "field-amount": "10.1000000000",
    "field-cash-amount": "1011.0100000000",
    "field-fees": "0.0202000000",
    "finished-at": 1494901400468,
    "user-id": 1000,
    "source": "api",
    "state": "filled",
    "canceled-at": 0,
    "exchange": "xxx",
    "batch": ""
  }
}
```


このエンドポイントは注文の詳細情報を返します。

<aside class="success">
署名認証が必要です。
</aside>

### HTTP Request

`GET /v1/order/orders/{order-id}`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
| order-id | true  | 注文ID 

### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| account-id        | true  | long   | アカウント ID
| amount            | true  | string | 注文数量
| canceled-at       | false | long   | 注文キャンセル時間
| created-at        | true  | long   | 注文作成時間
| field-amount      | true  | string | 約定数量
| field-cash-amount | true  | string | 約定総額
| field-fees        | true  | string | 約定手数料（買い入れは仮想通貨のために，売り出しは法定通貨のために）
| finished-at       | false | long   | 注文の終了時間は約定した時間ではなく、キャンセル済の時も含みます 
| id                | true  | long   | 注文ID
| price             | true  | string | 注文価格
| source            | true  | string | 注文ソース, default: api
| state             | true  | string | 注文ステータス, Range: {"submitting" // 送信中, "submitted"　// 送信済, "partial-filled" // 部分約定, "partial-canceled" //部分約定キャンセル, "filled" //完全約定, canceled キャンセル済み
| symbol            | true  | string | 取引通貨ペア
| type              | true  | string | 注文種類, Range: {"buy-market" // 成り行き買い, "sell-market" //成り行き売り, "buy-limit" //指値買い, "sell-limit" //指値売り, "buy-ioc" //IOC買い注文, "sell-ioc" //IOC売り注文

## 注文の約定詳細の照会


```shell
curl -X GET \
  "https://api-cloud.huobi.co.jp/v1/order/orders" 
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "data": [
    {
      "id": 59378,
      "symbol": "ethjpy",
      "account-id": 100009,
      "amount": "10.1000000000",
      "price": "100.1000000000",
      "created-at": 1494901162595,
      "type": "buy-limit",
      "field-amount": "10.1000000000",
      "field-cash-amount": "1011.0100000000",
      "field-fees": "0.0202000000",
      "finished-at": 1494901400468,
      "user-id": 1000,
      "source": "api",
      "state": "filled",
      "canceled-at": 0,
      "exchange": "xxx",
      "batch": ""
    }
  ]
}
```


このエンドポイントは注文の約定詳細情報を返します。

<aside class="success">
署名認証が必要です。
</aside>

### HTTP Request

`GET /v1/order/orders`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
| symbol     | true  | 取引通貨ペア
| types      | false | オーダーのタイプの組み合わせ照会、複数可、カンマ区切り, Range: {"buy-market" //成り行き買い, "sell-market" //成り行き売り, "buy-limit" //指値買い, "sell-limit" //指値売り, "buy-ioc" //IOC買い注文, "sell-ioc" //IOC売り注文
| start-date | false | 開始日時照会、 日時フォマットyyyy-mm-dd 
| end-date   | false | 終了日時照会、 日時フォマットyyyy-mm-dd 
| states     | true  | オーダー状態、　Range: {"submitted" //提出済み, "partial-filled" //部分約定, "partial-canceled" // 部分約定キャンセル, "filled" // 完全約定, "canceled" //キャンセル済み |
| from       | false | 開始 ID照会
| direct     | false | 方向照会, prev 前，next 後ろ
| size       | false | 大小記録の照会

### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| account-id        | true  | long   | アカウント ID
| amount            | true  | string | 注文数量
| canceled-at       | false | long   | キャンセル申請を受ける時間
| created-at        | true  | long   | 注文作成時間
| field-amount      | true  | string | 約定数量
| field-cash-amount | true  | string | 約定総金額
| field-fees        | true  | string | 約定済み手数料（買いは仮想通貨の為に，売りはお金のために）
| finished-at       | false | long   | 最終的な約定時間
| id                | true  | long   | 注文ID
| price             | true  | string | 注文価格
| source            | true  | string | 注文ソース, default: api
| state             | true  | string | 注文ステータス
| symbol            | true  | string | 取引通貨ペア
| type              | true  | string | 注文タイプ

## 約定履歴の照会


```shell
curl -X GET \
  "https://api-cloud.huobi.co.jp/v1/orders/matchresults" 
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "data": [
    {
      "id": 29555,
      "order-id": 59378,
      "match-id": 59335,
      "symbol": "ethjpy",
      "type": "buy-limit",
      "source": "api",
      "price": "100.1000000000",
      "filled-amount": "0.9845000000",
      "filled-fees": "0.0019690000",
      "created-at": 1494901400487
    }
  ]
}
```


このエンドポイントは約定履歴を返します。

<aside class="success">
署名認証が必要です。
</aside>

### HTTP Request

`GET /v1/orders/matchresults`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
| symbol     | true  | 取引通貨ペア
| types      | false | オーダータイプの組み合わせ照会，複数可, カンマ区切り
| start-date | false | 開始日の照会, 日時フォマットyyyy-mm-dd, Range: [-61日, Now]
| end-date   | false | 終了日の照会, 日時フォマットyyyy-mm-dd
| from       | false | 開始日の照会 ID
| direct     | false | 照会方向 default: next, Range: {"prev", "next"}
| size       | false | 大小記録の照会, Range: [0, 100]

### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| created-at    | true | long   | 約定時間 
| filled-amount | true | string | 約定数量
| filled-fees   | true | string | 約定手数料
| id            | true | long   | 約定注文記録ID
| match-id      | true | long   | マッチングID
| order-id      | true | long   | 注文 ID
| price         | true | string | 約定価格
| source        | true | string | 注文ソース, default: api
| symbol        | true | string | 取引通貨ペア
| type          | true | string | 注文タイプ

# ウォレット関連

<aside class="success">
引出しのみサポート
</aside>

## ステータス一覧

### 仮想通貨出金ステータスの定義：

| ステータス | 説明  |
|--|--|
| submitted | 提出済 |
| reexamine | 審査中 |
| canceled  | キャンセル済 |
| pass    | 承認 |
| reject  | 拒否 |
| pre-transfer | 处理中 |
| wallet-transfer | 送金済 |
| wallet-reject   | ウオレットの拒否 |
| confirmed      | ブロックチェーン上で承認済 |
| confirm-error  | ブロックチェーン上で承認エラー |
| repealed       | キャンセル済 |

### 仮想通貨入金ステータスの定義：

|ステータス|説明|
|--|--|
|unknown| 不明 |
|confirming| 確認中 |
|confirmed| 確認済み |
|safe| 完了 |
|orphan| 独立 |

## 仮想通貨の出金申請

```shell
curl -X POST \
     -H "Content-Type: application/json" \
     "https://api-cloud.huobi.co.jp/v1/dw/withdraw/api/create" \
     -d \
{
  "address": "0xde709f2102306220921060314715629080e2fb77",
  "amount": "0.05",
  "currency": "eth",
  "fee": "0.01"
}
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "data": 700
}
```

本APIは仮想通貨の出金申請を送信する。

<aside class="success">
署名認証が必要です。
</aside>


### HTTP Request

`GET /v1/dw/withdraw/api/create`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
| address | true | string   | 出金アドレス
| amount     | true | string | 出金数量
| currency | true | string | 通貨種別 
| fee     | false | string | 送金手数料
| addr-tag|false | string | 仮想通貨アドレスの共有tag，xrp

### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| data  | false | long | 出金レコードID

##  仮想通貨の出金のキャンセル

```shell
curl -X POST \
     -H "Content-Type: application/json" \
     "https://api-cloud.huobi.co.jp/v1/dw/withdraw-virtual/{withdraw-id}/cancel"
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "status": "ok",
  "data": 700
}
```

本APIは仮想通貨出金のキャンセルリクエストを送信する。

<aside class="success">
署名認証が必要です。
</aside>


### HTTP Request

`GET /v1/dw/withdraw-virtual/{withdraw-id}/cancel`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
| withdraw-id | true  | 出金ID，pathの中に記入

### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
| data  | false | string | 出金レコードID

## 入出金記録

```shell
curl -X GET \
     "https://api-cloud.huobi.co.jp/v1/query/deposit-withdraw" 
```

> 上記のコマンドは、次のような構造のJSONを返します。

```json
{
  "data":
    [
      {
        "id": 1171,
        "type": "deposit",
        "currency": "xrp",
        "tx-hash": "ed03094b84eafbe4bc16e7ef766ee959885ee5bcb265872baaa9c64e1cf86c2b",
        "amount": 7.457467,
        "address": "rae93V8d2mdoUQHwBDBdM4NHCMehRJAsbm",
        "address-tag": "100040",
        "fee": 0,
        "state": "safe",
        "created-at": 1510912472199,
        "updated-at": 1511145876575
      }
    ]
}
```

このエンドポイントは入出金記録を返します。

<aside class="success">
署名認証が必要です。
</aside>

### HTTP Request

`GET /v1/query/deposit-withdraw?currency=xrp&type=deposit&from=5&size=12`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
| currency | true |  銘柄 
| type    | true  |  'deposit' or 'withdraw'
| from    | false |  照会開始 ID 
| size    | false |  大小記録の照会

### Response Data

| Parameter | Required | Type | Description  
| ------ | ---- | ------ | -------  
|   id  |  true  |  long  |   
|   type  |  true  |  long  | タイプ 'deposit' 'withdraw'
|   currency  |  true  |  string  |  銘柄 
| tx-hash | true |string | トレードハッシュ
| amount | true | long | 個数 
| address | true | string | アドレス 
| address-tag | true | string | アドレスラベル 
| fee | true | long | 手数料 
| state | true | string | ステータス ステータスは下の表を参考に 
| created-at | true | long | 開始時間
| updated-at | true | long | 最後に更新した時間

