---
sidebar_position: 1
---

# PUT 組織画像アップロード

```http
/organizations/images/logo
```

## 説明

指定した組織ID、または呼び出し元の[サービス区画](/docs/common/サービス区画.md)に応じた[組織](/docs/common/組織.md)に対して、会社ロゴ等の画像をアップロードします。

- 既存の組織画像は上書きする
- 画像データは所定のS3に保存してS3パスを管理する

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Content-Type | コンテンツ種類 |  required | String | `multipart/form-data` 固定 |
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

### マルチパートBODY（binary）

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| binary | 組織画像 | requried | Binary | ・バイナリ画像データ <br/> ・1MB未満 <br/>・`.jpg` / `.bmp` / `.png` / `.svg` |

```http
Content-Disposition: form-data; name="binary"
Content-Type: image/png

(binary data ...)
```

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |
