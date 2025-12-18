---
sidebar_position: 2
---

# GET 組織作成準備

```http
/organizations/prepare
```

## 説明

新しい組織の作成のために準備された初期情報を取得します。  
PCAクラウド/PCAサブスクから事前準備されることを想定します。

- 以下の準備APIによって初期情報を準備する
  - [組織作成準備 POST /organizations/prepare](./POST%20組織作成準備.md)
- 初期情報が取得できなければ `404 Not Found` を返す

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| X-PCA-receipt-session-id | 受付セッションID | required | String | ・準備APIで発行した時限式のセッションIDで認証する |

```http
X-PCA-receipt-session-id: xxxx-xxx-xxx-xxxx
```

### BODY

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| client_id | クライアントID | requried | String | [OIDC/OAuthクライアント情報](/docs/common-dev/oidc-oauth-clients.md) |
| service_kind | サービス種類 | optional | String | サービス区画のプレフィックスとして定義する |
| service_contract_id | 契約ID | optional | String | PCAクラウド/PCAサブスクの契約（区画）を識別する  |
| organization_name <br/>（現在未使用） | 組織名 | optional | String | 組織名は自動生成するので指定値は無視する |
| organization_display_name | 組織表示名 | optional | String | |
| admin_email | 組織管理者（最初）のメールアドレス | optional | String | |
| admin_login_name | 組織管理者（最初）のログイン名 | optional | String | |
| admin_preferred_username | 組織管理者（最初）のユーザー名 | optional | String | |
| admin_family_name | 組織管理者（最初）の姓 | optional | String | |
| admin_given_name | 組織管理者（最初）の名 | optional | String | |
| admin_family_kana | 組織管理者（最初）の姓カナ | optional | String | |
| admin_given_kana | 組織管理者（最初）の名カナ | optional | String | |

```json
{
    "client_id": "pcaapp20-ui",
    "service_kind": "pca.cloud",
    "service_contract_id": 12345678,
    "organization_name": "iidabashi",
    "organization_display_name": "イイダバシ株式会社",
    "admin_email": "yamada@pca.co.jp",
    "admin_login_name": "ichiro",
    "admin_preferred_username": "飯田橋 一郎",
    "admin_family_name": "飯田橋",
    "admin_given_name": "一郎",
    "admin_family_kana": "イイダバシ",
    "admin_given_kana": "イチロウ",
}
```
