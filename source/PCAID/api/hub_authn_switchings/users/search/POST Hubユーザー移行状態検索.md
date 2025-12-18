# POST Hubユーザー移行状態検索

```http
/hub_authn_switchings/users/search
```

## 説明

PCA Hubのユーザー移行として、PCAアカウントのアカウントIDを指定してユーザーを検索します。

- 指定されたPCA Hubのサービス区画（`pca.hub.{tenant_name}`）から組織を特定する
- 組織に所属するユーザーのみ対象とし、未所属であれば検索結果に含めない
- 指定したアカウントIDに該当するユーザーが存在しなければ、検索結果に含めない
- 以下の場合はエラーとする
  - 検索条件とするアカウントIDが1件も指定されていない
  - 検索条件とするアカウントIDが100件を超えている

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
| account_ids | アカウントID一覧 | requried | String配列 | ・指定範囲は1～100件とする |

```json
{
  "account_ids": [
    "pca_id_account1", 
    "pca_id_account2"
  ]
}
```

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| users | ユーザー一覧 | requried | ユーザーデータ配列 |  |
| account_id | アカウントID | requried | String |・[PCAアカウント](/docs/common/PCAアカウント.md) を参照 |
| login_name | ログイン名 | requried | String | |
| email | メールアドレス | requried | String | |
| accept_policy | 同意ポリシー | requried | String | ・PCA ID利用規約への同意データ <br/>・未同意の場合は `none` とする |

```json
{
  "users": [
    {
      "account_id": "pca_id_account1",
      "longin_name": "yamada",
      "email": "yamada@email.com",
      "accept_policy": "20240501",
    },
    {
      "account_id": "pca_id_account2",
      "login_name": "satou",
      "email": "satou@email.com",
      "accept_policy": "none",
    },
  ]
}
```
