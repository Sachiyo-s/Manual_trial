---
sidebar_position: 1
---

# POST ユーザーインポート（非同期）

```http
/users/import
```

## 説明

対象となる組織に対して、CSVデータに含まれるユーザー情報をインポートします。

- 対象となる組織に新規のユーザーを作成する（更新は不可）
  - 別組織に作成済みユーザーが存在する場合は、複数の組織に所属するように作成する
  - ユーザー項目の扱いは、[ユーザー作成API](../POST%20ユーザー作成.md) と同様
- [ユーザーCSVフォーマット](/docs/common/ユーザーCSVフォーマット.md)
- 改行文字は CRLF または CR または LF をインポート可能とする
- ユーザーがアップロードするCSVファイルのサイズ上限は 500KB とする
  - 少なくとも 2000～3000人程度は一括でインポート可能とする
- ユーザーがアップロードしたCSVの有効期限は1日とする（1日で自動削除される）
  - CSVはS3オブジェクトとして保存する
- DynamoDBをイベントソースとしたインポート処理は、シャードあたりの同時バッチ（parallelization_factor）を 10 以上として、バッチ処理が詰まらないようにする

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Content-Type | コンテンツ種類 |  required | String | `multipart/form-data; boundary=...` |
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

### マルチパートBODY（file）

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| account_id | アカウントID | not required | String | 指定されても無視する（予備項目） |
| login_name | ログイン名 | required | String | |
| email | メールアドレス | required | String | |
| preferred_username | 表示名 | required | String | |
| family_name | 姓 | required | String | |
| given_name | 名 | optional | String | |
| family_kana | 姓カナ | required | String | |
| given_kana | 名カナ | optional | String | |

```http
Content-Disposition: form-data; name="file" filename="sample.csv"
Content-Type: application/octet-stream

Ver1.0
アカウントID,ログイン名,メールアドレス,表示名,姓,名,姓カナ,名カナ
xxxx-xx-xx-xxxx,yamada,yamada@email.com,総務部_山田太郎,山田,太郎,ヤマダ,タロウ
```

### マルチパートBODY（send_invitation_mail）

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| send_invitation_mail | 招待メールを送信するかどうか | required | Boolean | 送信する: `true`（デフォルト）<br/>送信しない: `false` |

```http
Content-Disposition: form-data; name="send_invitation_mail"
Content-Type: text/plain; charset="UTF-8"

true
```

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| task_id | タスクID | required | String | [インポートの実行状態を取得する](./tasks/task_id/POST%20ユーザーインポート状態取得.md)ために利用する |

```json
{
  "task_id": "0b671c50-5dd8-11ee-b7de-4b16f1ee9b45"
}
```
