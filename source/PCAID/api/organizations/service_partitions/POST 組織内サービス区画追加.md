---
sidebar_position: 1
---

# POST 組織内サービス区画追加

```http
/organizations/service_partitions
```

## 説明

指定した[組織](/docs/common/組織.md)に対して、利用を可能とする[サービス区画](/docs/common/サービス区画.md)と、そのサービス区画内で利用する任意の[ロール](/docs/common/ロール（役割）.md)を追加します。

- 上流サービスである契約・ライセンス管理システムから呼び出されることを想定する
- 同じ[サービス区画](/docs/common/サービス区画.md)が追加済みなら、繰り返し追加しない
- 同じ[ロール](/docs/common/ロール（役割）.md)が追加済みなら、繰り返し追加しない
- サービス契約情報の[キャッシュデータ](/docs/common-dev/service-contract-info.md#%E3%82%AD%E3%83%A3%E3%83%83%E3%82%B7%E3%83%A5%E3%83%87%E3%83%BC%E3%82%BF)から、[サービス区画](/docs/common/サービス区画.md)の顧客情報を反映する

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | [OAuthアクセストークン](/docs/common-dev/oidc-oauth-token-rules.md) | required | String | [管理APIの認可方法](/docs/common-dev/pcaid-management-api-granttype.md) |
| X-PCA-organization-id | 組織ID | required | String |・任意の組織を指定して利用されることを想定する |

```http
Authorization: Bearer xxxx-xxxx-xxxx-xxxx
```

```http
X-PCA-organization-id: xxxx-xx-xx-xxxx
```

### BODY

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| service_partition | [サービス区画](/docs/common/サービス区画.md) | required | String | ・原則として、利用を可能とする呼び出し元のサービスの区画（例：Hubならテナント名）を指定する |
| service_roles | サービスロール一覧 | optional | String | ・指定されたサービス区画で利用可能なロールとして作成する（service_partitionと結合する） <br/>・例：`pca.hub.tdi/gs:admin` |

```json
{
  "service_partition": "pca.hub.tdi",
  "service_roles": [
    "gs:admin", 
    "d:users"
  ]
}
```

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |
