# PUT 組織予約変更

```http
/organizations/organization_reservations/{organization_name}/rename
```

## 説明

指定した[組織名](/docs/common/組織名.md)の予約を変更します。

- 上流サービスであるHub登録支援システムから呼び出されることを想定する
- URLパスに指定される `organization_name` で予約された組織名を特定する
  - 組織名の予約が未登録であれば、`404 Not Found` エラーを返す（`NotFound`）
- 変更前と変更後の組織名に対して、以下のケースでは、`400 BadRequest` エラーで予約の変更に失敗する
  - 不正な組織名（`InvalidArgument`）
- 変更後の組織名に対して、以下のケースでは、`400 BadRequest` エラーで予約の変更に失敗する
  - 予約済みの組織名（`DuplicatedName`）
    - [予約済みの組織名](/docs/common/予約済みの組織.md)
    - リクエストの制限要求に応じてチェック内容を切り替える
  - 利用中の組織名（`DuplicatedName`）
  - 過去に一度でも利用されたことのある組織名（`DuplicatedName`）

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
| organization_name | [組織名](/docs/common/組織名.md) | requried | String | 変更後の組織名をあらわす |
| pcaid_restrict_required | PCA ID 仕様に基づく制限を要求するかどうか | optional | String | ・要求する：`true`（デフォルト） <br/>・要求しない：`false` |
| hub_restrict_required | PCA Hub 仕様に基づく制限を要求するかどうか | optional | String | ・要求する：`true` <br/>・要求しない：`false`（デフォルト）|

```json
{
  "organization_name": "tenant1",
  "pcaid_restrict_required": true, 
  "hub_restrict_required": true
}
```

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |
