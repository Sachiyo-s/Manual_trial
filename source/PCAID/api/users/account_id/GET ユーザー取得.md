---
sidebar_position: 1
---

# GET ユーザー取得

```http
/users/{account_id}
```

## 説明

アカウントIDを指定して、PCAアカウントのユーザー情報の取得します。

- アカウントIDとは、Keycloak User の内部ID（ID Token の sub）を指す
- URLパスに指定される `account_id` でユーザーを特定する
- 特定した組織の所属ユーザーであることを確認する
- [PCAアカウント](/docs/common/PCAアカウント.md) の「アカウント属性項目」を参照

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | [OAuthアクセストークン](/docs/common-dev/oidc-oauth-token-rules.md) | required | String | [管理APIの認可方法](/docs/common-dev/pcaid-management-api-granttype.md) |
| X-PCA-organization-id | 組織ID | conditional | String | ・[サービス区画](/docs/common/サービス区画.md)（X-PCA-service-partition）が未指定の場合は必須 <br/> ・任意の組織を指定して利用されることが可能だが、原則として、所属先となる呼び出し元の組織IDを指定する <br/> ・両方指定された場合はこの値を優先する |
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
| なし | | | | |

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| login_name | ログイン名 | requried | String | ・[PCAアカウント](/docs/common/PCAアカウント.md) の「ログイン名」を参照 |
| email | メールアドレス | requried | String | ・[PCAアカウント](/docs/common/PCAアカウント.md) の「メールアドレス」を参照 |
| preferred_username | 表示名 | requried | String | |
| family_name | 姓 | requried | String | |
| given_name | 名 | requried | String | 値が空文字列の場合あり |
| family_kana | 姓カナ | requried | String | |
| given_kana | 名カナ | requried | String | 値が空文字列の場合あり |
| lockout_status | ロックアウト状態 | requried | String | ・通常（未ロックアウト）：`active` <br/>・ロックアウト中：`locked` |
| lockout_at | ロックアウト日時 | requried | String | ISO8601 UTC 形式 |
| account_status | アカウント状態 | requried | String | ・有効：`active` <br/>・無効：`inactive` |

```json
{
  "login_name": "yamada",
  "email": "yamada@email.com",
  "preferred_username": "総務部_山田太郎",
  "family_name": "山田",
  "given_name": "太郎",
  "family_kana": "ヤマダ",
  "given_kana": "タロウ",
  "lockout_status": "active",
  "lockout_at": "2020-04-01T12:30:45Z",
  "account_status": "active",
}
```
