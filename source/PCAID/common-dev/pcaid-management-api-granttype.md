---
sidebar_position: 12
---

# 12. PCA ID 管理APIにおける認可方法

## 認可方法（アクセス許可チェック）

- OAuthアクセストークンの方式とする
  - Authorization リクエストヘッダーを利用する
  - `Authorization: Bearer xxxx-xxxx-xxxx-xxxx`
  - アクセストークンはJWT形式とする
- 指定されたアクセストークンは、API GatewayオーソライザーにおいてJWTトークンとして検証する
- 指定されたアクセストークンを使い KeycloakのAPIを呼び出すことでも、Keycloakによってアクセス許可がチェックされる

## アクセストークンの発行方法

- OAuthクライアント・クレデンシャルズ・グラントとして定義されているフローを利用する
  - クライアントに対して事前に `client_id` と `client_secret` を発行しておく
  - `client_secret` は１年ごとに更新する
    - 初期は手動更新を想定するが、将来的には自動更新を検討する
- [Keycloak サービス・アカウント仕様](https://keycloak-documentation.openstandia.jp/4.0.0.Final/ja_JP/server_admin/index.html#_service_accounts)
- リフレッシュトークンは利用しない
- アクセストークンの期限は2時間とし、クライアントは残り1時間までに新しいトークン発行を要求することを想定する
- 旧トークンは期限まで利用可能とする（新トークン発行に伴う無効化はしない）

## API利用フロー

1. RP（例：PCA HubのPcaPlus）は、Keycloakに対して、`client_id` と `client_secret` を渡して新しいアクセストークンの発行を要求する
2. RPはアクセストークンを取得する
3. RPはアクセストークンをヘッダーに付加してPCA ID管理APIを呼び出す
4. PCA ID管理APIのAPI Gatewayオーソライザーは、渡されたアクセストークン（JWT形式）を検証することで認可のチェックは成功と判定する
5. PCA ID管理APIは、渡されたアクセストークンを使い、OAuthイントロスペクションAPIまたはKeycloak Admin REST APIを呼び出す
6. Keycloakでも認可は求められるので、PCA ID管理APIも合わせた２重のチェックを通過することで、すべての認可チェックが成功となる
