# POST 組織予約削除

```http
/organizations/organization_reservations/{organization_name}/remove
```

## 説明

指定した[組織名](/docs/common/組織名.md)の予約を削除します。

- 上流サービスであるHub登録支援システムから呼び出されることを想定する
- URLパスに指定される `organization_name` で予約された組織名を特定する
  - 組織名の予約が未登録であれば、`404 Not Found` エラーを返す（`NotFound`）
- 以下のケースでは、`400 BadRequest` エラーで予約の削除に失敗する
  - 不正な組織名（`InvalidArgument`）
  - 利用中の組織名（`Used`）
  - 過去に一度でも利用されたことのある組織名（`Used`）

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
| なし | | | | |
