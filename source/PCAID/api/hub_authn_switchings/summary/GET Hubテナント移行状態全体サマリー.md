# GET Hubテナント移行状態全体サマリー

```http
/hub_authn_switchings/summary
```

## 説明

PCA Hubのユーザー移行として、すべてのテナントを対象に移行状態と案内メール送信状態の一覧を取得します｡

- 移行状態データが存在するテナントの一覧を返す
- 基本的には更新APIで指定された開始準備と移行状態についてのデータを返す
  - [Hubテナント移行状態準備API PUT /hub_authn_switchings/prepare](./../prepare/PUT%20Hubテナント移行開始準備.md)
  - [Hubテナント移行状態更新API PUT /hub_authn_switchings](./../PUT%20Hubテナント移行状態更新.md)
  - テナント移行状態（簡易）の値のみ判定する
  - データソースに項目が存在しない場合、レスポンスは項目を用意した上で既定値とする
- 準備・状態データが存在しなければ既定のデータを返す
  - 日付の値は、`1868-09-08T00:00:00Z` とする
    - この値は PCA Hubで利用可能な最小の値のことで、未設定と解釈する
  - 文字列の値は、空文字とする
  - オブジェクトの値は、`null` とする
- 組織作成前の呼び出しも想定するAPIのため、組織の存在を前提にしない
  - アプリケーションログの組織関連項目は出力対象外とする
- リクエストが成功なら `302 Found` を返して、レスポンスの Location ヘッダーにJSONデータファイル（レスポンスボディ）のURLを設定する
  - Location ヘッダーの値は、Cloudfront の署名付きURLとする

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | [OAuthアクセストークン](/docs/common-dev/oidc-oauth-token-rules.md) | required | String | [管理APIの認可方法](/docs/common-dev/pcaid-management-api-granttype.md) |

```http
Authorization: Bearer xxxx-xxxx-xxxx-xxxx
```

### BODY

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| tenant_name | PCA Hub テナント名 | requried | String | |
| pcaid_released_at | PCA ID リリース時刻 | requried | String | ・社内システムが設定した値を返す <br/>・未設定なら `1868-09-08T00:00:00Z` とする |
| simple_switching_status | テナント移行状態（簡易） | requried | String | ・移行状態の種類は下記の表を参照 <br/>・未設定なら `not_started` とする |
| switching_status | テナント移行状態（詳細） | requried | String | ・PCA Hub が設定した値を返す <br/>・未設定なら空文字とする |
| failed_reason | 移行失敗時のエラーコード | requried | String | ・PCA Hub が設定した値を返す <br/>・未設定なら空文字とする |
| switching_start_at | 移行開始時刻 | requried | String | ・PCA Hub が設定した値を返す <br/>・未設定なら `1868-09-08T00:00:00Z` とする |
| switching_end_at | 移行終了時刻 | requried | String | ・PCA Hub が設定した値を返す <br/>・未設定なら `1868-09-08T00:00:00Z` とする |
| reminder_email | 移行案内メール設定 | requried | Object | ・オブジェクト未設定なら null を返す |
| 　send_start_from | 送信開始時刻 | requried | String | ・社内システムが設定した値を返す <br/>・未設定なら `1868-09-08T00:00:00Z` とする |
| 　send_frequency | 送信頻度 | requried | String | ・社内システムが設定した値を返す <br/>・未設定なら空文字とする |
| 　latest_sent_at | 最新送信時刻 | requried | String | ・PCA Hub が設定した値を返す <br/>・未設定なら `1868-09-08T00:00:00Z` とする |
| 　email_template_id | メールテンプレートID | requried | String | ・社内システムが設定した値を返す <br/>・未設定なら空文字とする |
| features_limit | 機能制限設定 | requried | Object | ・オブジェクト未設定なら null を返す |
| 　limit_start_from | 制限開始時刻 | requried | String | ・社内システムが設定した値を返す <br/>・未設定なら `1868-09-08T00:00:00Z` とする |
| 　limit_scope | 制限対象 | requried | String | ・社内システムが設定した値を返す <br/>・未設定なら空文字とする |

```json
{
  "summary_results": [
    {
      "tenant_name": "tenant1", 
      "pcaid_released_at": "2024-08-01T00:00:00Z", 
      "simple_switching_status": "not_started", 
      "switching_status": "not_started", 
      "switching_start_at": "1868-09-08T00:00:00Z", 
      "switching_end_at": "1868-09-08T00:00:00Z", 
      "reminder_email": {
        "send_start_from": "2024-08-01T00:00:00Z",
        "send_frequency": "60d",
        "latest_sent_at": "1868-09-08T00:00:00Z",
        "email_template_id": "template1",
      },
      "features_limit": {
        "limit_start_from": "2025-01-01T00:00:00Z",
        "limit_scope": "user.create",
      },
    },
    {
      "tenant_name": "tenant2", 
      "pcaid_released_at": "2024-08-15T00:00:00Z", 
      "simple_switching_status": "running", 
      "switching_status": "by_tenant_convert_failed", 
      "failed_reason": "unexpected",
      "switching_start_at": "2024-08-03T15:00:00Z", 
      "switching_end_at": "1868-09-08T00:00:00Z", 
      "reminder_email": null,
      "features_limit": {
        "limit_start_from": "2025-02-01T00:00:00Z",
        "limit_scope": "user.create",
      },
    },
    {
      "tenant_name": "tenant3", 
      "pcaid_released_at": "2024-07-01T00:00:00Z", 
      "simple_switching_status": "done", 
      "switching_status": "by_tenant_convert_done", 
      "failed_reason": "",
      "switching_start_at": "2024-07-15T10:00:00Z", 
      "switching_end_at": "2024-08-31T12:00:00Z", 
      "email_sending_reservation": "* * * * * mail-template-Id", 
      "email_latest_sent_at": "2024-07-01T00:05:00Z", 
      "reminder_email": {
        "send_start_from": "2024-07-01T00:00:00Z",
        "send_frequency": "60d",
        "latest_sent_at": "2024-07-01T05:00:00Z",
        "email_template_id": "template1",
      },
      "features_limit": null,
    }
  ]
}
```

- 日付 `1868-09-08T00:00:00Z` は PCA Hubで利用可能な最小の値のことで、未設定と解釈する

### 移行状態（簡易）の種類

| 種類 | 説明 | 判定 |
| -- | -- | -- | -- |
| not_started | PCA Hub テナント管理者が移行を開始していない状態 | `switching_start_at` と `switching_end_at` が未設定 |
| running | PCA Hub テナント内の移行を開始した状態 | `switching_start_at` が設定済み |
| done | PCA Hub テナントにおけるすべての移行が完了した状態 | `switching_end_at` が設定済み |
