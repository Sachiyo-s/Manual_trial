---
sidebar_position: 3
---

# PUT 組織更新

## 説明

指定した組織ID、または呼び出し元の[サービス区画](/docs/common/サービス区画.md)に応じた[組織](/docs/common/組織.md)の情報を更新します。

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
| organization_display_name | 組織表示名 | requried | String | ・上書きする <br/>・[組織](/docs/common/組織.md)の「組織表示名」を参照 |
| external_customer_id | 顧客ID（外部管理）| optional | String | ・上書きする |
| arch_registration_id | Arch登録ID（外部管理）| optional | String | ・上書きする |

```json
{
  "organization_display_name": "○○株式会社",
  "external_customer_id": "123456",
  "arch_registration_id": "9999"
}
```

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |
