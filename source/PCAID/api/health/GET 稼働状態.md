# GET 稼働状態

```http
/api/health
```

- どの API グループ（`/api/****/v1`）にも属さない
- 本番環境であれば、<https://id.pca.jp/api/health> でアクセスできる
  - 環境ごとのドメインは [PCA ID ドメイン＆ベースURL定義](/docs/common-dev/pcaid-domain-and-baseurl.md)を参照する

## 説明

現在のシステム稼働状態を返します。

- 認証なしでアクセスを可能とする
- 別ドメインからの Ajax アクセスを可能とする
  - CORS 設定ですべてのドメインからのアクセスを許可する
- 稼働状況ページが利用する想定のため、レスポンスのデータ構造は PCA Hub と互換性のあるもにする
  - <https://pca1980.backlog.com/alias/wiki/1543785>
- アプリID `pcaid` に対して、正常（`healthy`）と判定する条件は以下の通りとする
  - Keycloak healthcheck 機能で確認する
    - Keycloak に対して Database (Aurora PostgreSQL) への接続を含めてチェックする
    - <https://docs.redhat.com/en/documentation/red_hat_build_of_keycloak/22.0/html/server_guide/health->
  - 既知の DynamoDB データを読めることを確認する
  - 既知の S3 データを読めることを確認する

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |

### BODY

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| services | サービス一覧 | required | Object配列 | |
| app_id | アプリID <br/> ※外部参照されない任意の値 | required | String | 「`pcaid`」のみ <br/> ※2024/08 時点 |
| status | 稼働状態 | required | String | 異常：`unhealthy` <br/> 劣化：`degraded` <br/> 正常：`healthy` |

```json
{
  "services": [
    {
      "app_id": "pcaid",
      "status": "healthy"
    }
  ]
}
```
