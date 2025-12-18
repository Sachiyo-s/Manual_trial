---
sidebar_position: 13
---

# 13. API オーソライザー仕様

## API オーソライザーとは

- [API Gateway Lambda オーソライザーを使用する \- Amazon API Gateway](https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/apigateway-use-lambda-authorizer.html)

## PCA ID オーソライザー

- API 構成イメージ
  - Lambda 関数により実装している各種の API は、目的に応じた API Gateway のオーソライザーを通過することで利用できる

![PCA ID API構成](./images/PCA%20ID%20API%20構成.png)

### [1. 管理 API オーソライザー（pcaid-authorizer）](https://github.com/pca-idp/pcaid-management-api/blob/develop/src/lambdas/pcaid-authorizer/index.ts)

- PCA ID 管理API
  - `/api/admin/v1`

#### 1-1. 入力値

- `traceparent`
  - トレースID
- `Authorization`
  - アクセストークン
    - オーソライザーによるクライアントクレデンシャルフロー
      - 呼び出し元のクライアントクレデンシャルをそのまま利用する
      - 例：`pcahub-management-api`（`realm-admin` ロールを持つ）

#### 1-2. 出力値（Lambda関数へ渡すコンテキスト情報）

- PCAアカウントの `account_id`
  - オーソライザーにおけるプリンシパルID

#### 1-3. 大まかな流れ

   1. API Gateway の Authorizer から呼び出される
   2. 渡された `event` から鍵を識別するための `kid` を取得する
   3. [well-known 情報](/docs/common-dev/pcaid-domain-and-baseurl.md#openid-well-known-configution) から、公開鍵エンドポイント `jwks_uri` を取得する
      - Keycloak の `/realms/pcaid/.well-known/openid-configuration` へアクセスする
   4. `kid` と `jwks_uri` を使用して、PCA ID が発行している公開鍵 `jwk` を取得する
      - Keycloak の `/auth/v1/certs` へアクセスする
   5. アクセストークンの検証を行い、有効期限や署名、発行者を確認し、結果として `JWT Payload` を受け取る
   6. 検証結果として返ってきた `JWT Payload` に有効な値が含まれていれば API Gateway の Authorizer に許可レスポンスを返す
      - `JWT Payload` が不正であるか、途中でエラーが発生したら拒否レスポンスを返す

### [2. アカウント設定向け API オーソライザー（pcaid-accountAuthorizer）](https://github.com/pca-idp/pcaid-management-api/blob/develop/src/lambdas/pcaid-accountAuthorizer/index.ts)

- アカウント設定画面向け
  - `/api/account/v1`

#### 2-1. 入力値

- `traceparent`
  - トレースID
- `Authorization`
  - アクセストークン
    - ユーザーによる認可コードフロー

#### 2-2. 出力値（Lambda関数へ渡すコンテキスト情報）

- PCAアカウントの `account_id`
  - オーソライザーにおけるプリンシパルID
- アクセストークン
  - オーソライザーによるクライアントクレデンシャルフロー
    - `pcaid-account-api`（`realm-admin` ロールを持つ）
- ユーザーロール
  - [ロール（役割）](/docs/common/ロール（役割）.md)
    - `pca.id.xxxx-xx-xx-xxxx/user`
      - オーソライザーにおける許可条件となるロール
    - ユーザーに割り当て済みのロール（組織管理者など）
      - オーソライザーではチェック対象外

#### 2-3. 大まかな流れ

   1. 呼び出し元のアクセストークン検証をして `JWT Payload` を取得するまでは、[管理 API オーソライザー（pcaid-authorizer）](./api-authorizers.md#1-%E7%AE%A1%E7%90%86-api-%E3%82%AA%E3%83%BC%E3%82%BD%E3%83%A9%E3%82%A4%E3%82%B6%E3%83%BCpcaid-authorizer) と同じ
      - `JWT Payload` が不正であれば拒否レスポンスを返す
   2. 管理クライアント `pcaid-account-api` のアクセストークンを取得する
      1. シークレット情報として扱っている、管理クライアントのアクセストークンを取得する
         - AWS Secrets Manager へ読み取りアクセスする
      2. 管理クライアントを使って呼び出し元のアクセストークンを検証する
         - Keycloak の `/auth/v1/token/introspect` へアクセスする
         - 呼び出し元が有効状態でなければ拒否レスポンスを返す
      3. アクセストークンが無効（空か期限切れ）なら、新たなアクセストークンを取得して更新する
         - Keycloak の `/auth/v1/token` へアクセスする
         - AWS Secrets Manager へ書き込みアクセスする
   3. ロールのチェックを行う
      1. 呼び出し元のアクセストークンから userinfo を取得する
         - Keycloak の `/auth/v1/userinfo` へアクセスする
      2. userinfo レスポンスに[有効なロール](/docs/common/ロール（役割）.md)が含まれているかチェックする
         - アクセス可能な組織の一般ユーザーロール：`pca.id.{organization_id}/user`
         - 不正なロールだった場合はエラーとなり拒否レスポンスを返す
   4. API Gateway の Authorizer に許可レスポンスを返す
      - 途中でエラーが発生したら拒否レスポンスを返す

### [3. 管理コンソール向け API オーソライザー（pcaid-portalAuthorither）](https://github.com/pca-idp/pcaid-management-api/blob/develop/src/lambdas/pcaid-portalAuthorizer/index.ts)

- 管理コンソール画面向け
  - `/api/orgs/v1`

#### 3-1. 入力値

- `traceparent`
  - トレースID
- `Authorization`
  - アクセストークン
    - ユーザーによる認可コードフロー
- `X-PCA-organization-id`
  - 組織ID
- `X-PCA-service-partition`
  - サービス区画

#### 3-2. 出力値（Lambda関数へ渡すコンテキスト情報）

- PCAアカウントの `account_id`
  - オーソライザーにおけるプリンシパルID
- アクセストークン
  - オーソライザーによるクライアントクレデンシャルフロー
    - `pcaid-management-ui-api`（`realm-admin` ロールを持つ）

#### 3-3. 大まかな流れ

   1. 呼び出し元のアクセストークン検証をして `JWT Payload` を取得するまでは、[管理 API オーソライザー（pcaid-authorizer）](./api-authorizers.md#1-%E7%AE%A1%E7%90%86-api-%E3%82%AA%E3%83%BC%E3%82%BD%E3%83%A9%E3%82%A4%E3%82%B6%E3%83%BCpcaid-authorizer) と同じ
      - `JWT Payload` が不正であれば拒否レスポンスを返す
   2. 管理クライアント `pcaid-management-ui-api` のアクセストークンを取得する
      - アクセストークン取得の流れは、[アカウント設定向け API オーソライザー（pcaid-accountAuthorizer）](./api-authorizers.md#2-%E3%82%A2%E3%82%AB%E3%82%A6%E3%83%B3%E3%83%88%E8%A8%AD%E5%AE%9A%E5%90%91%E3%81%91-api-%E3%82%AA%E3%83%BC%E3%82%BD%E3%83%A9%E3%82%A4%E3%82%B6%E3%83%BCpcaid-accountauthorizer) と同じ
   3. ロールのチェックを行う
      1. 呼び出し元のアクセストークンから userinfo を取得する
         - Keycloak の `/auth/v1/userinfo` へアクセスする
      2. リクエストヘッダーに組織ID `organization_id` が設定されていない場合はエラーとする
      3. userinfo レスポンスに[有効なロール](/docs/common/ロール（役割）.md)が含まれているかチェックする
         - アクセス対象の組織管理者ロール：`pca.id.{organization_id}/admin`
         - アクセス対象のサービス責任者ロール：`pca.id.{organization_id}/{service_partition}:head`
         - 不正なロールだった場合はエラーとなり拒否レスポンスを返す
   4. ロールに応じて[呼び出し可能なAPI](/docs/api/orgs-v1.md)をチェックする
      - 組織管理者は、すべてのAPIを呼び出せる
      - サービス責任者は、対象サービスを扱う特定のAPIのみ呼び出せる
   5. API Gateway の Authorizer に許可レスポンスを返す
      - 途中でエラーが発生したら拒否レスポンスを返す

### [4. アプリ向け API オーソライザー（pcaid-cloudAuthorither）](https://github.com/pca-idp/pcaid-management-api/blob/develop/src/lambdas/pcaid-cloudAuthorizer/index.ts)

- [PCA ID 連携クライアント](./apps-integration-clients.md)向け
  - `/api/apps/v1`

#### 4-1. 入力値

- `traceparent`
  - トレースID
- `Authorization`
  - アクセストークン
    - ユーザーによる認可コードフロー
- `X-PCA-service-partition`
  - サービス区画

#### 4-2. 出力値（Lambda関数へ渡すコンテキスト情報）

- PCAアカウントの `account_id`
  - オーソライザーにおけるプリンシパルID
- アクセストークン
  - オーソライザーによるクライアントクレデンシャルフロー
    - `apps-integration-api`（`realm-admin` ロールを持つ）
- ユーザーロール
  - [ロール（役割）](/docs/common/ロール（役割）.md)
    - `pca.id.xxxx-xx-xx-xxxx/user`
    - `pca.id.xxxx-xx-xx-xxxx/admin`
    - `pca.id.xxxx-xx-xx-xxxx/pca.cloud.org-xxxx-xxxx:head`

#### 4-3. 大まかな流れ

   1. 呼び出し元のアクセストークン検証をして `JWT Payload` を取得するまでは、[管理 API オーソライザー（pcaid-authorizer）](./api-authorizers.md#1-%E7%AE%A1%E7%90%86-api-%E3%82%AA%E3%83%BC%E3%82%BD%E3%83%A9%E3%82%A4%E3%82%B6%E3%83%BCpcaid-authorizer) と同じ
      - `JWT Payload` が不正であれば拒否レスポンスを返す
   2. クライアントが呼び出すアプリAPIが、アクセスを許可した[スコープ](./apps-integration-clients.md#oidcoauth-%E3%82%B9%E3%82%B3%E3%83%BC%E3%83%97)に含まれることを確認する
   3. 管理クライアント `apps-integration-api` のアクセストークンを取得する
      - アクセストークン取得の流れは、[アカウント設定向け API オーソライザー（pcaid-accountAuthorizer）](./api-authorizers.md#2-%E3%82%A2%E3%82%AB%E3%82%A6%E3%83%B3%E3%83%88%E8%A8%AD%E5%AE%9A%E5%90%91%E3%81%91-api-%E3%82%AA%E3%83%BC%E3%82%BD%E3%83%A9%E3%82%A4%E3%82%B6%E3%83%BCpcaid-accountauthorizer) と同じ
   4. 呼び出し元のアクセストークンから userinfo を取得する
      - Keycloak の `/auth/v1/userinfo` へアクセスする
   5. 呼び出し元のクライアントが認められたサービス種類を使用していることをチェックする
      1. クライアントIDとサービス種類（例：pca.cloud）の組み合わせを確認する
         - DynamoDB の `pcaid-service_client_master_table` テーブルへアクセスする
         - 組み合わせを取得できなければ拒否レスポンスを返す
   6. ロールのチェックを行う
      1. リクエストヘッダーのサービス区画から、組織ID `organization_id` を取得する
         - Keycloak の `/groups?search={service_partition}&exact=true&briefRepresentation=true` へアクセスする
         - 組織IDを特定できなければ拒否レスポンスを返す
      2. userinfo レスポンスに[有効なロール](/docs/common/ロール（役割）.md)が含まれているかチェックする
         - アクセス対象の組織管理者ロール：`pca.id.{organization_id}/admin`
         - アクセス対象のサービス責任者ロール：`pca.id.{organization_id}/{service_partition}:head`
         - アクセス対象の一般ユーザーロール：`pca.id.{organization_id}/user`
         - 不正なロールだった場合は拒否レスポンスを返す
   7. ロールに応じて[呼び出し可能なAPI](/docs/api/apps-v1.md)をチェックする
      - 組織管理者は、すべてのAPIを呼び出せる
      - サービス責任者は、すべてのAPIを呼び出せるが、対象データはサービス内に制限する
      - 一般ユーザーは、自分自身を扱うAPIのみ呼び出せる
      - 呼び出しが認められていないAPIであれば拒否レスポンスを返す
   8. サービス契約状態をチェックする
      1. 連携コードから契約IDを取得する
         - リクエストヘッダーのサービス区画から、連携コードを解析する
         - DynamoDB の `pcaid-new-orgs-connect_code` テーブルへアクセスする
      2. 契約IDからサービス契約のキャッシュデータを取得する
         - サービス契約をキャッシュ管理するLambda関数 `pcaid-contractEventBridge` を呼び出す
         - [サービス契約データのキャッシュ実装](https://github.com/pca-idp/pcaid/discussions/1057)
      3. サービス契約が有効状態であることを確認する
         - サービス契約が存在しなかったり、契約が無効状態なら拒否レスポンスを返す
   9. API Gateway の Authorizer に許可レスポンスを返す
      - 途中でエラーが発生したら拒否レスポンスを返す

### [5. 組織作成向け API オーソライザー（pcaid-newOrgAuthorither）](https://github.com/pca-idp/pcaid-management-api/blob/develop/src/lambdas/pcaid-newOrgsAuthorizer/index.ts)

- 組織作成画面向け
  - `/api/orgs-new/v1`

#### 5-1. 入力値

- `traceparent`
  - トレースID
- `Authorization`
  - ワンタイムパスワード（OTP）
- `X-PCA-receipt-session-id`
  - 受付セッションID

#### 5-2. 出力値（Lambda関数へ渡すコンテキスト情報）

- アクセストークン
  - オーソライザーによるクライアントクレデンシャルフロー
    - `apps-integration-api`（`realm-admin` ロールを持つ）

#### 5-3. 大まかな流れ

   1. 管理クライアント `apps-integration-api` のアクセストークンを取得する
      1. シークレット情報として扱っている、管理クライアントのアクセストークンを取得する
         - AWS Secrets Manager へ読み取りアクセスする
      2. アクセストークンが無効（空か期限切れ）なら、新たなアクセストークンを取得して更新する
         - Keycloak の `/auth/v1/token` へアクセスする
         - AWS Secrets Manager へ書き込みアクセスする
   2. 受付セッションが指定されていたら有効かチェックする
      - DynamoDB の `pcaid-new-orgs-new_org_prepare` テーブルへアクセスする
      - 受付セッションを取得できなければ拒否レスポンスを返す
   3. ワンタイムパスワード（OTP）が指定されていたら有効かチェックする（[OTP認証](./otp-authentication.md)）
      - シークレット情報として扱っている、ソルト値を取得する
        - AWS Secrets Manager へ読み取りアクセスする
      - ワンタイムパスワードを計算して検証する
        - リクエストヘッダーの `UserAgent` とソルト値を組み合わせてHMAC鍵とする
        - HMAC鍵を使って現在時刻に対するハッシュ値を求める
      - ワンタイムパスワードが不一致であれば拒否レスポンスを返す
   4. API Gateway の Authorizer に許可レスポンスを返す
      - 途中でエラーが発生したら拒否レスポンスを返す

## 関連文書

- [PCA ID ドメイン＆ベースURL定義](./pcaid-domain-and-baseurl.md)
