# GET ユーザー情報取得

```http
/me
```

## 説明

自身のユーザー情報の取得します。

## 認可条件

- アカウントID（sub）
- [ロール（役割）](/docs/common/ロール（役割）.md)
  - 一般ユーザー `pca.id.{organization_id}/user` ロールを必要とする
  - アプリ向けAPI `/api/apps/v1` では、他のAPIに合わせて、呼び出し元の[サービス区画](/docs/common/サービス区画.md)の指定を求める
  - 外部に非公開の `/api/account/v1` では、指定不要で呼び出せるが、セキュリティ上の問題はない
- イントロスペクションAPIで、現在の認可状態が有効なことを確認する

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | [OAuthアクセストークン](/docs/common-dev/oidc-oauth-token-rules.md) | required | String | 認可コードフローが必要 |
| X-PCA-service-partition | [サービス区画](/docs/common/サービス区画.md) | conditional | String |・`/api/apps/v1` では必須 <br/>・呼び出し元のサービス区画を指定する |

```http
Authorization: Bearer xxxx-xxxx-xxxx-xxxx
```

```http
X-PCA-service-partition: pca.cloud.org-xxx-xxx
```

### QUERY PARAMETER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| org-sp | 組織のサービス区画の扱い（org_service_partitions の値） | optional | String | ・組織に紐づくすべてのサービス区画を含める：`all` <br/>・ユーザーに紐づくサービス区画のみを含める：`allowed`（デフォルト） |

※パラメーターが未指定であってもデフォルト値を適用する

### BODY

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| account_id | アカウントID | requried | String | |
| account_status | アカウント状態 | required | String |  ・有効：`active` <br/>・無効：`inactive` |
| lockout_status | ロックアウト状態 | required | String | ・通常（未ロックアウト）：`active` <br/>・ロックアウト中：`locked` |
| email | メールアドレス | requried | String | ・[PCAアカウント](/docs/common/PCAアカウント.md) の「メールアドレス」を参照 |
| email_status | メールアドレス状態 | requried | String | ・確認済みなら `enable` それ以外は `unable` |
| preferred_username | 表示名 | requried | String | |
| family_name | 姓 | requried | String | |
| given_name | 名 | requried | String | 値が空文字列の場合あり |
| family_kana | 姓カナ | requried | String | |
| given_kana | 名カナ | requried | String | 値が空文字列の場合あり |
| in_organizations | 所属組織 | required | Object Array | ・ロールが指す組織が対象となる |
| - organization_id | 組織ID | required | String |・[組織](/docs/common/組織.md)の「組織ID」を参照 |
| - organization_name | 組織名 | required | String | |
| - organization_display_name | 組織表示名 | required | String |・[組織](/docs/common/組織.md)の「組織表示名」を参照 |
| - org_service_partitions | 組織が利用可能な[サービス区画](/docs/common/サービス区画.md) | required | String Array | `org-sp` クエリーパラメーターを参照 |
| - external_customer_id | 顧客ID（外部管理） | required | String | ・値が空文字列の場合あり <br/> ・値が複数の場合あり（スペース区切り） |
| - arch_status | Archサービス有効状態 | required | String | ・有効なら `valid`、無効なら `invalid`  |
| - is_admin | 組織管理者フラグ | requried | Boolean | 組織管理者なら `true`、それ以外は `false` |
| - arch_plan | PCA Arch プラン情報 | optional | Object | ・Arch プランを含む組織のみ |
| login_names | 組織ごとのログイン名 | requried | String Array | ・[PCAアカウント](/docs/common/PCAアカウント.md) の「ログイン名」を参照 |
| user_service_partitions | ユーザーが利用可能な[サービス区画](/docs/common/サービス区画.md) | required | String Array | |

### PCA Arch プラン情報（arch_plan オブジェクト）

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| arch_service_partitions | 組織が利用可能なArchプランの[サービス区画](/docs/common/サービス区画.md) | required | String Array | ・親項目 `org_service_partitions` のサブセット |
| arch_registration_id | Arch登録ID（外部管理） | required | String | ・組織内に１つのみ |
| arch_customer_id | 顧客ID（外部管理） | required | String | ・Archプランの顧客IDは組織内に１つのみ <br/>・親項目 `external_customer_id` のサブセット |

```json
{
  "account_id": "id-xx-xx-1234",
  "account_status": "active",
  "lockout_status": "active",
  "email": "yamada@email.com",
  "email_verified": true,
  "preferred_username": "総務部_山田太郎",
  "family_name": "山田",
  "given_name": "太郎",
  "family_kana": "ヤマダ",
  "given_kana": "タロウ",
  "in_organizations": [
    {
      "organization_id": "xxxxx-xxx-xxx-xxxxx",
      "organization_name": "pca",
      "organization_display_name": "ピー・シー・エー株式会社",
      "org_service_partitions": [
        "pca.hub.pca", 
        "pca.cloud.xxx-12345", 
      ],
      "external_customer_id": "12345678",
      "arch_status": "valid",
      "is_admin": true, 
      "arch_plan": {
        "arch_service_partitions": [
          "pca.hub.pca", 
          "pca.cloud.xxx-12345", 
        ],
        "arch_registration_id": "999999",
        "arch_customer_id": "12345678",
      },
    },
    {
      "organization_id": "xxxxx-xxx-xxx-xxxxx",
      "organization_name": "xronos",
      "organization_display_name": "クロノス株式会社",
      "org_service_partitions": [
        "pca.hub.xronos", 
        "pca.subsc.xxx-55555", 
      ],
      "external_customer_id": "23456789",
      "arch_status": "invalid",
      "is_admin": false, 
    },
    {
      "organization_id": "xxxxx-xxx-xxx-xxxxx",
      "organization_name": "org-xxxx-1234",
      "organization_display_name": "ドリームホップ株式会社",
      "org_service_partitions": [
        "pca.cloud.xxxx-6666", 
      ],
      "external_customer_id": "23456789",
      "arch_status": "invalid",
      "is_admin": false, 
    },
  ],
  "login_names": [
    "pca\\yamada",
    "xronos\\yamada.tarou",
    "org-xxxx-1234\\yamada@email.com",
  ],
  "user_service_partitions": [
    "pca.hub.pca", 
    "pca.hub.xronos", 
    "pca.cloud.xxx-12345", 
    "pca.cloud.xxxx-6666", 
  ],
}
```
