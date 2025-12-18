# POST 連携コード発行

```http
/connect_code
```

## 説明

サービス区画に必要となる連携コードを発行します。  
組織作成や組織管理において、PCAクラウド等の利用サービスを追加する際に、呼び出されることを想定します。  
また、モダン化後のPCAクラウドにおいて、契約ID（サービスユーザーID）の区画ごとに、HTTPサブドメインとして使用する目的で、連携コードを発行・取得することを想定します。  

- サービス種類ごとの契約IDと、連携コードとの対応関係は、無期限で管理する
  - 設定情報の保存先は DynamoDB を想定する
- 連携コードを発行済みの契約IDをリクエストされたら、発番せずに、以前の連携コードを返す（冪等にする）
- 契約IDの存在はチェックしない
  - 任意の利用サービスにおける契約IDは、外部システムとなる可能性がある
    - （例）ドリームホップ、クロノス

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | ワンタイムパスワード <br/>（OTP） | conditional | String | ・項目設定は必須（Terraform仕様のため） <br/>・受付セッションID（X-PCA-receipt-session-id）が未指定なら値が必須 <br/>・[OTP認証](/docs/common-dev/otp-authentication.md) を使用する <br/> ・主にバックエンドからの利用を想定する |
| X-PCA-receipt-session-id | 受付セッションID <br/>（時限式） | conditional | String | ・OTP（Authorization）の値が未設定なら必須（両指定ならこちらを優先） <br/>・準備APIで発行したセッションIDで認証する <br/>・主にフロントエンドからの利用を想定する |

```http
Authorization: Totp eac38d043df30f177d7bb1ee2c7f20a039c5959637810cc48ba02686dd1f0bbd
```

```http
X-PCA-receipt-session-id: xxxx-xxx-xxx-xxxx
```

### BODY

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| client_id | クライアントID | requried | String | [OIDC/OAuthクライアント情報](/docs/common-dev/oidc-oauth-clients.md) |
| service_kind | サービス種類 | requried | String | サービス区画のプレフィックスとして定義する |
| service_contract_id | 契約ID | requried | String | PCAクラウド/PCAサブスクの契約（区画）を識別する  |

```json
{
    "client_id": "pcaapp20-ui",
    "service_kind": "pca.cloud",
    "service_contract_id": "12345678",
}
```

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| connect_code | 連携コード | required | String | ・`org-` から始まる「org-xxxx-xxxx」形式のランダムな値とする |

```json
{
    "connect_code": "org-xxxx-xxxx",
}
```
