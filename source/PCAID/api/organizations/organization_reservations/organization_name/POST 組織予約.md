---
sidebar_position: 1
---

# POST 組織予約

```http
/organizations/organization_reservations/{organization_name}
```

## 説明

指定した[組織名](/docs/common/組織名.md)をもつ[組織](/docs/common/組織.md)を作成前に予約します。

- 上流サービスであるHub登録支援システムから呼び出されることを想定する
- 以下のケースでは予約に失敗する
  - 不正な組織名（`InvalidArgument`）
  - 利用不可（または予約済み）の組織名（`DuplicatedName`）
    - [予約済みの組織名](/docs/common/予約済みの組織.md)
    - リクエストの制限要求に応じてチェック内容を切り替える
  - 利用中の組織名（`DuplicatedName`）
  - 過去に一度でも利用されたことのある組織名（`DuplicatedName`）
- [組織作成](./../../POST%20組織作成.md)は予約を前提とする
  - 組織作成時に予約からは削除する
- 過去に利用された組織名は履歴管理して再利用は不可とする

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
| pcaid_restrict_required | PCA ID 仕様に基づく制限を要求するかどうか | optional | String | ・要求する：`true`（デフォルト） <br/>・要求しない：`false` |
| hub_restrict_required | PCA Hub 仕様に基づく制限を要求するかどうか | optional | String | ・要求する：`true` <br/>・要求しない：`false`（デフォルト）|

※ボディが空の状況も想定してデフォルト値を適用し、PCA Hub からの呼び出しに対する互換性を維持する  
※いずれのパラメーターも外部公開しない想定とする（内部利用を想定したAPIとする）  

```json
{
  "pcaid_restrict_required": true, 
  "hub_restrict_required": true
}
```

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |
