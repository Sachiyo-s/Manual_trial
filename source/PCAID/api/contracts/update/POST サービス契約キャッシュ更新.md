---
sidebar_position: 1
---

# POST サービス契約キャッシュ更新

```http
/contracts/update
```

## 説明

指定したサービス契約IDの識別情報に基づいて、サービス契約情報のキャッシュデータを更新します。  
キャッシュのオリジンとなるサービス契約情報は、それぞれの[別AWSアカウント](/docs/common-dev/pcaid-external-accounts-list.md)のサービス管理や顧客管理に[DBアクセス](/docs/common-dev/pcaid-external-dbservers-access.md)して取得します。

- PCAクラウド（pca.cloud）と PCAサブスク（pca.subsc）は、連携コードから契約IDを特定する
- PCA Hub（pca.hub）は、テナント名を契約IDと見なす
- 現状（2025/11）、それ以外のサービス契約の種類はサポートしていない
- サービス契約情報の[キャッシュデータ](/docs/common-dev/service-contract-info.md#%E3%82%AD%E3%83%A3%E3%83%83%E3%82%B7%E3%83%A5%E3%83%87%E3%83%BC%E3%82%BF)は、DynamoDBに一定期間だけ保存する

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | [OAuthアクセストークン](/docs/common-dev/oidc-oauth-token-rules.md) | required | String | ・認可コードフローが必要 |

```http
Authorization: Bearer xxxx-xxxx-xxxx-xxxx
```

### BODY

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| contract_id | サービス契約IDの識別情報 | required | String | 以下の形式の値を受け付ける <br/> - `pca.hub#{tenant_name}` <br/> - `pca.cloud#{contract_id}` <br/> - `pca.subsc#{contract_id}` <br/> - `pca.hub.{tenant_name}` <br/> - `pca.cloud.{connect_code}` <br/> - `pca.subsc.{connect_code}` <br/> ※ `#` 区切りは契約IDを直接扱う <br/> ※ `.` 区切りはサービス区画として扱う |
| notForceCreateCache | キャッシュしているサービス契約情報を再作成を強制するかどうか | optional | Boolean | 期限切れでなければ作成済みのキャッシュを参照する：`true` <br/> 常にキャッシュを再作成してから参照する：`false` |

```json
{
  "contract_id": "pca.cloud.org-xxxx-xxxx",
  "notForceCreateCache": false 
}
```

## OUTPUT

### サービス契約情報

| パラメータ | 名前 | データ型 | 値の例 | 備考 |
| --- | --- | --- | --- | --- |
| service_partition | [サービス区画](/docs/common/サービス区画.md) | String | pca.cloud.org-xxxx-xxxx |  |
| service_contract_id | サービス契約ID | String | 10123456 |  |
| service_contract_status | サービス契約状態 | String | valid | 有効：`valid` <br/> 無効：`invalid` |
| arch_registration_id | Arch登録ID | String | A123456 | A+6桁数字(100000以降) |
| customer_id | 顧客ID | String | 8888888 |  |
| force_create_cache | 強制的なキャッシュ再作成 | Boolean | false | 強制あり：`true` <br/> 強制なし：`false` |
| end_update_time | キャッシュ更新日時 | Number | 1727159072051 | UNIX時間 |

```json
{
  "service_partition": "pca.cloud.org-xxxx-xxxx", 
  "service_contract_id": "10123456", 
  "service_contract_status": "valid", 
  "arch_registration_id": "A123456", 
  "customer_id": "8888888", 
  "force_create_cache": false, 
  "end_update_time": 1727159072051
}
```
