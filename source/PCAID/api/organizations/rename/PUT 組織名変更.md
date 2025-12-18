# PUT 組織名変更

```http
/organizations/rename
```

## 説明

指定した組織ID、または呼び出し元の[サービス区画](/docs/common/サービス区画.md)に応じた[組織](/docs/common/組織.md)に対して、組織名を変更します。

- 組織名の重複は不可とする
- 以下のケースでは、`400 BadRequest` エラーで組織名の変更に失敗する
  - 不正な組織名（`InvalidArgument`）
  - 予約済みの組織名（`SystemReserved`）
    - [予約済みの組織名](/docs/common/予約済みの組織.md)
    - PCA ID と PCA Hub の要件をどちらも満たすようにチェックする
  - 利用中の組織名（`Used`）
  - 過去に一度でも利用されたことのある組織名（`Used`）

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
| organization_name | [組織名](/docs/common/組織.md) | requried | String | |

```json
{
  "organization_name": "tenant1",
}
```

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |
