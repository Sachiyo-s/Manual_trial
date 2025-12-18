---
sidebar_position: 1
---

# POST 招待メール送信

```http
/users/{account_id}/invite
```

## 説明

PCAアカウントのユーザーに招待メールを送信します。

- 基本的に組織管理機能（管理コンソール）、または PCA Hub からの操作となる
- URLパスに指定される `account_id` でユーザーを特定する
- 特定した組織の所属ユーザーであることを確認する
- 内部的には、Keycloak Admin REST API を呼び出す
  - `PUT /admin/realms/{realm}/users/{user-id}/execute-actions-email`
  - クエリーパラメーターには、以下の値（Optional）を指定する
    - lifespan
    - client_id
    - redirect_uri
  - ボディには、以下の文字列配列をJSONで指定する
    - "VERIFY_EMAIL"
    - "UPDATE_PASSWORD"
    - "CONFIGURE_RECOVERY_AUTHN_CODES"
- 招待メールの送信したら、[イベント通知](/docs/common-dev/events-notification-design.md)として `user.email-invited` を発行するために、SNSへメッセージを送信する

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
| lifespan | メールアクション期限（秒） | optional | Number | 1週間であれば 604800 秒 |
| clientId | OAuthクライアントID | optional | String | |
| redirectURI | リダイレクトURI | optional | String | クライアントに登録済みのコールバック先URL |

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |
