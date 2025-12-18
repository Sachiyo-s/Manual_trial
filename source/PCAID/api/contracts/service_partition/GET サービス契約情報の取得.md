---
sidebar_position: 1
---

# GET サービス契約情報の取得

```http
/contracts/{service_partition}
```

## 説明

指定した[サービス区画](/docs/common/サービス区画.md)についてのサービス契約情報を取得します。  
サービス契約情報を取得するにはアクセス対象の組織で追加済みサービスであることが必要です。

- サービス契約情報は、[サービス契約データの扱い](/docs/common-dev/service-contract-info.md) に従い、キャッシュデータを返す
- サービス契約の元データを管理するサーバーがダウンしても、キャッシュが自動削除されるまではリクエストに応答する

## 認可条件

- アカウントID（sub）
- スコープ
  - `id.contracts.read`
- [ロール（役割）](/docs/common/ロール（役割）.md)
  - 一般ユーザー `pca.id.{organization_id}/user` ロールを必要とする
  - 呼び出し元の[サービス区画](/docs/common/サービス区画.md)の指定を求める
    - 例１：`pca.arch.xxxx-xxx-xxx-xxxx`
    - 例２：`pca.cloud.org-abcd-1234`
- 所属組織（orgs）
  - 取得対象の[サービス区画](/docs/common/サービス区画.md)の指定を求める

## 取得対象サービス

- PCAクラウド（`pca.cloud.xxxxx`）
- PCAサブスク（`pca.subsc.xxxxx`）

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | [OAuthアクセストークン](/docs/common-dev/oidc-oauth-token-rules.md) | required | String | ・認可コードフローが必要 |
| X-PCA-service-partition | [サービス区画](/docs/common/サービス区画.md) | conditional | String |・呼び出し元のサービス区画を指定する |

```http
Authorization: Bearer xxxx-xxxx-xxxx-xxxx
```

```http
X-PCA-service-partition: pca.arch.xxxx-xxx-xxx-xxxx
```

### BODY

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |

## OUTPUT

### サービス契約情報

| パラメータ | 名前 | データ型 | 値の例 | 備考 |
| --- | --- | --- | --- | --- |
| customer_id | 顧客ID | string | 10009999 | |
| service_partition | サービス区画 | string | pca.cloud.org-abcd-1234 | |
| service_contract_type | サービス種類 | string | pca.cloud | `pca.hub`：PCA Hub <br/> `pca.cloud`：PCAクラウド <br/> `pca.subsc`：PCAサブスク |
| service_contract_id | サービス契約ID | string | 40001234 | |
| service_contract_company | サービス契約会社 | string | イイダバシ株式会社 | |
| service_contract_status | サービス契約状態 | string | valid | `valid`：有効 <br/> `invalid`：無効 |
| cache_expiry | キャッシュ期限 | integer | 1753612542 | |
| cache_updated_at | キャッシュ更新日時 | string | 2024-11-01T10:20:00.123ZS | |

#### 製品ライセンス情報（app_licenses）

| パラメータ | 名前 | データ型 | 値の例 | 備考 |
| --- | --- | --- | --- | --- |
| app_id | アプリID | string | Acc20 <br/> SPay20 | 製品を特定するID |
| app_edition_id | アプリエディションID | string | Acc20 <br/> SPay20SQL | 製品エディションを特定するID |
| app_name | アプリ名 | string | PCAクラウド 会計 dx <br/> PCAサブスク 給与 hyper | |
| app_usage_status | アプリ利用状態 | string | valid | `valid`：有効 <br/> `invalid`：無効 |
| app_usage_cal | アプリ利用CAL | integer | 3 <br/> NULL | 未設定なら `NULL` |
| app_usage_period_start | アプリ利用開始日 | string | 2024-04-01 | 未設定なら `0001-01-01` |
| app_usage_period_end  | アプリ利用終了日 | string | 2024-03-31 <br/> 0001-01-01 | 未設定なら `0001-01-01` |

#### 製品サポート情報（app_supports）

| パラメータ | 名前 | データ型 | 値の例 | 備考 |
| --- | --- | --- | --- | --- |
| app_id | アプリID | string | Acc20 <br/> SPay20 | app_licenses 配列の app_id と紐づく |
| app_registerno | 登録番号 | integer | 21607285 | |
| app_serialno | 製造番号 | string | 1780 | |
| app_category1 | 大分類 | integer | 133 | 202：PCA Hub <br/> 133：PCAクラウド <br/> 199：PCAサブスク |
| app_category2 | 小分類 | integer | 1437 | |
| app_plan_name | 製品プラン | string | PCAクラウド 会計 dx 月額プラン <br/> PCAサブスク 給与 hyper 年額プラン | |
| app_status_name | 契約種別 | string | 月額利用中 <br/> 年額利用中 | |
| app_support_period_start | 契約開始日 | string | 2025-04-01 | 未設定なら `0001-01-01` |
| app_support_period_end | 契約終了日 | string | 2026-03-31 | 未設定なら `0001-01-01` |

```json
{
  "customer_id": "c12345678", 
  "service_partition": "pca.cloud.org-abcd-1234",
  "service_contract_type": "pca.cloud",
  "service_contract_id": "12345",
  "service_contract_company": "イイダバシ株式会社",
  "service_contract_status": "valid",
  "cache_expiry": 1753612542,
  "cache_updated_at": "2024-11-01T10:20:00.123ZS",
  "app_licenses": [
    {
      "app_id": "Acc20",
      "app_edition_id": "Acc20",
      "app_name": "PCAクラウド 会計 dx",
      "app_usage_status": "valid", 
      "app_usage_cal": 10, 
      "app_usage_period_start": "2024-04-01",
      "app_usage_period_end": "2025-03-31",
    },
    {
      "app_id": "SPay20",
      "app_edition_id": "SPay20",
      "app_name": "PCAクラウド 給与 hyper",
      "app_usage_status": "valid", 
      "app_usage_cal": 3, 
      "app_usage_period_start": "2020-04-01",
      "app_usage_period_end": "0001-01-01",
    },
  ],
  "app_supports": [
    {
      "app_id": "Acc20",
      "app_registerno": 22000111,
      "app_serialno": "12345678",
      "app_category1": 133,
      "app_category2": 1437,
      "app_plan_name": "PCAｸﾗｳﾄﾞ 会計dx 買取ﾌﾟﾗﾝ",
      "app_status_name": "年額利用中",
      "app_support_period_start": "2015-04-01",
      "app_support_period_end": "2026-03-31",
    },
    {
      "app_id": "Acc20",
      "app_registerno": 22000222,
      "app_serialno": "23456789",
      "app_category1": 133,
      "app_category2": 1439,
      "app_plan_name": "PCAｸﾗｳﾄﾞ ｿﾌﾄ利用ﾗｲｾﾝｽ会計dx",
      "app_status_name": "月額利用中",
      "app_support_period_start": "2015-04-01",
      "app_support_period_end": "2026-03-31",
    },
    {
      "app_id": "SPay20",
      "app_registerno": 22000333,
      "app_serialno": "34567890",
      "app_category1": 133,
      "app_category2": 1452,
      "app_plan_name": "PCAｸﾗｳﾄﾞ 給与dx 買取ﾌﾟﾗﾝ",
      "app_status_name": "年額利用中",
      "app_support_period_start": "2015-04-01",
      "app_support_period_end": "2026-03-31",
    },
  ],
}
```
