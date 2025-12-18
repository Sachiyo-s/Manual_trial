---
sidebar_position: 21
---

# 21. PCA ID 連携クライアント

## PCA ID 連携用のAPIエンドポイント

PCA IDを利用する外部クライアント向けのAPIとなる。
外部クライアントは、PCAクラウドやクロノス、ドリームホップ等を想定する。

- OIDC/OAuth エンドポイント
  - `https://{pcaid-domain}/auth/v1`
    - 認可エンドポイント
    - トークンエンドポイント
- PCA ID 連携エンドポイント
  - `https://{pcaid-domain}/api/apps/v1`
    - 組織情報の取得
    - ユーザー一覧の取得
    - サービスロールの設定・取得
  - API Gateway Authorizer で認可チェックする
    - クライアントからのアクセストークンをチェックして、管理API用のトークンを後続のLambdaへ発行する

### 関連文書

- [OIDC/OAuth クライアント情報](./oidc-oauth-clients.md)

## PCA ID 連携クライアントへの要求事項

- 認可コードグラント（認可コードフロー）
  - パブリッククライアント
  - コンフィデンシャルクライアント
    - クライアント認証は ClientSecret を使用する
- PKCE必須
- リフレッシュトークンを保存する場合、スコープにオフラインアクセス（offline_access）を指定する

## OIDC/OAuth スコープ

- クライアントはスコープを明示する必要がある
  - 自動的に適用されるスコープはない
- 実行ユーザー（アクセス許可したユーザー）に紐づくロールに応じて、アクセス可能な対象ユーザーの範囲が決まる
  - 組織管理者ロールを保持していると、組織内のすべてのユーザーにアクセスできる
  - サービス責任者ロールを保持していると、対象のサービス区画に紐づくユーザーにアクセスできる

| スコープ                | 対応API | [ロール要件](/docs/common/ロール（役割）.md) |
| ---                    | --- | --- |
| openid                 | [IDトークン、/auth/v1/userinfo](./oidc-oauth-token-rules.md) | なし |
|                        | [APIリクエスト条件確認 /ensure](/docs/api/ensure/POST%20APIリクエスト条件確認.md) | 一般ユーザー |
| offline_access         | オフライントークン | なし |
| email                  | メールアドレス | なし |
| profile                | プロフィール（姓名） | なし |
| id.me                  | [ユーザー情報取得 GET /me](/docs/api/me/GET%20ユーザー情報取得.md) | 一般ユーザー |
|                        | [プロフィール画像 GET /me/images/profile](/docs/api/users/account_id/GET%20ユーザー取得.md) | 　　〃 |
| id.organization.read   | [組織取得 GET /organizations](/docs/api/organizations/GET%20組織取得.md) | 　　〃 |
|                        | [組織画像取得 GET /organizations/images/logo](/docs/api/organizations/images/logo/GET%20組織画像取得.md) | 　　〃 |
| id.users.read          | [ユーザー一覧取得 GET /users](/docs/api/users/GET%20ユーザー一覧取得.md) | ロールごとに取得可能ユーザーを以下とし、実行ユーザーに紐づくより強いロールを採用する <br/><br/> ＜組織責任者＞：組織内ユーザー <br/> ＜サービス責任者＞：サービス区画内ユーザー <br/><br/> いずれのロールであっても、対象ユーザーは、APIで指定するサービス区画でフィルターされる |
|                        | [ユーザー一覧検索 GET /users/query](https://github.com/pca-idp/pcaid/issues/134) | 　　〃 |
|                        | [ユーザー取得 GET /users/{account_id}](/docs/api/users/account_id/GET%20ユーザー取得.md) | 　　〃 |
|                        | [プロフィール画像取得 GET /users/{account_id}/images/profile](/docs/api/users/account_id/images/profile/GET%20プロフィール画像取得.md) | 　　〃 |
| id.service_roles.write | [ユーザーロールの割り当て POST /users/{account_id}/service_roles](/docs/api/users/account_id/service_roles/POST%20ユーザーロールの割り当て.md) | 　　〃 |
|                        | [ユーザーロールの割り当て解除 POST /users/{account_id}/service_roles/unassign](/docs/api/users/account_id/service_roles/unassign/POST%20ユーザーロールの割り当て解除.md) | 　　〃 |
| id.contracts.read      | [サービス契約情報の取得 GET /contracts/{service_partition}](/docs/api/contracts/service_partition/GET%20サービス契約情報の取得.md) | ・一般ユーザー <br/>・組織に追加済みのサービス |
| id.customer_ids.read   | [製品ダウンロード可能な顧客ID一覧取得 GET /contracts/download_customer_ids](/docs/api/contracts/download_customer_ids/GET%20製品ダウンロード可能な顧客ID一覧取得.md) | ・組織ごとの製品ダウンロード許可 |

## OAuth トークン変換（OAuth 2.0 Token Exchange）

- PCA IDトークンからPCA Hubトークンを取得できるようにする
  - PCA ID サブジェクトを確認して、PCA Hub アクセストークンを発行する
  - 主に PCA Hub 側の対応が必要になる
    - PCA ID だけでなく、PCA Hub にもクライアント登録が必要となる
- PCA Hubトークンエンドポイントを利用する
  - `https://{tenant_name}.pcahub.jp/auth/v1/token`

### 参考文書

- [RFC 8693 OAuth 2\.0 Token Exchange \- Authlete](https://www.authlete.com/ja/developers/token_exchange/)
- [RFC 8693 OAuth 2\.0 Token Exchange \#OAuth \- Qiita](https://qiita.com/TakahikoKawasaki/items/d9be1b509ade87c337f2)
