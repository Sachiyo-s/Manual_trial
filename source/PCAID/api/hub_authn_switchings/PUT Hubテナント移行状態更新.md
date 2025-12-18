---
sidebar_position: 2
---

# PUT Hubテナント移行状態更新

```http
/hub_authn_switchings
```

## 説明

PCA Hubのユーザー移行として、テナントごとの移行状態を更新します｡

- 状態データが存在しなければ作成し、存在したら更新する
- Hubテナント名をキーとして、準備データと状態データをセットで管理する
  - [Hubテナント移行状態準備API PUT /hub_authn_switchings/prepare](./prepare/PUT%20Hubテナント移行開始準備.md)
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
| switching_status | テナント移行状態 | requried | String | ・移行状態の種類は下記の表を参照 |
| failed_reason | 移行失敗時のエラーコード | requried | String | ・最新のみ <br/> ・エラーなしなら空文字列とする |
| switching_start_at | 移行開始時刻 | requried | String | ・ISO8601 UTC 形式 <br/>・未設定なら `1868-09-08T00:00:00Z` とする |
| switching_end_at | 移行終了時刻 | requried | String | ・ISO8601 UTC 形式 <br/>・未設定なら `1868-09-08T00:00:00Z` とする |

```json
{    
  "switching_status": "by_tenant_convert_running",
  "failed_reason": "unexpected",  
  "switching_start_at": "2024-04-10T15:00:00Z", 
  "switching_end_at": "2024-04-10T15:10:00Z"
}
```

### 移行状態の種類

| 種類 | 説明 |
| -- | -- |
| not_started | PCA Hub テナント管理者が移行を開始していない状態 |
| by_tenant_convert_running | PCA Hub テナントのコンバートを開始した状態 <br/> PCA Hub テナント管理者がすべてのテナント内ユーザーを強制的に移行させる |
| by_tenant_convert_failed | PCA Hub テナントのコンバート中になんらかの原因で失敗した状態 |
| by_tenant_convert_done | PCA Hub テナントのコンバートが完了した状態 <br/>PCA Hub テナント管理者が移行を実施し、テナントにおけるすべての移行が完了する |

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |
