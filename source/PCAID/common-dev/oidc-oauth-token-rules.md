---
sidebar_position: 11
---

# 11. OIDC/OAuth トークン仕様

## トークン形式

- JWT仕様に準拠した電子署名付きのJSON形式（JWS）とする
  - 署名アルゴリズムは `ES256` とする
- トークンの暗号化はしない
  - KeycloakはIDトークンの暗号化（JWE）をサポートしてるが利用しない
  - Keycloakはアクセストークンの暗号化をサポートしていない

### 参考情報

- [2020年版 チーム内勉強会資料その1 : JSON Web Token \- r\-weblife](https://ritou.hatenablog.com/entry/2020/06/08/050000)
- [3\. JWT 型アクセストークンの考慮事項 \- Authlete](https://www.authlete.com/ja/resources/videos/20200422/03/)
- [OAuth アクセストークンの実装に関する考察 \#OAuth \- Qiita](https://qiita.com/TakahikoKawasaki/items/970548727761f9e02bcd)

## グラント種類のユースケースと利用するトークン種類

### 1. ユーザーと組織管理用のクライアントクレデンシャルズフロー

- PCA ID管理API（Keycloak Admin REST API をラップする Lambda実装）の呼び出し許可に利用される
  - [PCA ID管理APIにおける認可方法](./pcaid-management-api-granttype.md)
  - PCA Hub においては PcaPlus が利用する
  - PCAクラウド/PCAサブスクにおいては開発フレームワークが利用する
- アクセストークンのみ利用する
  - IDトークンは存在せずリフレッシュトークンは利用しない
  - トークン期限は **2時間** とする
- 拡張クレームなし
- Keycloak管理権限あり
  - Keycloakをすべて管理＆利用する権限が付与される

#### トークン設定（Client Advanced settings）

| 設定項目 | Keycloak設定名 | 値 |
| --- | --- | --- |
| リフレッシュトークン利用 | Use refresh tokens | Off |
| アクセストークン期限 | Access Token Lifespan | 2時間 |

### 2. PCAサービス等のRPが利用するユーザー認証用の認可コードフロー

- RPのフロントアプリ（OIDCクライアント）が、PCA IDからユーザー認証の結果を受け取るために利用する
  - ユーザー認証を依頼したRPはトークンの発行に利用する
  - PCA IDのフロントアプリ（アカウント設定、管理コンソール）もOIDCクライアントの１つとなる
- IDトークンとアクセストークン、およびリフレッシュトークンを利用する
  - IDトークンとアクセストークンの期限は **60分** とする
    - トークンの永続化は禁止とする
    - APIにおけるトークンの検証は、最新状態を確認するため、 **10分** 以内にイントロスペクションAPIを利用する
      - 上記の範囲で、オンメモリでのトークン検証や、キャッシュしたイントロスペクション結果を利用して構わない
      - とくに安全性が求められるクライアントにおいては、トークンの有効状態をリアルタイムで反映するため、**常に**イントロスペクションAPIを利用することを検討する
    - APIにおけるトークンの検証は、**常に**イントロスペクションAPIを利用する
      - トークンの有効状態はリアルタイムで反映される
  - リフレッシュトークンの期限は **90日** とする
    - ~~最新のリフレッシュトークンのみ有効とする~~
    - 世代の古くなったリフレッシュトークンも有効とする
      - １つのWebフロントアプリを複数タブで同時に開く状況であっても、独立してリフレッシュトークンを扱えるようにする
      - PCA Hub のトークン仕様との互換性を維持する

#### トークン設定（Realm settings）

| 設定項目 | Keycloak設定名 | 値 | [PCA Hub の既存仕様](https://pca1980.backlog.com/alias/wiki/890587) |
| --- | --- | --- | --- |
| アクセストークン期限 | Access Token Lifespan | 60分 | 60分 |
| リフレッシュトークン期限 | Offline Session Idle | 90日 | 90日 |
| リフレッシュトークン継続期限 | Offline Session Max | 無期限に継続可能 | 無期限に継続可能 |
| リフレッシュトークン破棄 | Revoke Refresh Token | 破棄しない（無効） | 破棄しない |
| リフレッシュトークン再利用回数 | Refresh Token Max Reuse | 制限なし | 制限なし |

- [Red Hat build of Keycloak セッションおよびトークンのタイムアウト](https://access.redhat.com/documentation/ja-jp/red_hat_build_of_keycloak/22.0/html/server_administration_guide/timeouts)

#### トークン項目（クレーム）

- トークン共通項目（必須クレーム）
  - 発行者
  - 発行先
  - 発行日時と期限
  - アカウントID
- IDトークン
  - 原則として何らかの実体を指すID類を含める
    - 所属組織（複数あり）
      - 組織名
    - 利用可能なサービス区画（複数あり）
- UserInfo API
  - IDトークンに準じる
    - 割り当て済みロールは IDトークンには含めず UserInfo にのみ含める
    - Hubでのクッキー認証Claimの肥大化問題を回避するため
  - フロントで所属組織を判断するためのロール情報を含める
    - 割り当て済みのロール（複数あり）
      - 組織ID
      - 組織内権限
  - 拡張データは、類似の独自API（/me）として提供する
    - [ユーザー情報取得API GET /me](/docs/api/me/GET%20ユーザー情報取得.md)
- アクセストークン
  - アカウントIDのみを含める
    - 必要な権限情報はイントロスペクションAPIから取得する
  - クライアントはトークン内容を参照しないことを前提とする

IDトークンの例

```json
{
    "exp": 1738640446,
    "iat": 1738636846,
    "auth_time": 1738636745,
    "jti": "b29ffc21-f570-4b90-a37b-61b8990b7c25",
    "iss": "https://poc.id.pca-dev.net/realms/pcaid",
    "aud": "pcaapp20-ui",
    "sub": "cb8a7e9b-6495-4149-825a-1d6e8dc26894",
    "typ": "ID",
    "azp": "pcaapp20-ui",
    "nonce": "E435FdmYQG9MYkNwSoTwS3uCqDeuRPJA6oPrbeViuXo",
    "session_state": "010d33c9-63ac-4671-940f-add3e88792c6",
    "at_hash": "md_Og93mxGu7ToG3TPI1bg",
    "acr": "0",
    "sid": "010d33c9-63ac-4671-940f-add3e88792c6",
    "orgs": [
        "/org-0aa2-84cd",
        "/org-0aa2-84cd/pca.hub.tenant1",
        "/org-0aa2-84cd/pca.cloud.org-8106-e7ba",
        "/org-0aa2-84cd/pca.subsc.org-0aa2-84cd"
    ]
}
```

UserInfoレスポンスの例

```json
{
    "sub": "cb8a7e9b-6495-4149-825a-1d6e8dc26894",
    "realm_access": {
        "roles": [
            "pca.id.18ac8025-39b5-4cbf-81cc-b41044e582c9/user",
            "pca.id.04472e89-1060-4483-8b6e-8df5bba370be/admin",
            "pca.id.0108d967-2c0e-4189-b5ae-4e48fd0e8b5a/user",
            "offline_access",
            "default-roles-pcaid",
            "uma_authorization"
        ]
    },
    "orgs": [
        "/org-0aa2-84cd",
        "/org-0aa2-84cd/pca.hub.tenant1",
        "/org-0aa2-84cd/pca.cloud.org-8106-e7ba",
        "/org-0aa2-84cd/pca.subsc.org-0aa2-84cd"
    ]
}
```

アクセストークンの例

```json
{
    "exp": 1738640446,
    "iat": 1738636846,
    "auth_time": 1738636745,
    "jti": "df338f91-7c2e-444d-8851-f289566111b3",
    "iss": "https://poc.id.pca-dev.net/realms/pcaid",
    "aud": [
        "realm-management",
        "account"
    ],
    "sub": "cb8a7e9b-6495-4149-825a-1d6e8dc26894",
    "typ": "Bearer",
    "azp": "pcaapp20-ui",
    "nonce": "E435FdmYQG9MYkNwSoTwS3uCqDeuRPJA6oPrbeViuXo",
    "session_state": "010d33c9-63ac-4671-940f-add3e88792c6",
    "acr": "0",
    "allowed-origins": [
        "http://localhost"
    ],
    "resource_access": {
        "realm-management": {
            "roles": [
                "view-events"
            ]
        },
        "account": {
            "roles": [
                "manage-account",
                "view-groups"
            ]
        }
    },
    "scope": "openid group_membership id.organization.read id.me",
    "sid": "010d33c9-63ac-4671-940f-add3e88792c6"
}
```
