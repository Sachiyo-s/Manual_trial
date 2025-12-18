---
sidebar_position: 2
---

# PUT ユーザー更新

```http
/users/{account_id}
```

## 説明

PCAアカウントのユーザー情報を更新します。

- URLパスに指定される `account_id` でユーザーを特定する
- 組織内において、重複するログイン名への変更は不可とする
- PCA ID全体において、重複するメールアドレスへの変更は不可とする
- 以下の場合はエラーとする
  - 存在しない組織IDが指定された
  - 指定された組織の所属ユーザーではなかった
  - 指定された組織内に同じログイン名を持つPCAアカウントが既に存在する
    - 失敗レスポンスとして、エラーコード `ConflictOrgLoginName` を返す
  - 同じメールアドレスを持つPCAアカウントが既に存在する
    - 失敗レスポンスとして、エラーコード `ConflictOrgEmail` を返す
  - 管理者（is_self_update = false）が複数の組織に所属しているユーザーのメールアドレスを変更する
    - 失敗レスポンスとして、エラーコード `MultipleOrgEmail` を返す
    - 組織管理者が所属ユーザーを乗っ取ることで、別組織のユーザーになりすますことを防止する
    - 本人による変更（is_self_update = true）であればエラーにしない

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
| login_name | ログイン名 | requried | String | ・[PCAアカウント](/docs/common/PCAアカウント.md) の「ログイン名」を参照 <br/> ・内部的には組織IDとセットで管理する <br/> ・ログイン名は組織IDごとに一意とする <br/> ・別組織IDのログイン名は変更不可とする |
| email | メールアドレス | requried | String | ・一般ユーザーによる変更には Keycloak標準フローを利用するため値を維持する <br/> ・管理者による強制変更ではこの値を書き換えて更新する |
| preferred_username | 表示名 | requried | String | ・上書きする |
| family_name | 姓 | requried | String | ・上書きする |
| given_name | 名 | optional | String | ・上書きする |
| family_kana | 姓カナ | requried | String | ・上書きする |
| given_kana | 名カナ | optional | String | ・上書きする |
| is_self_update | 本人による更新 | optional | Bool | ・`true`: 本人 <br/>・`false`: 管理者（デフォルト） <br/> ・項目ごと未指定なら `false` として扱う |

```json
{
  "login_name": "yamada",
  "email": "yamada@email.com",
  "preferred_username": "総務部_山田太郎",
  "family_name": "山田",
  "given_name": "太郎",
  "family_kana": "ヤマダ",
  "given_kana": "タロウ",
}
```

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |
