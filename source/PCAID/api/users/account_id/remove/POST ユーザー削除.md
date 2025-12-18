---
sidebar_position: 1
---

# POST ユーザー削除

```http
/users/{account_id}/remove
```

## 説明

PCAアカウントのユーザーを組織から削除します。

- 基本的に組織管理機能（管理コンソール）からの操作となる
- URLパスに指定される `account_id` でユーザーを特定する
- 特定した組織の所属ユーザーであることを確認する
- 現在の組織のみ所属している（結果として未所属になる）ユーザーの削除は、物理的な削除とする
  - 管理されない個人情報が発生することを避けるため、どの組織にも所属しないユーザーは作らないようにする
- 他組織にも所属しているユーザーの削除は、現在の組織に所属していない状態にする
  - 現在の組織に紐づくサービス区画からは取得できなくなる
  - 現在の組織内で利用する[ロール](/docs/common/ロール（役割）.md)について、ユーザーへの割り当てをすべて解除（紐づけを削除）する
    - 組織関連ロール（`pca.id.{組織ID}/{ID基盤におけるロール名}`）
      - 例：組織管理者： `pca.id.xxxx-xx-xx-xxxx/admin`
    - サービス区画関連ロール（`{サービス区画}/{サービスにおけるロール名}`）
      - 例：PCA Hub テナント管理者：`pca.hub.tenant1/gs:admin`
- Hub等のサービスにおけるユーザー削除では、このAPIは呼び出さない
  - サービス内のユーザー情報を削除するだけ
  - PCAアカウントはそのままの状態で維持する

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
| なし | | | | |

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |
