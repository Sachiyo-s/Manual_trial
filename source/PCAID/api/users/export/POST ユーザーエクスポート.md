# POST ユーザーエクスポート

```http
/users/export
```

## 説明

指定した組織に所属するすべてのユーザー情報をCSVデータとしてエクスポートします。

- [ユーザーCSVフォーマット](/docs/common/ユーザーCSVフォーマット.md)
- 改行文字は CRLF とする
- ファイル名は「ユーザーリスト_yyyy-mm-dd_hh-mm-ss.csv」とする
  - 例：2024/02/21 16:45:10 にエクスポート時のファイル名は「ユーザーリスト_2024-02-21_16-45-10.csv」
- リクエストが成功なら `302 Found` を返して、レスポンスの Location ヘッダーにCSVファイルのURLを設定する
  - Location ヘッダーの値は、Cloudfront の署名付きURLとする

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Accept | 受入コンテンツ種類 |  required | String | `text/csv` 固定 |
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
| なし | | | | |

## OUTPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Content-Type | コンテンツ種類 |  required | String | `text/csv; charset=utf-8` 固定 |
| Content-Disposition | コンテンツ配置 |  required | String | `attachment; filename=ユーザーリスト_yyyy-mm-dd_hh-mm-ss.csv` |

### BODY

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| account_id | アカウントID | required | String |・常に空（予備項目） |
| login_name | ログイン名 | required | String | |
| email | メールアドレス | required | String | |
| preferred_username | 表示名 | required | String | |
| family_name | 姓 | required | String | |
| given_name | 名 | optional | String | |
| family_kana | 姓カナ | required | String | |
| given_kana | 名カナ | optional | String | |

```csv
Ver1.0
アカウントID,ログイン名,メールアドレス,表示名,姓,名,姓カナ,名カナ
,yamada,yamada@email.com,総務部_山田太郎,山田,太郎,ヤマダ,タロウ
```
