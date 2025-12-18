# PUT Hubテナント移行開始準備

```http
/hub_authn_switchings/prepare
```

## 説明

PCA Hubのユーザー移行として、テナントごとの移行開始に必要となる設定を準備します｡

- 準備データが存在しなければ作成し、存在したら更新する
- Hubテナント名をキーとして、準備データと状態データをセットで管理する
  - [Hubテナント移行状態更新API PUT /hub_authn_switchings](./../PUT%20Hubテナント移行状態更新.md)
- PCA IDリリース時刻（`pcaid_released_at`）は、リリース予定日として利用する
  - リリース前は、現在時刻より未来の値のみ保存できる
  - リリース後は、保存した時刻を変更できない
- 送信開始時刻（`send_start_from`）は、案内メール予定日として利用する
  - 未設定（`1868-09-08T00:00:00Z`）、または PCA IDリリース時刻（`pcaid_released_at`）以降の値のみ保存できる
- 制限開始時刻（`limit_start_from`）は、機能制限の予定日として利用する
  - 未設定（`1868-09-08T00:00:00Z`）、または PCA IDリリース時刻（`pcaid_released_at`）以降の値のみ保存できる
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
| pcaid_released_at | PCA IDリリース時刻 | requried | String <br/>ISO8601 | ・移行案内を開始する日時 <br/>・未来の時刻なら設定する <br/>・リリース（移行案内）済みなら変更不可として無視する |
| reminder_email | 移行案内メール設定 | optional | Object |・未指定ならオブジェクト内の項目を更新しない |
| 　send_start_from | 送信開始時刻 | requried | String <br/>ISO8601 | ・案内メール送信を開始する日時 <br/>・`pcaid_released_at` 以降とする <br/>・未設定（予定なし）なら `1868-09-08T00:00:00Z` とする |
| 　send_frequency | 送信頻度 | requried | String | ・日単位で指定する <br/> ・例えば30日ごとなら `30d` とする |
| 　latest_sent_at | 最新送信時刻 | optional | String <br/>ISO8601 | ・PCA Hubが案内メール送信時に更新する <br/>・項目が存在するときのみ上書きする <br/>・通常は設定不要 |
| 　email_template_id | メールテンプレートID | requried | String | ・案内メールの本文を区別するID |
| features_limit | 機能制限設定 | optional | Object | ・未指定ならオブジェクト内の項目を更新しない |
| 　limit_start_from | 制限開始時刻 | requried | String <br/>ISO8601 | ・機能制限を開始する日時 <br/>・`pcaid_released_at` 以降とする <br/>・未設定（予定なし）なら `1868-09-08T00:00:00Z` とする |
| 　limit_scope | 制限対象 | requried | String | ・機能単位で指定する <br/>・例えばユーザー作成なら `user.create` とする |

```json
{    
  "pcaid_released_at": "2024-04-01T00:00:00Z",
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

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |
