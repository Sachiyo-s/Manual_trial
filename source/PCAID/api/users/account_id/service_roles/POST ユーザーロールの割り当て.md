---
sidebar_position: 1
---

# POST ユーザーロールの割り当て

```http
/users/{account_id}/service_roles
```

## 説明

指定したPCAアカウントに対して、紐付けられたサービス区画用に登録された[ロール](/docs/common/ロール（役割）.md)を割り当てます。

- 同じ[ロール](/docs/common/ロール（役割）.md)が割り当て済みなら何もしない
- ユーザーに紐付けられたサービス区画が特定できなければエラーとする
- 指定したロールを特定できなければエラーとする

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | [OAuthアクセストークン](/docs/common-dev/oidc-oauth-token-rules.md) | required | String | [管理APIの認可方法](/docs/common-dev/pcaid-management-api-granttype.md) |
| X-PCA-service-partition | [サービス区画](/docs/common/サービス区画.md) | required | String | ・原則として、呼び出し元のサービス区画を指定する |

```http
Authorization: Bearer xxxx-xxxx-xxxx-xxxx
```

```http
X-PCA-service-partition: pca.hub.tenant1
```

### BODY

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| service_roles | サービスロール一覧 | required | String | ・指定されたサービス区画用ロール（X-PCA-service-partitionと結合する）をユーザーへ割り当てる <br/>・例：`pca.hub.tdi/gs:admin` |

```json
{
  "service_roles": [
    "gs:admin", 
    "d:users"
  ]
}
```

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |
