# Day 7: HTTP API、SAP ODATAの基本

## ゴール
!!! success "Day 7 Goals"
    - HTTP APIの基礎理解
    - SAP ODataサービスの理解
    - Postman/Rest Clientを使ったODataサービスのテスト実行
    - ODataクエリオプションの使用方法の理解

## HTTP APIの基礎
### HTTPの基本概念
HTTPは、WebブラウザとWebサーバー間でデータを送受信するためのプロトコルです。APIの理解には以下の要素が重要です。

#### HTTPメソッド

各メソッドの用途と特徴：

- **GET**: データの取得（読み取り専用）
- **POST**: 新しいリソースの作成
- **PUT**: 既存リソースの更新・置換
- **PATCH**: 既存リソースの部分更新
- **DELETE**: リソースの削除

#### HTTPステータスコード

レスポンスの結果を表す3桁の数字：

- **2xx成功**: 200 (OK), 201 (Created), 204 (No Content)
- **4xxクライアントエラー**: 400 (Bad Request), 401 (Unauthorized), 404 (Not Found)
- **5xxサーバーエラー**: 500 (Internal Server Error), 503 (Service Unavailable)

#### HTTPヘッダー

リクエストやレスポンスに関するメタデータ：

- **Authorization**: 認証情報（API キー、Bearer トークンなど）
- **Content-Type**: データの形式（application/json, text/xmlなど）
- **Accept**: クライアントが受け入れ可能なレスポンス形式

#### リクエスト例
```http
# 顧客情報を取得するGETリクエスト
GET /api/v1/customers/12345 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Accept: application/json
User-Agent: MyApp/1.0
```

#### レスポンス例
```http
# 成功時のレスポンス
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 156
Date: Mon, 04 Nov 2024 10:30:00 GMT

# レスポンスボディ（JSON形式）
{
  "id": "12345",
  "name": "田中太郎",
  "email": "tanaka@example.com",
  "created_at": "2025-01-15T09:30:00Z"
}
```

### REST APIの設計原則

RESTfulなAPIの特徴：

- **リソースベース**: URLでリソースを識別
- **ステートレス**: 各リクエストは独立
- **HTTPメソッドの適切な使用**: 操作をメソッドで表現
- **統一されたインターフェース**: 予測可能なURL構造

```http
# RESTful API設計例
GET    /api/customers           # 顧客一覧の取得
GET    /api/customers/123       # 特定顧客の取得
POST   /api/customers           # 新規顧客の作成
PUT    /api/customers/123       # 顧客情報の更新
DELETE /api/customers/123       # 顧客の削除
```

### ODataの特徴
- **標準化されたURL構造**: 予測可能なエンドポイント
- **メタデータサポート**: スキーマ情報の自動取得
- **CRUD操作対応**: Create, Read, Update, Delete操作をサポート
- **フィルタリング機能**: `$filter`, `$orderby`などのクエリオプション

## SAP APIサンドボックスの使用方法
// TODO


## Rest ClientでAPIを試す
### Step 1: Rest Clientのセットアップ
1. Rest Client拡張機能をVSCodeにインストール
   ![rest-client-install](../images/screenshots/d7-rest-client.png)
2. 新規ファイルを作成し、`.http`拡張子を付ける
   ![new-http-file](../images/screenshots/d7-new-http-file.png)

### Step 2: ファイルにGETリクエストを記述
```
GET https://8q5zg2p8tj.us-east-1.awsapprunner.com/odata/v4/order/OrderData
Headers:
  Accept: application/json
```

### Step 3: 基本的なGETリクエスト
![send-request](../images/screenshots/d7-get-request.png)
#### リスポンスが返却されることを確認
![response-check](../images/screenshots/d7-request-response.png)


### Step 4: ODataクエリオプションの使用
#### フィルタリング
特定IDの注文データを取得
```
GET https://8q5zg2p8tj.us-east-1.awsapprunner.com/odata/v4/order/OrderData?$filter=ID eq '12345'
Headers:
  Accept: application/json
```
![zero-response](../images/screenshots/d7-zero-response.png)

#### 上位N件の取得
上位3件を取得
```
GET https://8q5zg2p8tj.us-east-1.awsapprunner.com/odata/v4/order/OrderData?$top=3
Headers:
  Accept: application/json
```
![three-response](../images/screenshots/d7-three-responses.png)

#### ソート
オーダー作成日の古い順に取得（昇順 - asc）
```
GET https://8q5zg2p8tj.us-east-1.awsapprunner.com/odata/v4/order/OrderData?$orderby=created_at asc
Headers:
  Accept: application/json
```
![date-asc](../images/screenshots/d7-date-asc.png)

#### 特定フィールドの選択
ID、顧客コード、作成日のみ取得
```
GET https://8q5zg2p8tj.us-east-1.awsapprunner.com/odata/v4/order/OrderData?$select=ID,customer_code,created_at
Headers:
  Accept: application/json
```
![select-fields](../images/screenshots/d7-select-fields.png)

#### 複合クエリ
複数条件を組み合わせて取得:
数量が5より大きい注文を作成日降順で2件取得し、ID、顧客コード、数量、作成日のみ表示
```
GET https://8q5zg2p8tj.us-east-1.awsapprunner.com/odata/v4/order/OrderData?$filter=quantity gt 5&$orderby=created_at desc&$top=2&$select=ID,customer_code,quantity,created_at
Headers:
  Accept: application/json
```
![complex-query](../images/screenshots/d7-complex-query.png)

#### エラー時のレスポンス
GETリクエストで存在しないフィールドを指定した場合のレスポンス例

```
GET https://8q5zg2p8tj.us-east-1.awsapprunner.com/odata/v4/order/OrderData?$filter=nonexistentField eq 'value'
Headers:
  Accept: application/json
```
![error-response](../images/screenshots/d7-error-response.png)

## ハンズオン演習
### ハンズオン 1: POSTを使ったデータ作成
新しい注文データを作成するPOSTリクエストを実行してみましょう。

```
POST https://8q5zg2p8tj.us-east-1.awsapprunner.com/odata/v4/order/OrderData
Headers:
  Content-Type: application/json
Body:

{
  "ID": ,
  "customer_code": ,
  "product_code": ,
  "quantity": ,
  "unit_price": ,
  "estimated_cost": ,
  "delivery_date": ,
  "created_at": 
}
```

#### 確認ポイント
- ステータスコード HTTP/1.1 201 Created が返却されること
- JSONレスポンスが正しく取得できること

### ハンズオン 2: 作成したデータの取得
GETリクエストとODataクエリオプションを使って、先ほど作成した注文データのみを取得してみましょう。

####　ヒント
- `$filter`を使用して、特定の`ID`でフィルタリング
  
<details>
<summary>解答例</summary>
```http
GET https://8q5zg2p8tj.us-east-1.awsapprunner
.com/odata/v4/order/OrderData?$filter=ID eq 'your-created-id'
Headers:
  Accept: application/json
```
</details>
&nbsp;
#### 確認ポイント
- 作成した注文データが1件だけ返却されること
- レスポンスの内容が正しいこと
- ステータスコード HTTP/1.1 200 OK が返却されること


### ハンズオン 3: 商品マスタの検索
商品マスタAPIを使用して商品情報を検索してみましょう。

```http
GET /s4hanacloud/sap/opu/odata/sap/API_PRODUCT_SRV/A_Product
```


### ハンズオン 4: エラーハンドリングの確認
わざと間違ったリクエストを作成してエラーレスポンスを確認してみましょう。

1. 不正なAPIキーの使用
2. 存在しないフィールドでのフィルタリング
3. 権限エラーの確認


## トラブルシューティング
### よくあるエラーと対処法

1. **404 Not Found**
   - URLのパスが正しいかを確認
   - サービスの名前を再確認

2. **400 Bad Request**
   - ODataクエリの構文が正しいかを確認
   - フィルタ条件の記述を確認

3. Rest Clientでリクエストが送信できない
   - ファイルの拡張子が`.http`または`.rest`であることを確認
   - Rest Client拡張機能が有効になっていることを確認

4. Header name must be a valid HTTP token ["{"] のエラー
   - Bodyセクションの後に空行を入れる


## 参考資料
- [SAP API Business Hub](https://api.sap.com/)
- [OData公式ドキュメント](https://www.odata.org/)
- [SAP ODataプログラミング](https://help.sap.com/docs/SAP_NETWEAVER_750/68bf513362174d54b58cddec28794093/59283fc4528f486b83b1a58a4f1063c0.html)
- [Rest Client拡張機能](https://marketplace.visualstudio.com/items?itemName=humao.rest-client)