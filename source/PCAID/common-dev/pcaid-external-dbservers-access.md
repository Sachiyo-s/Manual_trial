---
sidebar_position: 25
---

# 25. 別アカウントDBサーバーアクセス

## 製品ライセンス情報

| パラメータ | 名前 | データ型 | 値の例 | 備考 |
| --- | --- | --- | --- | --- |
| service_contract_id | サービス契約ID | character varying | 40001234 | |
| service_contract_company | サービス契約会社 | character varying | イイダバシ株式会社 | |
| service_contract_status | サービス契約状態 | character varying | valid | `valid`：有効 <br/> `invalid`：無効 |
| app_edition_id | アプリエディションID | character varying | Acc20 <br/> SPay20SQL | |
| app_name | アプリケーション名 | character varying | PCAクラウド 会計 dx <br/> PCAサブスク 給与 hyper | |
| app_usage_status | アプリケーション利用状態 | character varying | valid | `valid`：有効 <br/> `invalid`：無効 |
| app_usage_cal | アプリケーション利用CAL | integer | 3 <br/> NULL | 未設定なら `NULL` |
| app_usage_period_start | アプリケーション利用開始日 | timestamp | 2024-04-01 | 未設定なら `0001-01-01` |
| app_usage_period_end  | アプリケーション利用終了日 | timestamp | 2024-03-31 <br/> 0001-01-01 | 未設定なら `0001-01-01` |

### PCAクラウドSQL

- [DBサーバー接続先情報](./pcaid-external-accounts-list.md#pca%E3%82%AF%E3%83%A9%E3%82%A6%E3%83%89)

```sql
SELECT 
  pusr.PartnerUserID service_contract_id, 
  usr.ServiceUserName service_contract_company, 
  CASE usr.ServiceValid 
    WHEN 1 THEN CAST ('valid' AS character varying) 
    ELSE CAST ('invalid' AS character varying) 
  END service_contract_status, 
  app.AppId app_edition_id, 
  app.AppName app_name, 
  CASE svc.ServiceAppValid 
    WHEN 1 THEN CAST ('valid' AS character varying) 
    ELSE CAST ('invalid' AS character varying) 
  END app_usage_status, 
  CAST (lic.LicenseNum AS integer) app_usage_cal, 
  CASE svc.ServiceStartDate 
    WHEN 0 THEN CAST ('0001-01-01 00:00:00+00' AS timestamp)
    ELSE CAST (CAST (svc.ServiceStartDate AS character varying) AS timestamp)
  END app_usage_period_start, 
  CASE svc.ServiceEndDate 
    WHEN 0 THEN CAST ('0001-01-01 00:00:00+00' AS timestamp)
    ELSE CAST (CAST (svc.ServiceEndDate AS character varying) AS timestamp)
  END app_usage_period_end 
FROM 
  pcacert.PartnerUser pusr
INNER JOIN 
  pcacert.PartnerUserGroup grp ON (grp.PartnerUserGroupID = pusr.GroupID AND grp.PartnerID = pusr.PartnerID) 
INNER JOIN 
  pcacert.ServiceUser usr ON (usr.ServiceUserID = grp.ServiceUserID) 
INNER JOIN 
  pcacert.ContractType contract ON (contract.ContractTypeID = usr.ContractTypeID) 
INNER JOIN 
  pcacert.ServiceApp svc ON (svc.ServiceUserID = usr.ServiceUserID) 
INNER JOIN 
  pcacert.AppInfo app ON (app.AppID = svc.AppID) 
LEFT OUTER JOIN 
  pcacert.License lic ON (lic.AppID = svc.AppID AND lic.LicenseKey = svc.LicenseKey) 
WHERE 
  pusr.PartnerID = 1 AND 
  pusr.PartnerUserID = @契約ID
```

### PCAサブスクSQL

- [DBサーバー接続先情報](./pcaid-external-accounts-list.md#pca%E3%82%B5%E3%83%96%E3%82%B9%E3%82%AF)

```sql
SELECT 
  contract.Id service_contract_id, 
  contract.DisplayName service_contract_company, 
  CASE contract.Status 
    WHEN 'valid' THEN CAST ('valid' AS character varying) 
    ELSE CAST ('invalid' AS character varying) 
  END service_contract_status, 
  prod.ProductKey app_edition_id, 
  prod.AppName app_name, 
  lic.Status app_usage_status, 
  lic.Cal app_usage_cal, 
  lic.UsageStartDate app_usage_period_start, 
  lic.UsageEndDate app_usage_period_end 
FROM 
  Contract contract
INNER JOIN 
  License lic ON (lic.ContractId = contract.Id)
INNER JOIN 
  ContractModel model ON (lic.ContractType = model.ContractType)
INNER JOIN 
  Product prod ON (prod.ProductNo = lic.ProductNo) 
WHERE 
  contract.Id = @契約ID
```

## 顧客契約情報

| パラメータ | 名前 | データ型 | 値の例 | 備考 |
| --- | --- | --- | --- | --- |
| customer_id | 顧客ID | bigint | 10009999 | ・契約済み <br/> ・Arch体験利用（仮顧客） |
| service_type | サービス種類 | character varying | pca.cloud <br/> pca.subsc | `pca.hub`：PCA Hub <br/> `pca.cloud`：PCAクラウド <br/> `pca.subsc`：PCAサブスク |
| service_contract_id | サービス契約ID | character varying | 40001234 | |
| arch_registration_id | Arch登録ID | character varying | 123456 | ・Archプランでのみ有効 <br/> ・それ以外のプランならNULL |

| パラメータ | 名前 | データ型 | 値の例 | 備考 |
| --- | --- | --- | --- | --- |
| customer_id | 顧客ID | integer | 10009999 | 契約済み顧客が対象 <br/> （体験利用では未登録） |
| customer_company | 顧客会社 | character varying | イイダバシ株式会社 | |
| app_registerno | 登録番号 | bigint | 21607285 | |
| app_serialno | 製造番号 | character varying | 1780 | |
| app_category1 | 大分類 | integer | 133 | 202：PCA Hub <br/> 133：PCAクラウド <br/> 199：PCAサブスク |
| app_category2 | 小分類 | integer | 1437 | |
| app_plan_name | 製品プラン | character varying | PCAクラウド 会計 dx 月額プラン <br/> PCAサブスク 給与 hyper 年額プラン | |
| app_status_name | 契約種別 | character varying | 月額利用中 <br/> 年額利用中 | |
| app_support_period_start | 契約開始日 | timestamp | 2025-04-01 | 未設定なら `1900-01-01` |
| app_support_period_end | 契約終了日 | timestamp | 2026-03-31 | 未設定なら `1900-01-01` |

### 顧客契約SQL

- [DBサーバー接続先情報](./pcaid-external-accounts-list.md#%E9%A1%A7%E5%AE%A2db%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC)

```sql
SELECT 
  customer_id, 
  CASE service_type 
    WHEN 1 THEN CAST ('pca.cloud' AS character varying) 
    WHEN 2 THEN CAST ('pca.subsc' AS character varying) 
    WHEN 3 THEN CAST ('pca.hub' AS character varying) 
    ELSE CAST ('pca.unknown' AS character varying) 
  END, 
  service_id service_contract_id, 
  arch_registration_id 
FROM 
  service_info 
```

### 顧客契約詳細SQL

```sql
SELECT 
  cm_cstmcode customer_id,
  cm_name customer_company,
  cp_toutyaku app_registerno,
  cp_serialno app_serialno,
  pc1cate_no app_category1,
  pc2cate_no app_category2,
  pc2cate_name app_plan_name,
  pca_ky_name app_status_name,
  cp_hosyudate_s app_support_period_start,
  cp_hosyudate_e app_support_period_end
FROM cust_prdcts customer_product 
INNER JOIN 
  ct_pca_keiyaku contract ON (contract.pca_ky_no = customer_product.cp_hosyuno AND pca_ky_name LIKE ('%利用中%'))
INNER JOIN 
  cust_mstr customer ON (customer.cm_no = customer_product.cp_cstmno)
INNER JOIN 
  prdct_mstr product ON (product.pm_no = customer_product.cp_prdtno)
INNER JOIN 
  pc2category category2 ON (category2.pc2cate_no = product.pm_pc2no)
INNER JOIN 
  pc1category category1 ON (category1.pc1cate_no = category2.pc2cate_pc1no)
WHERE 
  cm_cstmcode = @顧客ID AND 
  pc1cate_no = 133 --PCAクラウド
--pc1cate_no = 199 --PCAサブスク
--pc1cate_no = 202 --PCA Hub
```
