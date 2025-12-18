---
sidebar_position: 1
---

# GET Hubテナント移行状態取得

```http
/hub_authn_switchings
```

## 説明

PCA Hubのユーザー移行として、テナントごとの開始準備・移行状態を取得します｡

- 更新APIで指定された開始準備と移行状態についてのデータを返す
  - [Hubテナント移行状態準備API PUT /hub_authn_switchings/prepare](./prepare/PUT%20Hubテナント移行開始準備.md)
  - [Hubテナント移行状態更新API PUT /hub_authn_switchings](./PUT%20Hubテナント移行状態更新.md)
- 準備・状態データが存在しなければ既定のデータを返す
  - 日付の値は、`1868-09-08T00:00:00Z` とする
    - この値は PCA Hubで利用可能な最小の値のことで、未設定と解釈する
  - 文字列の値は、空文字とする
  - オブジェクトの値は、`null` とする
- 組織作成前の呼び出しも想定するAPIのため、指定されたサービス区画に紐づく組織の存在を前提にしない
  - アプリケーションログの組織関連項目は出力対象外とする

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | [OAuthアクセストークン](/docs/common-dev/oidc-oauth-token-rules.md) | required | String | [管理APIの認可方法](/docs/common-dev/pcaid-management-api-granttype.md) |
| X-PCA-service-partition | [サービス区画](/docs/common/サービス区画.md) | required | String |  ・呼び出し元のサービス区画を指定する |

```http
Authorization: Bearer xxxx-xxxx-xxxx-xxxx
```

```http
X-PCA-service-partition: pca.hub.tenant1
```

### BODY

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| pcaid_released_at | PCA ID リリース時刻 | requried | String | ・社内システムが設定した値を返す <br/>・未設定なら `1868-09-08T00:00:00Z` とする |
| switching_status | テナント移行状態 | requried | String | ・PCA Hub が設定した値を返す <br/>・移行状態の種類は[更新API](./PUT%20Hubテナント移行状態更新.md)の表を参照 <br/>・未設定なら `not_started` を返す |
| failed_reason | 移行失敗時のエラーコード | requried | String | ・PCA Hub が設定した値を返す <br/>・未設定なら空文字とする |
| switching_start_at | 移行開始時刻 | requried | String | ・PCA Hub が設定した値を返す <br/>・未設定なら `1868-09-08T00:00:00Z` とする |
| switching_end_at | 移行終了時刻 | requried | String | ・PCA Hub が設定した値を返す <br/>・未設定なら `1868-09-08T00:00:00Z` とする |
| reminder_email | 移行案内メール設定 | requried | Object | ・社内システムが設定した値を返す <br/>・オブジェクト未設定なら null を返す |
| 　send_start_from | 送信開始時刻 | requried | String | ・社内システムが設定した値を返す <br/>・未設定なら `1868-09-08T00:00:00Z` とする |
| 　send_frequency | 送信頻度 | requried | String | ・社内システムが設定した値を返す <br/>・未設定なら空文字とする |
| 　latest_sent_at | 最新送信時刻 | requried | String | ・PCA Hub が設定した値を返す <br/>・未設定なら `1868-09-08T00:00:00Z` とする |
| 　email_template_id | メールテンプレートID | requried | String | ・社内システムが設定した値を返す <br/>・未設定なら空文字とする |
| features_limit | 機能制限設定 | requried | Object | ・社内システムが設定した値を返す <br/>・オブジェクト未設定なら null を返す |
| 　limit_start_from | 制限開始時刻 | requried | String | ・社内システムが設定した値を返す <br/>・未設定なら `1868-09-08T00:00:00Z` とする |
| 　limit_scope | 制限対象 | requried | String | ・社内システムが設定した値を返す <br/>・未設定なら空文字とする |

```json
{    
  "pcaid_released_at": "2024-04-01T00:00:00Z",
  "switching_status": "by_tenant_convert_running",
  "failed_reason": "unexpected",  
  "switching_start_at": "2024-04-10T15:00:00Z", 
  "switching_end_at": "2024-04-10T15:10:00Z",
  "reminder_email": {
    "send_start_from": "2024-04-01T00:00:00Z",
    "send_frequency": "60d",
    "latest_sent_at": "1868-09-08T00:00:00Z",
    "email_template_id": "template1",
  },
  "features_limit": {
    "limit_start_from": "2024-04-01T00:00:00Z",
    "limit_scope": "user.create",
  },
}
```

### 状態データが存在しない場合

```json
{
  "pcaid_released_at": "1868-09-08T00:00:00Z",
  "switching_status": "not_started",
  "failed_reason": "",
  "switching_start_at": "1868-09-08T00:00:00Z",
  "switching_end_at": "1868-09-08T00:00:00Z",
  "reminder_email": null,
  "features_limit": null,
}
```

- 日付 `1868-09-08T00:00:00Z` は PCA Hub で利用可能な最小の値のことで、未設定と解釈する
