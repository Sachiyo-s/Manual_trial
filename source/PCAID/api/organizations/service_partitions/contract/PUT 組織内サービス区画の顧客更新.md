# PUT 組織内サービス区画の顧客更新

```http
/organizations/service_partitions/contract
```

## 説明

指定した[組織](/docs/common/組織.md)、または[サービス区画](/docs/common/サービス区画.md)に対して、顧客ID等の顧客情報を更新します。

- ヘッダーに[組織](/docs/common/組織.md)（X-PCA-organization-id）が指定された場合は、組織に対する顧客情報を更新する
- ヘッダーに[サービス区画](/docs/common/サービス区画.md)（X-PCA-service-partition）が指定された場合は、そのサービス区画に対する顧客情報を更新する

### 注意事項

通常であれば、このAPIを使用する必要はありません。  
[POST 組織内サービス区画追加 API](../POST%20組織内サービス区画追加.md)を使用することで、サービス契約情報から顧客情報が反映されます。  
サービス契約情報から顧客情報を反映できないときの緊急措置として、このAPIを利用することを想定しています。

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
| external_customer_id | 顧客ID | required | String | ・社内システムの値 <br/> ・空文字列なら無視する（顧客IDの切り替えを想定） |
| arch_registration_id | Arch登録ID | optional | String | ・社内システムの値 <br/> ・この項目を含めたら空文字列でも上書きする（Archプランの取り消しを想定） |

```json
{
  "external_customer_id": "1111111",
  "arch_registration_id": "A123456"
}
```

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |
