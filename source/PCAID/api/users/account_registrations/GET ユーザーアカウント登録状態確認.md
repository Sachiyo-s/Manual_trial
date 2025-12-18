# GET ユーザーアカウント登録状態確認

```http
/account_registrations/{email}/status
```

## 説明

メールアドレスを指定して、PCAアカウントの登録・設定状態を取得します。

- URLパスに指定される `email` でユーザーアカウントを特定する
- 対象アカウントが未登録なら `404 Not Found` エラーとして、規定のエラー応答を返す（例の記載あり）

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
| account_status | アカウント状態 | requried | String | ・有効：`active` <br/>・無効：`inactive` |
| email_verified | メールアドレス確認状態 | requried | String | ・確認済み： `enable`<br/>・未確認： `unable` |
| credentials | クレデンシャル一覧 | requried | String配列 | ・パスワード：`password`<br/>・バックアップコード：`backupcode` <br/>・パスキー：`passkey`<br/>※重複値は１つにする |

```json
{
  "account_status": "active",
  "email_verified": "enable"
  "credentials": [ "password", "backupcode", "passkey" ],
}
```

### 404 Not Found エラー時

```json
{
  "status": 404,
  "code": "AccountNotFound",
  "message": "指定されたアカウントが存在しません。",
  "domain": "pcaid",
  "trace": "00-7771d76ec5783c7d09a4eefb28938c74-5e81b5182264d1b8-00"
}
```
