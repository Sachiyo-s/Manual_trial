---
sidebar_position: 2
---

# GET ユーザー一覧取得

```http
/users
```

## 説明

指定した組織、または呼び出し元のサービス区画における、ユーザー一覧を取得します。

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | [OAuthアクセストークン](/docs/common-dev/oidc-oauth-token-rules.md) | required | String | [管理APIの認可方法](/docs/common-dev/pcaid-management-api-granttype.md) |
| X-PCA-organization-id | 組織ID | conditional | String | ・[サービス区画](/docs/common/サービス区画.md)（X-PCA-service-partition）が未指定の場合は必須 <br/> ・任意の組織を指定して利用されることが可能だが、原則として、所属先となる呼び出し元の組織IDを指定する <br/> ・両方指定された場合はこの値を優先する |
| X-PCA-service-partition | [サービス区画](/docs/common/サービス区画.md) | conditional | String |  ・組織ID（X-PCA-organization-id）が未指定の場合は必須 <br/> ・PCA Hubサービスからの作成時は必須 <br/>・原則として、呼び出し元のサービス区画を指定する |

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
| account_id | アカウントID | requried | String | ・[PCAアカウント](/docs/common/PCAアカウント.md) の「アカウント属性項目」を参照 |
| login_name | ログイン名 | requried | String | |
| email | メールアドレス | requried | String | |
| preferred_username | 表示名 | requried | String | |
| family_name | 姓 | requried | String | |
| given_name | 名 | requried | String | 値が空文字列の場合あり |
| family_kana | 姓カナ | requried | String | |
| given_kana | 名カナ | requried | String | 値が空文字列の場合あり |
| lockout_status | ロックアウト状態 | requried | String | 要検討（N+1問題あり） |
| lockout_at | ロックアウト日時 | requried | String | 要検討（N+1問題あり） |
| account_status | アカウント状態 | requried | String | |

```json
{
  "users": [
    {
      "account_id": "xxxx-xx-xx-xxxx",
      "login_name": "yamada@email.com",
      "email": "yamada@email.com",
      "preferred_username": "総務部_山田太郎",
      "family_name": "山田",
      "given_name": "太郎",
      "family_kana": "ヤマダ",
      "given_kana": "タロウ",
      "lockout_status": "active",
      "lockout_at": "2020-04-01T12:00Z",
      "account_status": "active",
    },
  ]
}
```
