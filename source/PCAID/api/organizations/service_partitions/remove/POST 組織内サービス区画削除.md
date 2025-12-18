# POST 組織内サービス区画削除

```http
/organizations/service_partitions/remove
```

## 説明

利用不可とする指定した[サービス区画](/docs/common/サービス区画.md)を、[組織](/docs/common/組織.md)から削除します。

- 上流サービスである契約・ライセンス管理システムから呼び出されることを想定する
- 対象の[サービス区画](/docs/common/サービス区画.md)が削除済みなら何もしない
- 対象組織内のユーザーに追加済みの[サービス区画](/docs/common/サービス区画.md)も一括で削除する
- 対象のサービス区画内で利用するロールも一括で削除する
  - ユーザーに対する対象ロールの紐づけも削除する

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
| service_partition | [サービス区画](/docs/common/サービス区画.md) | required | String | ・原則として、利用不可とする呼び出し元のサービスの区画（例：Hubならテナント名）を指定する |

```json
{
  "service_partition": "pca.hub.tdi"
}
```

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |
