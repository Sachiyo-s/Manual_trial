# POST ユーザーからのサービス区画削除

```http
/users/{account_id}/service_partitions/remove
```

## 説明

利用不可とする指定した[サービス区画](/docs/common/サービス区画.md)を、PCAアカウントから削除します。

- 指定された `account_id` のアカウントが存在しなければ成功と見なす（冪等な動作とする）
- 組織が特定できなければエラーとする
- 削除対象の[サービス区画](/docs/common/サービス区画.md)が存在しなければ何もしない
- 削除対象のサービス区画内で利用するロールについて、ユーザーへの割り当てを解除（紐づけを削除）する

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | [OAuthアクセストークン](/docs/common-dev/oidc-oauth-token-rules.md) | required | String | [管理APIの認可方法](/docs/common-dev/pcaid-management-api-granttype.md) |
| X-PCA-organization-id | 組織ID | conditional | String | ・[サービス区画](/docs/common/サービス区画.md)（X-PCA-service-partition）が未指定の場合は必須 <br/> ・任意の組織を指定して利用されることを想定する <br/> ・両方指定された場合はこの値を優先する |
| X-PCA-service-partition | [サービス区画](/docs/common/サービス区画.md) | conditional | String |  ・組織ID（X-PCA-organization-id）が未指定の場合は必須 <br/> ・原則として、呼び出し元のサービス区画を指定する |

```http
Authorization: Bearer xxxx-xxxx-xxxx-xxxx
```

```http
X-PCA-organization-id: xxxx-xx-xx-xxxx
```

```http
X-PCA-service-partition: pca.hub.tenant1
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
