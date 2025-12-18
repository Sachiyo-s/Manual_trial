---
sidebar_position: 1
---

# GET 製品ダウンロード可能な顧客ID一覧取得

```http
/contracts/download_customer_ids
```

## 説明

PCAクラウド等の製品ダウンロードが許可されている所属組織の顧客ID一覧を取得します。

- 自身に対して製品ダウンロード許可ロールが付与されている所属組織を特定する
- 上記組織に追加されている、特定のサービス区画（PCAクラウド／PCAサブスク）の顧客IDを返す

## 認可条件

- アカウントID（sub）
- [スコープ](/docs/common-dev/apps-integration-clients.md#oidcoauth-スコープ)
  - `id.customer_ids.read`
- [ロール（役割）](/docs/common/ロール（役割）.md)
  - 製品ダウンロード許可をあらわす `pca.id.{organization_id}/pcaapp20-download` ロールを必要とする

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | [OAuthアクセストークン](/docs/common-dev/oidc-oauth-token-rules.md) | required | String | ・認可コードフローが必要 |
| X-PCA-service-partition | [サービス区画](/docs/common/サービス区画.md) | conditional | String | `pca.support.customerid` 固定 |

```http
Authorization: Bearer xxxx-xxxx-xxxx-xxxx
```

```http
X-PCA-service-partition: pca.support.customerid
```

### BODY

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |

## OUTPUT

### サービス契約情報

| パラメータ | 名前 | データ型 | 値の例 | 備考 |
| --- | --- | --- | --- | --- |
| customer_ids | 顧客ID一覧 | String配列 | 10009999 | ・顧客IDが不明の場合や、ダウンロード許可がなければ空配列 |

```json
{
  "customer_ids": ["12345678", "23456789"], 
}
```
