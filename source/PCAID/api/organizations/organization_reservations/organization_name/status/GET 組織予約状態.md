# GET 組織予約状態

```http
/organizations/organization_reservations/{organization_name}/status
```

## 説明

指定した[組織名](/docs/common/組織名.md)の予約状況を確認します。

- 上流サービスであるHub登録支援システムから呼び出されることを想定する
- 以下のケースでは利用不可を返す
  - 不正な組織名（`InvalidArgument`）
  - 利用不可（または予約済み）の組織名（`DuplicatedName`）
    - [予約済みの組織名](/docs/common/予約済みの組織.md)
    - リクエストの制限要求に応じてチェック内容を切り替える
      - GET API のため、制限要求にはクエリーパラメーターを使用する
  - 利用中の組織名（`DuplicatedName`）
  - 過去に一度でも利用されたことのある組織名（`DuplicatedName`）
- 以下のケースでは取り消し可能とする
  - [組織予約](./../POST%20組織予約.md)により予約した組織名である
  - 組織名の利用が開始されていない

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | [OAuthアクセストークン](/docs/common-dev/oidc-oauth-token-rules.md) | required | String | [管理APIの認可方法](/docs/common-dev/pcaid-management-api-granttype.md) |

```http
Authorization: Bearer xxxx-xxxx-xxxx-xxxx
```

### QUERY PARAMETER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| pcaid_restrict_required | PCA ID 仕様に基づく制限を要求するかどうか | optional | String | ・要求する：`true`（デフォルト） <br/>・要求しない：`false` |
| hub_restrict_required | PCA Hub 仕様に基づく制限を要求するかどうか | optional | String | ・要求する：`true` <br/>・要求しない：`false`（デフォルト）|

※すべてのパラメーターが未指定であってもデフォルト値を適用し、PCA Hub からの呼び出しに対する互換性を維持する  
※いずれのパラメーターも外部公開しない想定とする（内部利用を想定したAPIとする）  

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| status | 予約状況 | required | String | `available` または `not_available` |
| cancellable | 取り消し可能 | required | Boolean | `true` または `false` |

```json
{
  "status": "available",
  "cancellable": "true",
}
```
