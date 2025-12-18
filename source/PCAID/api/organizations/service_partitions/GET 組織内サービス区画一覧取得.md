---
sidebar_position: 2
---

# GET 組織内サービス区画一覧取得

```http
/organizations/service_partitions
```

## 説明

指定した組織ID、または呼び出し元の[サービス区画](/docs/common/サービス区画.md)に応じた[組織](/docs/common/組織.md)に紐づく、サービス区画の一覧を取得します。

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
| organization_id <br/> (service_partition_id) | サービス区画ID | requried | String | Keycloakグループをあらわすuuid値 <br/> ※今後カッコ内の名前に変更する想定 |
| organization_name <br/> (service_partition) | [サービス区画](/docs/common/サービス区画.md) | requried | String |  |
| permitted <br/> (everyone_permitted) | ユーザー利用許可フラグ | requried | Boolean | 組織内のユーザーにサービス利用をどのように許可するかの扱い <br/> ・すべてのユーザーに許可する：`true` <br/> ・管理者（責任者）が選択したユーザーに許可する（デフォルト）：`false` |
| contract_id | 契約ID | requried | String | 社内システムの値 |
| arch_registration_id | Arch登録ID | requried | String | 社内システムの値 |
| customer_id | 顧客ID | requried | String | 社内システムの値 |

```json
{
  "organization_id": "xxxx-xx-xx-xxxx",
  "organization_name": "pca.cloud.org-xxxx-xxxx",
  "permitted": false,
  "contract_id": "10123456",
  "arch_registration_id": "A123456",
  "customer_id": "8888888"
}
```

現在はプロパティ名がその意味と一致していないので、今後、以下のように変更する想定  
情シスには社内システム連携のために共有済みのデータ構造なので、後方互換性に注意する必要がある

```json
{
  "service_partition_id": "xxxx-xx-xx-xxxx",
  "service_partition": "pca.cloud.org-xxxx-xxxx",
  "everyone_permitted": false,
  "contract_id": "10123456",
  "arch_registration_id": "A123456",
  "customer_id": "8888888"
}
```
