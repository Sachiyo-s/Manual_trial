---
sidebar_position: 1
---

# 1. PCA ID ドメイン＆ベースURL定義

## ドメイン定義

| 環境 | ドメイン | VPC CIDR | 用途 |
| -- | -- | -- | -- |
| PoC環境 | poc.id.pca-dev.net | 10.62.0.0/16 | ・PCA IDの開発作業用<br/> ・開発管理のAWSアカウントに構築 |
| 開発環境 | dev.id.pca-dev.net | 10.57.0.0/16 | ・PCA IDを利用するプロダクト開発用<br/> ・開発管理のAWSアカウントに構築 |
| 検証環境 | stg.id.pca-dev.net | 10.59.0.0/16 | ・検証作業用で本番相当とする<br/> ・開発管理のAWSアカウントに構築 |
| 開発フリー環境（要検討） | free.id.pa-dev.net | | ・開発チームが自由に触ることを想定した開発相当の環境 |
| 本番環境 | id.pca.jp | 10.60.0.0/16 | ・CBC管轄の独立したAWSアカウントに構築する |
| 本番サンドボックス環境 | id-sandbox.pca.jp | 10.61.0.0/16 | ・外部ベンダーへの公開を目的とした本番相当の環境 |

## 【外部公開】ベースURL定義

| 対象システム | ベースURL | デフォルトURL | 備考 |
| -- | -- | -- | -- |
| OIDC/OAuth | /auth/v1 | /realms/pcaid/protocol/openid-connect | バージョン管理は念のため別システムへの移行を想定する |
| 外部連携用 API Gateway | /api/apps/v1 | /dev-apps | ・外部連携APIを提供する管理APIラッパー<br/>・認可コードフロー <br/> ・CORS保護なし<br/>・[スコープ](./apps-integration-clients.md)を要求する <br/>・[組織またはアプリの管理者ロール](/docs/common/ロール（役割）.md)が必要 |
| 管理コンソール画面 | /orgs | /pca-pcaid-ui/orgs | 管理者画面において別途URLパス構成を設計する |
| 組織作成画面 | /orgs-new | /pca-pcaid-ui/orgs-new | 組織作成画面において別途URLパス構成を設計する |
| アカウント設定画面 | /account<br/>※ドメイン直下（`/`）も同じ | /pca-pcaid-ui/account | アカウント設定画面において別途URLパス構成を設計する |
| プロフィール画像<br/>（署名付き）| /storage/user/images | /pca-pcaid-images/users | OIDC対象外で署名期限は10秒とする |
| 組織画像<br/>（署名付き）| /storage/organization/images | /pca-pcaid-images/ogranizations | OIDC対象外で署名期限は10秒とする |

- 外部向け URL は、Cloudfront により公開することを想定する
- OIDC の認可コードフロー（Authorization Code Grant）での認証で保護される
- 画面系のデフォルト URL は、S3 の Web サイトホスティング機能における URL（先頭パスの `pca-pcaid-ui` はS3バケット名）を指している
- 画像系のデフォルト URL は、S3 パス（先頭パスの `pca-pcaid-images` は S3 バケット名）を指している

## 【非公開】ベースURL定義

| 対象システム | ベースURL | デフォルトURL | 備考 |
| -- | -- | -- | -- |
| PCA ID 管理API | /api/admin/v1 | /dev | ・Keycloak の標準管理API を拡張した管理API<br/>・クライアント認証フローとCORSで保護する |
| Keycloak 管理API | /api/admin-raw/v1 | /admin/realms/pcaid | ・[Keycloak Admin REST API](https://www.keycloak.org/docs-api/23.0.7/rest-api/index.html)<br/>・クライアント認証フローとCORSで保護する |
| 管理コンソール画面向け API Gateway | /api/orgs/v1 | /dev-orgs | ・管理者画面に特化したAPIを提供する管理APIラッパー<br/>・認可コードフローと同一オリジンポリシーで保護する<br/>・[組織管理者ロール](/docs/common/ロール（役割）.md)が必要 |
| 組織作成画面向け API Gateway | /api/orgs-new/v1 | /dev-orgs-new | ・組織作成画面に特化したAPIを提供する管理APIラッパー<br/>・PCAサービスごとの独自認証と同一オリジンポリシーで保護する |
| アカウント設定画面向け API Gateway | /api/account/v1 | /dev-account | ・アカウント設定画面に特化したAPIを提供する管理APIラッパー<br/>・認可コードフローと同一オリジンポリシーで保護する<br/>・[一般ユーザーロール](/docs/common/ロール（役割）.md)が必要 |
| 死活監視API | /api/health | /health | ・PCA ID全体のヘルスチェック用API <br/>・認証なし <br/>・IP制限なし <br/>・CORS保護なし |
| Keycloakトップページ | アクセス不可 | / | Keycloakの管理者コンソールやドキュメント類へアクセスできる |
| Keycloak管理者コンソール | パススルー | /admin/master/console | 開発・運用（CBC）向けのKeycloak標準機能で特権を利用できる |
| Keycloakアカウント設定 | パススルー | /realms/pcaid/account | 開発向けのKeycloak標準機能だが独自画面の参考にする程度 |

- API 系システムは、将来の破壊的変更を想定してバージョン管理（`/v1`）する
- クライアント認証フロー（Client Credentials Grant）においては、クライアントシークレットの定期的なローテーションを検討する
- 非公開システムであっても Cloudfront 経由でインターネット公開されるため、少なくとも IP 等によるアクセス制限を実施する
  - API 系システムについては、同一オリジンポリシーやオリジン間リソース共有（CORS）により保護する
  - Keycloak ページについて、本番環境においては、踏み台サーバーからのアクセス（IP 制限をさらに強化）に制限する
- 管理者コンソールの認証は、2FA の有効化を必須とする

## OpenID Well-Known Configution

### Configution エンドポイント

- 公開方法
  - Cloudfront によってアクセス先を制御する
  - 外部公開用 URL は S3 へルーティングし、内部確認用は Keycloak の ALB へルーティングする
- 外部公開用（S3配置）
  - `/realms/pcaid/.well-known/openid-configuration`
- 内部確認用（Keycloak標準機能）
  - `/realms/pcaid-raw/.well-known/openid-configuration`

### OpenID Well-Known エンドポイント

- `/auth/v1`
  - `/realms/pcaid/protocol/openid-connect` を置き換える
  - 可能な限り Keycloak に依存する外部との接点を小さくする
- 発行元（Issuer）
  - `https://{pcaid-domain}/realms/pcaid`
  - JWTトークンにも含まれる
  - Keycloak ではこの値をカスタマイズできない
- 認可エンドポイント
  - `https://{pcaid-domain}/auth/v1/auth`
- トークンエンドポイント
  - `https://{pcaid-domain}/auth/v1/token`

## 関連情報

- [OIDC/OAuthクライアント情報](./oidc-oauth-clients.md)
