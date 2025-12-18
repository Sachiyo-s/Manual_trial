---
sidebar_position: 22
---

# 22. OIDC/OAuth クライアント情報

## クライアント一覧

| クライアントID | 名前 | 種類 | 提供先 | 用途 |
| --- | --- | --- | --- | --- |
| pcaid-management-ui | PCA ID 管理コンソール | Public <br/> (Authz Code) | PCA ID（内部利用） | 組織管理者向けフロントエンド |
| pcaid-management-ui-api | PCA ID 管理コンソールAPI | Confidential <br/> (Client Credentials) | PCA ID（内部利用） | 組織管理者向けバックエンドで、対応するフロントエンドから呼び出されることを想定する |
| pcaid-account-settings-ui | PCA ID アカウント設定 | Public <br/> (Authz Code) | PCA ID（内部利用） | 一般ユーザー向けアカウント設定フロントエンド |
| pcaid-account-api | PCA ID アカウント設定API | Confidential <br/> (Client Credentials) | PCA ID（内部利用） | 一般ユーザー向けアカウント設定バックエンドで、対応するフロントエンドから呼び出されることを想定する <br/> 一部のAPI（例：`/me`）は、RPにも提供することを想定する |
| pcaid-batch-job | PCA ID バッチジョブ | Confidential <br/> (Client Credentials) | PCA ID（内部利用） | バッチジョブによる管理 <br/>例：ユーザーインポート |
| pcahub-authn-ui | PCA Hub | Public <br/> (Authz Code) | PCA Hub | PCA Hub でのユーザー認証 |
| pcahub-management-api | PCA Hub API | Confidential <br/> (Client Credentials) | PCA Hub | Hub管理APIからの管理 |
| pca-crm | PCA顧客システム | Confidential <br/> (Client Credentials) | PCA社内システム<br/>例：顧客管理、運用管理 | 顧客ID（外部）の管理 |
| apps-integration-api | PCA ID 連携アプリAPI | Confidential <br/> (Client Credentials) | PCA ID 連携アプリ | PCA ID 連携アプリ向けバックエンドで、PCA ID と連携する任意のアプリ（例：PCAクラウド等）から呼び出されることを想定する |

## PCA ID 連携クライアント一覧

| クライアントID | 名前 | 種類 | 提供先 | 用途 |
| --- | --- | --- | --- | --- |
| pcaapp20-ui | PCAクラウド／PCAサブスク | Public <br/> (Authz Code) | PCAクラウド <br/> PCAサブスク | PCAクラウドとPCAサブスクのPCA ID連携 |
| pcacloud-devsite | PCAクラウド 開発者サイト | Confidential <br/> (Authz Code) | PCAクラウド | PCAクラウド 開発者サイトでのユーザー認証 |
| pcacloud-api | PCAクラウド Web-API | Confidential <br/> (Authz Code) | PCAクラウド | PCAクラウド Web-APIでのユーザー認証 |
| pca-portal-ui | PCA Arch ポータル | Public <br/> (Authz Code) | PCA Arch | PCA Arch ポータルのフロントエンド |
| pca-apps-download | PCAソフトダウンロード | Public <br/> (Authz Code) | PCAウェブサイト | お客様サポートの会員限定メニューでのユーザー認証 |
| xronosid-auth | PCA Hub 経費精算 SSO | Confidential <br/> (Authz Code) | クロノス基盤 | クロノスIDとのSSO連携 |
| xronos-ui | PCA Hub 経費精算 | Public <br/> (Authz Code) | クロノス基盤 | WebアプリのフロントエンドSLO連携（IDトークン取得） |
| xronos-ui-mobile | PCA Hub 経費精算 モバイル | Public <br/> (Authz Code) | クロノス基盤 | スマホアプリのフロントエンドSLO連携（IDトークン取得） |
| xronos-management-api | PCA Hub 経費精算 API | Confidential <br/> (Client Credentials) | クロノス基盤 | バックエンドで組織情報を取得する |
| dreamhop-resq | ドリームホップ Res-Q | Public <br/> (Authz Code) | ドリームホップ Res-Q | Res-Qとの認証連携 |

### リダイレクトURL

| クライアント | 環境別URL<br/> （+ローカルDocker環境） | 用途 |
| --- | --- | --- |
| PCA ID 管理コンソール <br/> (pcaid-management-ui)  | `https://{pcaid-domain-name}/orgs`<br/> ※ローカル環境不可 | 管理コンソールにおける認証コールバック先 |
| PCA ID アカウント設定 <br/> (pcaid-account-settings-ui)  | `https://{pcaid-domain-name}/account`<br/> `https://{pcaid-domain-name}` <br/> ※ローカル環境不可 | アカウント設定における認証コールバック先 |
| PCA Hub <br/> (pcahub-authn-ui) | `https://*.{hub-domain-name}/auth/v1/account/pcaid_redirect`<br/> (`http://*.docker.internal:9999/auth/v1/account/pcaid_redirect`) | ユーザーの認証コールバック先 |
| PCA Hub API <br/> (pcahub-management-api) |  `https://*.{hub-domain-name}/g` <br/> (`http://*.docker.internal:9999/g/api/swagger/index.html`) | メールアクション完了後のリダイレクト先<br/>例：Hubユーザーポータル（/g） |
| PCAクラウド <br/> PCAサブスク <br/> (pcaapp20-ui)  | `urn:ietf:wg:oauth:2.0:oob` <br/> `http://localhost/`<br/> ※ポート番号はランダム指定可、末尾スラッシュは必須 | 認可コードのコールバック先 |
| PCAクラウド 開発者サイト <br/> (pcacloud-devsite) | `urn:ietf:wg:oauth:2.0:oob` <br/> `https://{pcacloud-domain-name}/signin-pcaid` | 開発者サイトに対する認可コードのコールバック先 |
| PCAクラウド Web-API <br/> (pcacloud-api) | `urn:ietf:wg:oauth:2.0:oob` <br/> `https://{pcacloud-domain-name}/signin-pcaid` | Web-APIに対する認可コードのコールバック先 |
| PCA Arch ポータル <br/> (pca-portal-ui) | `https://arch.pca.jp/oauth/pcaid-redirect` <br/> `https://arch-sandbox.pca.jp/oauth/pcaid-redirect` <br/> `https://*.portal.pca-dev.net/oauth/pcaid-redirect` <br/> `http://localhost:3000/oauth/pcaid-redirect` <br/> ※ローカル環境（localhost）は poc と dev のみ <br/><br/> ※Post logout URL <br/> `https://{portal-domain-name}/*` | PCA Arch ポータルにおける認可コードの[コールバック先](https://github.com/pca-idp/pcaid/issues/1597) |
| PCAソフトダウンロード <br/> (pca-apps-download) | `https://{pca-domain-name}/pcaid_auth_callback` | ダウンロードサービスに対する認可コードのコールバック先 |
| PCA Hub 経費精算 SSO <br/> (xronosid-auth) | `https://{instance-id}.identity.oraclecloud.com/oauth2/v1/social/callback` <br/><br/> 本番インスタンス：idcs-33e5259d6c6f4dc8a03829b79fe6a740 <br/> 検証インスタンス：idcs-86d163727771448e820f9ff8164901f3 <br/> 開発インスタンス：idcs-aa09601e5c88415185b98e7b43abd2c1 | クロノスIDに対する認可コードのコールバック先 |
| PCA Hub 経費精算 <br/> (xronos-ui) | `https://pca-oauth-service.{hub-domain-name}/k/oauth-callback-pcaid` <br/> `https://pca-oauth-service.{hub-domain-name}/ks/oauth-callback-pcaid` <br/><br/> ※Post logout URL <br/> `https://*.{hub-domain-name}/k/` <br/> `https://*.{hub-domain-name}/ks/` | クロノスIDに対する認可コードのコールバック先（WebアプリSLO用） |
| PCA Hub 経費精算 モバイル <br/> (xronos-ui-mobile) | `pca-hub-keihi://OAuthLoginP` <br/><br/> ※Post logout URL <br/> `pca-hub-keihi://OAuthLogoutP` | クロノスIDに対する認可コードのコールバック先（スマホアプリSLO用） |
| ドリームホップ Res-Q <br/> (dreamhop-resq) | `https://resq-survey.com/auth/callback` <br/> `https://resq.dev.dreamhop.jp/auth/callback` | Res-Qに対する認可コードのコールバック先 |

## PCA ID 連携クライアント一覧（OAuthリソースサーバー）

| クライアントID        | 名前                    | 提供先      | 用途                               | 必須スコープ               |
| ---                  | ---                    | ---         | ---                                | ---                       |
| pca-portal-api       | PCA Arch ポータル API   | PCA Arch    | Archポータル APIでの認可チェック用   | arch.portal.basic         |
| pca-ai-assistant-api  | AIアシスタント API      | PCA Arch    | AIアシスタント APIでの認可チェック用  | arch.ai-assistant.basic  |

※ OAuthリソースサーバーは、原則として Confidential クライアント (認証方法は Client Credentials フロー) として扱います。

## 関連文書

- [PCA ID ドメイン＆ベースURL定義](./pcaid-domain-and-baseurl.md)
- [OIDC/OAuthトークン仕様](./oidc-oauth-token-rules.md)
- [クロノスによる共有資料（CBC様へ依頼する内容_20250213.xlsx）](./materials/CBC様へ依頼する内容_20250213.xlsx)
