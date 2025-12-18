---
sidebar_position: 1
---

# POST 組織作成準備

```http
/organizations/prepare
```

## 説明

新しい組織の作成に必要となる初期情報を準備します。  
PCAクラウド/PCAサブスクから呼び出されることを想定します。

- 設定情報は有効期間は 1時間 とし、期限を過ぎたら自動的に削除する
  - 初期情報の保存先は DynamoDB を想定する

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | ワンタイムパスワード（OTP） | required | String | ・[OTP認証](/docs/common-dev/otp-authentication.md) を使用する <br/>・認証タイプとして `Totp` プレフィックスを付加する |

```http
Authorization: Totp eac38d043df30f177d7bb1ee2c7f20a039c5959637810cc48ba02686dd1f0bbd
```

### BODY

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

#### 必須以外の情報を省略した場合

```json
{
    "client_id": "pcaapp20-ui",
}
```

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| receipt_session_id | 受付セッションID | required | String | ・準備済みの初期情報を指すユニークなGUID値とする <br/> ・（可能性は低いが）準備済みのGUID値と衝突する限り、繰り返し発番する |

```json
{
    "receipt_session_id": "xxxx-xxx-xxx-xxxx",
}
```
