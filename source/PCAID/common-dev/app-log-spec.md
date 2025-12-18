---
sidebar_position: 53
---

# 53. アプリログ仕様

## アプリログの目的

- 障害の原因を特定するための情報として利用する
- システムのパフォーマンスを監視するための情報として利用する
- システムのセキュリティを監視するための情報として利用する
- データ破損の検知や復旧のための情報として利用する

## ログ要件

- アプリ機能としてリクエスト時のログを記録する
- 例外発生時のスタックトレースを記録する
- 握りつぶす例外は後で確認できるように記録する
- トレースID により追跡可能にする
  - トレースID はユーザーに対して画面表示する
  - 問い合わせにおいてトレースID やエラーメッセージをキーワードとして扱う
- 構造化ログで記録し AWS Athena でクエリ可能にする
  - Athena テーブル（Glue カタログ）の作成を含む

## 記録仕様

- 正常レスポンス時には必ず `inf` ログを記録する（リクエストログ）
- 例外が発生した場合は `exception` にスタックトレースを含めたログを出力する
- １つのログは１行 json の構造化ログとする
  - json の配列は利用しない
- 時刻は **UTC** とする
  - 時刻表示では必要に応じて日本時間などに変換する
- PCA ID の情報（**PcaId** プレフィックスのログ項目）を可能な限り含める
- ログ出力がアプリ性能に悪影響を与えないように収集・保存は非同期にする
- ビジネスロジックと切り離した横断的実装として設計する

### 構造化ログ（JSONログ）

以下の例のように、１つのログごとに１行の json オブジェクトとする

```json
{ "text":"ログ1" }
{ "text":"ログ2" }
```

### ログレベル（**lv**）

- `inf`: 情報
  - リクエストログとして記録する場合
    - **elapsed** を埋める必要があるためレスポンスを返すタイミング（ロジック出口）で記録する
  - トラブルシュートに必要と思われる情報をメッセージ（**mt**）として記録する
- `err`: 予期せぬ例外エラー
  - スタックトレース（**exception**）を記録する
- `wrn`: 警告
  - 例外を握りつぶした場合
  - 通常呼ばれないはずの処理が呼ばれた場合
- `ftl`: 致命的なエラー
  - システムが停止するようなエラー
  - ユーザーや組織の取り違えを検知した場合

:::caution

デバッグログは本番では出力しない

:::

### トレースID（**TraceId**）

- トレースIDは、外側システムから伝搬されてくるトレースペアレントを引き継ぐ
  - W3C Trace Context を利用した分散トレースとして実装する
  - [参考:SpanId、TraceId、ParentIdってなに？W3C Trace Context を利用した分散トレースの利用](https://qiita.com/karuakun/items/7f75c774a22d69590125)

### S3保存とAthenaクエリー

- 専用のS3バケットとする
- Glue のカタログを生成して Athena でクエリーを実行できるようにする
  - [Amazon Athena で SQL クエリを実行する \- Amazon Athena](https://docs.aws.amazon.com/ja_jp/athena/latest/ug/querying-athena-tables.html)
  - [【初心者向け】AthenaからクエリしてS3上のデータを取得してみた \| DevelopersIO](https://dev.classmethod.jp/articles/get_s3data_by_athena/)
- パス構成は `stream/2024/04/06/` のように Athena クエリーに最適化したパーティショニングとする
- ECS のコンテナアプリは標準出力からログを収集する
  - ECS -> FireLens -> Firehose -> S3
  - ECS -> CloudWatch Logs -> Firehose -> S3
- Lambda アプリは CloudWatch Logs からログを収集する
  - Lambda -> CloudWatch Logs -> Firehose -> S3

### 記録不可データ

- 以下の情報は記録しないようにする
  - パスワードやクレジットカード情報などの機密情報
  - アクセストークンやリフレッシュトークン
  - ログインクッキー
  - シークレットキーや秘密鍵などのシステム上の重要設定
- リクエストボディやURLクエリーの一部に含まれている場合は、マスク処理として `[redacted]` で置き換える
  - 大文字小文字が違うと加工エラーになるので注意すること

## アプリログ項目

| No | 項目           | 例                                                                               | 説明                                                                             |
| --: | -------------- | -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| 1 | dt             | "2023-04-05T20:51:17.4430031Z"                                                   | ログの日時、必須、timestamp 型に変換可能な値                               |
| 2 | lv             | "inf" or "wrn" or "err" or "ftl"                                                 | ログのレベル                                                                     |
| 3 | mt             | "Request finished in {Elapsed}ms {HttpStatusCode}"                               | ログのメッセージテンプレート                                                     |
| 4 | exception             | "System.NotImplementedException: The method or operation is not implemented." | エラー時のスタックトレースなど                                                     |
| 5 | Elapsed        | 95.1888                                                                          | リクエストの処理時間 (ミリ秒)、double 型                                         |
| 6 | HttpStatusCode | 200                                                                              | HTTP ステータスコード、int 型                                                    |
| 7 | Payload        | 後述                                                                             | リクエストのペイロード (後述)、struct 型                                         |
| 8 | ClientId       | "pca-integration"                                                                | トークンの client_id クレーム                                                    |
| 9 | ObjectSize     | 63589                                                                            | オブジェクトのサイズ (eDOC 以外は常に 0)、bigint 型                              |
| 10 | TenantName     | "tenant1"                                                                        | テナント名                                                                       |
| 11 | TraceId        | "00-292b0ba49acb5017415dba0aa4795de0-94717d4497686bef-00"                        | W3C 形式トレース ID                                                              |
| 12 | TIssuedAt      | 1710462757                                                              | トークンの exp クレーム、bigint 型                                               |
| 13 | TExpiresAt     | 1710459157                                                              | トークンの iat クレーム、bigint 型                                               |
| 14 | TId            | "9b741d54-ca15-4b57-8cce-edec2170ffa1"                 | トークンの oi_tkn_id クレーム                                                    |
| 15 | SId            | "ye7bjczwl9sx"                                                               | トークンの sub クレーム (テナントユーザーID)                                  |
| 16 | SourceContext  | "PCA.ApShared.Logger.AppLogger.PcaAppLogger"                                     | ログを記録したクラス名                                                           |
| 17 | ResourceName   | "edocv2", "edocv2-aiocr" <br/> "pcaplus-edocv2-upload-commit-7f83bf$LATEST" など | ログを記録したリソース名                                                         |
| 18 | ResourceId     | "401cfe9eb67142c191d14e161b003c49-545564089"                                     | ログを記録したリソース ID <br/> ECS はコンテナランタイム ID、Lambda は RequestId |
| 19 | PcaIdOId        | "88ac9b8b-aa87-494d-a3e9-a009dc023737"                                                               | PCA IDの組織ID                                |
| 20 | PcaIdOn         | "tenant1"                                                               | PCA IDの組織名                                 |
| 21 | PcaIdSp          | "pca.hub.tenant1"                                                          | PCA IDのサービス区画                           |
| 22 | PcaIdId           | "8dbdb2b3-c577-4624-a4d8-e4644cdafb46"                                                               | PCA IDのアカウントID                                |
| 23 | PcaIdCId        | "abcdefg"                                                               | 社内システムで利用している顧客ID <br/> ※組織への顧客IDの割り当てはPCAが後から実行するため､アクセストークン生成時にPCA IDにデータがあれば設定されます                       |

PCA Hub 実装 [PcaAppLogger.cs](https://github.com/pca-web/ap-shared/blob/55eed8227325127e056c6ee0058913706b236a46/ApShared/ApShared/Logger/AppLogger/PcaAppLogger.cs)

### Payload 内の項目

| No | 項目                  | 例                                                                     | 説明                                  |
| --: | --------------------- | ----------------------------------------------------------------- | ------------------------------------- |
| 1 | ClientIp              | "202.240.53.143,64.252.113.71"                                             | クライアントの IP アドレス (IPv4)     |
| 2 | Host                  | "tenant1.pcahub.jp"                                                        | リクエストのホスト名                  |
| 3 | Path                  | "/auth/v1/connect/token"                                                   | リクエストのパス                      |
| 4 | IsHttps               | true                                                                       | HTTPS プロトコルを使用したか、bool 型 |
| 5 | Scheme                | "https"                                                                    | リクエストのスキーム                  |
| 6 | Method                | "POST"                                                                     | リクエストのメソッド                  |
| 7 | Protocol              | "HTTP/1.1"                                                                 | リクエストのプロトコル                |
| 8 | QueryString           | "appid=edoc20&limit=1"                                                     | クエリ文字列                          |
| 9 | Query                 | ["appid","limit"]                                                          | クエリ文字列のキー、string 配列       |
| 10 | RequestContentType    | "application/x-www-form-urlencoded"                                        | リクエストのコンテンツタイプ          |
| 11 | RequestHeaders        | {"Connection":"close","Host":"m-kuwahara-20230215-1.pcaplus-dev.net",...}  | リクエストのヘッダー、map 型          |
| 12 | RequestCookies        | {"has_auth"="abcduser0091",...}                                            | リクエストのクッキー、map 型          |
| 13 | RequestBody           | {"job_id": "1295ac96-3b7c-41bd-8ce3-eddd5fc73522",...}                     | リクエストのボディ、string 型         |
| 14 | RequestContentLength  | 134                                                                        | リクエストのコンテンツ長、bigint 型   |
| 15 | ResponseHeaders       | {"Connection":"close","Content-Type":"application/json;charset=UTF-8",...} | レスポンスのヘッダー、map 型          |
| 16 | ResponseContentType   | "application/json;charset=UTF-8"                                           | レスポンスのコンテンツタイプ          |
| 17 | ResponseContentLength | 1697                                                                       | レスポンスのコンテンツ長、bigint 型   |
| 18 | ServerIp              | "::ffff:127.0.0.1"                                                         | サーバーの IP アドレス                |
| 19 | ServerPort            | 80                                                                         | サーバーのポート番号、int 型          |
| 20 | HttpStatusCode        | 200                                                                        | HTTP ステータスコード、int 型         |

```json
{
  "payload": {
    "clientip": "string",
    "host": "string",
    "path": "string",
    "ishttps": "boolean",
    "scheme": "string",
    "method": "string",
    "protocol": "string",
    "querystring": "string",
    "query": "string[]",
    "requestcontenttype": "string",
    "requestheaders": {
      "key": "string",
      "value": "string"
    },
    "requestcookies": {
      "key": "string",
      "value": "string"
    },
    "requestbody": "string",
    "requestcontentlength": "bigint",
    "responseheaders": {
      "key": "string",
      "value": "string"
    },
    "responsecontenttype": "string",
    "responsecontentlength": "bigint",
    "serverip": "string",
    "serverport": "int",
    "httpstatuscode": "int"
  }
}
```

PCA Hub 実装 [HttpPayload.cs](https://github.com/pca-web/ap-shared/blob/55eed8227325127e056c6ee0058913706b236a46/ApShared/ApShared/Logger/AppLogger/HttpPayload.cs)

### Glue の Schema 定義

| No |column name|data type|
| --: | -- | -- |
| 1 |dt|string|
| 2 |lv|string|
| 3 |mt|string|
| 4 |exception|string|
| 5 |elapsed|double|
| 6 |httpstatuscode|int|
| 7 |payload|struct|
| 8 |clientid|string|
| 9 |traceid|string|
| 10 |tissuedat|bigint|
| 11 |texpiresat|bigint|
| 12 |tid|string|
| 13 |sourcecontext|string|
| 14 |resourcename|string|
| 15 |resourceid|string|
| 16 |pcaidoid|string|
| 17 |pcaidon|string|
| 18 |pcaidsp|string|
| 19 |pcaidid|string|
| 20 |pcaidcid|string|

PCA Hub 実装 [raw_app_log_stream](https://ap-northeast-1.console.aws.amazon.com/glue/home?region=ap-northeast-1#/v2/data-catalog/tables/view/raw_app_log_stream?database=pcaplus_app_log_7f83bf&catalogId=932757145519&versionId=latest)

### 【参考】PCA Hub 仕様

- [PCA Hub アプリケーションログの項目](https://github.com/pca-web/HubTechs/discussions/888)
- [PCA Hub アプリケーションログの例](https://github.com/pca-web/HubTechs/discussions/570)
- [設計/基礎設計/ログ](https://pca1980.backlog.com/alias/wiki/864455)
