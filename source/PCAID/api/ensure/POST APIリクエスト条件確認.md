---
sidebar_position: 1
---

# POST APIリクエスト条件確認

```http
/ensure
```

## 説明

認可チェックに PCA ID を利用する外部のAPIエンドポイントに対してアクセスがあったときに、そのリクエストが必要な条件を満たしているかを確認します。  
条件を満たすリクエストには、アクセストークン、およびリクエストを許可したクライアントやスコープ、ユーザーについての情報を返します。

- リクエストボディで受け取ったアクセストークンに対して、以下をチェックする
  - トークンを許可したユーザーが、ヘッダーに指定された[サービス区画](/docs/common/サービス区画.md)に割り当てられている
    - ユーザーは対象サービスの利用を許可されている必要がある
  - トークンの許可スコープに、APIエンドポイントの要求スコープがすべて含まれている
    - 例：ArchポータルAPIの呼び出しには、`arch.portal.basic` スコープが許可されている必要がある

## 認可条件

- アカウントID（sub）
- スコープ
  - `openid`
- [ロール（役割）](/docs/common/ロール（役割）.md)
  - APIサーバーをあらわす `pca-api-services` ロールを必要とする

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | [OAuthアクセストークン](/docs/common-dev/oidc-oauth-token-rules.md) | required | String | ・リソースサーバーとしてのトークン <br/>・クライアントクレデンシャルフローが必要 |
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
| access_token | アクセストークン | requried | String | APIエンドポイントがクライアントから受け取るトークン |
| required_scopes | 要求スコープ一覧 | optional | String配列 | APIエンドポイントがクライアントに要求するスコープの一覧 |

```json
{
  "access_token": "xxxx-xxxx-xxxx-xxxx",
  "required_scopes": [ "id.contracts.read", "arch.portal.basic" ],
}
```

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| active | アクセストークンの有効状態 | required | Boolean | true: トークンは利用可能 <br/> false: トークンは利用不可 |
| exp | アクセストークンの失効日時 | required | Number | UNIX時間 |
| iat | アクセストークンの発行日時 | required | Number | UNIX時間 |
| jti | アクセストークンの識別ID | required | String | |
| iss | 発行元ID管理サービス（PCA ID） | required | String | <https://{pcaid-domain-name}/realms/pcaid> |
| sub | サブジェクト（アカウントID） | required | String | クライアントにアクセスを許可したユーザー |
| scope | 許可スコープ | required | String | ユーザーがクライアントに許可した機能・動作 |
| client_id | 許可クライアントID | required | String | ユーザーがAPIアクセスを許可したクライアント |
| organization_id | アクセス先の組織ID | required | String | GUID値 |
| organization_name | アクセス先の組織名 | required | String | |
| customer_id | アクセス先組織の顧客ID | required | String | 契約済み：数字のみ（2,147,483,647 まで） <br/> 体験利用（仮顧客）：A + 6桁数字(100000以降) |

※ OAuthやIDトークン標準の定義と同じ名前の項目は、標準のクレーム（Claim）仕様に準拠しています。

```json
{
  "active": true,
  "exp": 1754386437,
  "iat": 1754382837,
  "jti": "onrtac:16121285-1e53-a383-eb63-fb69b5a2d7cf",
  "iss": "https://poc.id.pca-dev.net/realms/pcaid",
  "sub": "cb8a7e9b-xxxx-xxxx-xxxx-1d6e8dc26894",
  "scope": "group_membership openid",
  "client_id": "pca-portal-ui",
  "organization_id": "04472e89-xxxx-xxxx-xxxx-8df5bba370be",
  "organization_name": "tenant1",
  "customer_id": "12345678"
}
```
