---
sidebar_position: 2
---

# GET プロフィール画像取得

```http
/users/{account_id}/images/profile
```

## 説明

PCAアカウントのプロフィール画像を取得します。

- URLパスに指定される `account_id` でユーザーを特定する
- 特定した組織の所属ユーザーであることを確認する
- プロフィール画像はすべての所属組織で共用する
- 画像データは所定のS3に保存してS3パスを管理する
- ユーザーによってアップロードされたプロフィール画像がなければ、デフォルト画像を返す
  - [PCA Hub で利用しているテナント・プロフィール画像のデフォルト画像](/docs/common/materials/pcahub-default-images.md)
- リクエストが成功なら `302 Found` を返して、レスポンスの Location ヘッダーに画像ファイルのURLを設定する
  - Location ヘッダーの値は、Cloudfront の署名付きURLとする

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Accept | 受入コンテンツ種類 |  required | String | `image/*` 固定 |
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
| Content-Type | コンテンツ種類 |  required | String | ・`image/jpeg` <br/> ・`image/bmp` <br/> ・`image/png`  <br/> ・`image/svg+xml` |

### BODY

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| --- | | requried | Binary | ・バイナリ画像データ <br/> ・`.jpg` / `.bmp` / `.png` / `.svg` |
