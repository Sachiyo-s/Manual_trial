# POST 組織削除

```http
/organizations/reset
```

## 説明

指定した[組織](/docs/common/組織.md)を、所属ユーザーを含めて削除します。

- 顧客システムから呼び出されることを想定する
  - 組織の顧客ID（external_customer_id）は、削除更新により設定される
    - [組織更新API PUT /organizations](./../PUT%20組織更新.md)
    - 顧客システムが組織の作成イベントを処理することを想定する
  - 顧客システムが自身の顧客レコードを削除する時に、事前に組織を削除することを想定する
- 指定した顧客IDが、登録済み顧客ID（external_customer_id）と一致していなければエラーとする
  - 組織の顧客IDが未登録ならチェックを省略する
    - 顧客が利用していない組織（例：テスト用）を意味しているので問題なく削除できる
- 複数の組織に所属しているユーザーは、削除する組織への所属をクリアし、ユーザー削除しない
  - ただし、削除するサービス区画やロールについて、ユーザーから紐づけを解除する
- 対象組織で利用を可能としているサービス区画を削除する
  - 削除するサービス区画についてユーザーへの割り当てを解除する
- 削除するサービス区画内で利用するすべてのロールを削除する
  - 削除するロールについてユーザーへの割り当てを解除する
- Hubテナント移行状態を削除する
  - [Hubテナント移行状態取得API GET /hub_authn_switchings](./../../hub_authn_switchings/GET%20Hubテナント移行状態取得.md)
- 削除後の[組織名](/docs/common/組織名.md)は再利用可能とする
  - 過去に組織名の利用を記録した履歴情報は削除する
- 対象組織が削除済みなら何もしない
- 処理時間が残り10秒を切ったら `408 Request Timeout` エラーとする
- `408 Request Timeout` エラーの状況でも、APIを繰り返し呼び出すことで、最終的には成功するようにする
  - ユーザーを１人ずつ（可能なかぎり）アトミックに削除する
  - サービス区画や組織の削除は、すべてのユーザーの削除後に実行する

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | [OAuthアクセストークン](/docs/common-dev/oidc-oauth-token-rules.md) | required | String | [管理APIの認可方法](/docs/common-dev/pcaid-management-api-granttype.md) |
| X-PCA-organization-id | 組織ID | required | String | ・任意の組織を指定して利用されることを想定する |

```http
Authorization: Bearer xxxx-xxxx-xxxx-xxxx
```

```http
X-PCA-organization-id: xxxx-xx-xx-xxxx
```

### BODY

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| customer_id | 顧客ID | required | String | ・ダブルチェックを目的として、顧客システムで管理している削除対象の顧客IDも要求する<br/>・削除する組織に顧客ID（external_customer_id）が未登録ならチェックは省略し、指定した値は無視する |

```json
{
  "customer_id": "12345678",
}
```

### 顧客IDと紐づけていない組織を削除する場合

- 以下のように任意のダミー値（例："empty"）を指定する
- `customer_id` は必須項目としているため、無指定（`""` や `null`）はエラーとなる

```json
{
  "customer_id": "empty",
}
```

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |
