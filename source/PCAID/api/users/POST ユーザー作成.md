---
sidebar_position: 1
---

# POST ユーザー作成

```http
/users
```

## 説明

PCAアカウントとなるユーザーを作成します。

- PCA Hubにおいて、ユーザーを作成するとき、暗黙的にPCAアカウントを作成する
- 未所属の組織からは、作成済みのPCAアカウントに対する作成要求があり得ることを想定する
  - 他の作成条件をすべて満たしていれば、指定された組織に所属させて成功として扱う
  - 成功レスポンスとして、 `account_id` と共に `account_handling` の値 `OrganizationJoined` を返す
- アカウントが作成済みかどうかはメールアドレスで判定する
  - このAPIではメールアドレスを変更できない
- 以下の場合は成功とする
  - 指定された組織内に同じログイン名とメールアドレスを持つPCAアカウントが作成済み
    - 作成済みPCAアカウントとの紐づけ処理を続行できるように、成功レスポンスとして `account_id` と共に `account_handling` の値 `IdempotentAction` を返す
- 以下の場合はエラーとする
  - 存在しない組織IDが指定された
  - 指定された組織内に同じログイン名を持つPCAアカウントが作成済み
    - メールアドレスは異なる状況とする
    - このAPIでは所属済み組織のログイン名は変更できず、ログイン名の追加のみ可能とする
    - 作成済みPCAアカウントとの紐づけ処理を続行できるように、失敗レスポンスとして、エラーコード `ConflictOrgLoginName` と共に `conflict_account_id` を返す
  - 同じメールアドレスを持つPCAアカウントが作成済み
    - 指定された組織内で、異なるログイン名が登録済みか、ログイン名が未登録の状況とする
    - 作成済みPCAアカウントとの紐づけ処理を続行できるように、失敗レスポンスとして、エラーコード `ConflictOrgEmail` と共に `conflict_account_id` を返す

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
| login_name | ログイン名 | requried | String | ・[PCAアカウント](/docs/common/PCAアカウント.md) の「ログイン名」を参照 <br/> ・内部的には組織IDとセットで管理する <br/> ・未所属の組織IDならユーザー属性の追加となる |
| email | メールアドレス | requried | String |・作成済みアカウントに対する変更不可 |
| preferred_username | 表示名 | requried | String |・アカウント作成済みで、別組織に所属済みなら上書きしない <br/> ・例外として、どのサービスも利用していない場合は上書きする |
| family_name | 姓 | requried | String |・上書きルールは `preferred_username` と同様 |
| given_name | 名 | optional | String |・上書きルールは `preferred_username` と同様 |
| family_kana | 姓カナ | requried | String |・上書きルールは `preferred_username` と同様 |
| given_kana | 名カナ | optional | String |・上書きルールは `preferred_username` と同様 |

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
| account_id | アカウントID | requried | String |・[PCAアカウント](/docs/common/PCAアカウント.md) の「アカウント属性項目」を参照 |
| account_handling | アカウント処理 | requried | String | `Created`：アカウントを新規作成 <br/> `OrganizationJoined`：指定組織に所属（アカウントは作成済み）<br/> `IdempotentAction`：冪等性による振る舞い（指定組織には所属済み）|
| account_setup | アカウント設定状態 | requried | String | `Initial`：アカウント初期状態（パスワード等が未設定） <br/> `Completed`：アカウント設定済み <br/> ※ Ver2.0.5 から対応 |

```json
{
  "account_id": "xxxx-xx-xx-xxxx",
  "account_handling": "Created",
  "account_setup": "Initial"
}
```
