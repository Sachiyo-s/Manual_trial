---
sidebar_position: 51
---

# 51. WAF ルール

## AWSマネージドルール

- ルールバージョンは固定とし、手動で最新へ更新する運用とする
  - 原則として、ルール見直しの時点において、AWSが定義するデフォルトバージョンを採用していく
- [AWS マネージドルールの変更ログ](https://docs.aws.amazon.com/ja_jp/waf/latest/developerguide/aws-managed-rule-groups-changelog.html)

### 共通ルール調整

#### scope-down statement ルール調整

- `/admin` を対象外とする
  - `/api/admin` は対象であることに注意

### [ベースラインルールグループ](https://docs.aws.amazon.com/ja_jp/waf/latest/developerguide/aws-managed-rule-groups-baseline.html)

#### コアルールセット (CRS) マネージドルールグループ

- AWSManagedRulesCommonRuleSet
- Capacity: 700
- Version: 1.12
- ルール調整
  - NoUserAgent_HEADER
    - count
  - SizeRestrictions_BODY
    - count
  - EC2MetaDataSSRF_QUERYARGUMENTS
    - count
  - EC2MetaDataSSRF_BODY
    - count
- scope-down statement
  - 以下を対象外とする
    - `/auth/v1`
    - `/realms/pcaid/protocol/openid-connect`
    - `*/images/*`
  - 共通 scope-down（`/admin`）との組み合わせになることに注意

#### 既知の不正な入力マネージドルールグループ

- AWSManagedRulesKnownBadInputsRuleSet
- Capacity: 200
- Version: 1.22
- ルール調整
  - なし

### [ユースケース固有のルールグループ](https://docs.aws.amazon.com/ja_jp/waf/latest/developerguide/aws-managed-rule-groups-use-case.html)

#### SQL データベースマネージドルールグループ

- AWSManagedRulesSQLiRuleSet
- Capacity: 200
- Version: 1.2
- ルール調整
  - SQLi_BODY
    - count
- scope-down statement
  - `/auth/v1` と `/realms/pcaid/protocol/openid-connect` を対象外とする
  - 共通 scope-down（`/admin`）との組み合わせになることに注意

#### Linux オペレーティングシステムマネージドルールグループ

- AWSManagedRulesLinuxRuleSet
- Capacity: 200
- Version: 2.2
- ルール調整
  - なし

#### POSIX オペレーティングシステムマネージドルールグループ

- AWSManagedRulesUnixRuleSet
- Capacity: 100
- Version: 2.2
- ルール調整
  - なし

### [IP 評価ルールグループ](https://docs.aws.amazon.com/ja_jp/waf/latest/developerguide/aws-managed-rule-groups-ip-rep.html)

#### Amazon IP 評価リストマネージドルールグループ

- AWSManagedRulesAmazonIpReputationList
- Capacity: 25
- Version: Default
- ルール調整
  - count
  - 本番環境で様子を見る

## カスタムルール

### レートリミット

- すべてのユーザーが通常利用する認証系URL
  - ~~**1,800,000回/分**~~
    - ~~100万ユーザーに対して１秒あたりの同時アクセスを3％と考えると30,000回/秒~~
    - → レートリミットとして機能していないので採用しない
  - 対象URL
    - `/auth`
    - `/realms/pcaid`
- 攻撃者から防御すべきURL
  - **180回/分**
    - 手動操作であれば1秒間に2~3回アクセスできれば十分、社内NWがIP共有していても問題ないレベルと判断
  - 対象URL
    - `/realms/pcaid/login-actions/reset-credentials`
- その他URL
  - ~~**2400回/分**~~
    - ~~Hubが全体設定として設定している12,000回/5分間を採用する~~
    - → API Gateway スロットリングや Lambda 同時実行数で制御する
  - 対象URL
    - 管理APIエンドポイント
      - `/api`
      - `/storage`
    - フロントアプリ
      - `/orgs`
      - `/account`
      - `/storage`

### リクエストボディサイズ制限（16KB）の緩和

- ファイルアップロードを要件とするAPIエンドポイント
  - **50MB** でブロックとする
  - [組織画像アップロードAPI PUT /organizations/images/logo](/docs/api/organizations/images/logo/PUT%20組織画像アップロード.md)
  - [プロフィール画像アップロードAPI PUT /users/{account_id}/images/profile](/docs/api/users/account_id/images/profile/POST%20プロフィール画像アップロード.md)
  - [ユーザーインポートAPI PUT /users/import](/docs/api/users/import/POST%20ユーザーインポート非同期.md)
- それ以外のすべてのURL
  - コアルールと同じ **16KB** でブロックとする

### IP制限

- [許可IP（PoC・開発・検証環境）](./allow-ip.md)
- Keycloak管理コンソール（`/admin/master` から始まる）と Admin API（`/admin/realms/pcaid` から始まる）
  - 以下のシステムおよびネットワークのみ許可する（以下を除いて拒否する）
    - PCA Hub の許可IP
    - PCA
    - 外注開発組織
      - TDI
  - 本番環境は上記よりネットワークを厳しくして、CBC所有の踏み台サーバー（IP非公開）のみに許可範囲を制限する

## WAF設定

| # | ルール | アクション | 備考 |
| ---: | --- | :---: | --- |
| 0 | BBSec 脆弱性診断 | ◯ 許可 | 検証環境で使いWAFをスキップする |
| 1 | Keycloakイベント 内部呼び出し <br/> （Keycloak ECS から Lambda へのリクエスト） | ◯ 許可 | イベント専用の認証を使う |
| 2 | PCA ID 内部呼び出し <br/>（Lambda から ALB へのリクエスト） | ◯ 許可 | Refererでキーフレーズを使う |
| 3 | PCA Hub 内部呼び出し <br/>（Hub ECS から ALB へのリクエスト） | ◯ 許可 | TOTP認証を使う |
| 4 | パスワードリセット画面のレート制限 | ✕ 拒否 | 180回/分/IP |
| 5 | 50MB リクエストサイズ制限 | ✕ 拒否 | アップロード系API |
| 6 | 16KB リクエストサイズ制限 | ✕ 拒否 | アップロード系を除くすべてのAPI |
| 7 | Keycloak 管理コンソール | ✕ 拒否 | 本番環境→CBC踏み台のみ <br/> それ以外→社内NW＋TDIのみ |
| 8 | Keycloak 管理API（オリジナルパス） | ✕ 拒否 | 〃 |
| 9 | Keycloak 管理API（カスタムパス） | ✕ 拒否 | 〃 |
| 10 | PCA ID 管理API | カウント | 今後制限予定 |
| 11 | システム共通（AWSマネージドルール） | ✕ 拒否 <br/>（一部はカウント） ||
| 12 | 悪意のあるIP（AWSマネージドルール） |  カウント ||
| 13 | 不正入力（AWSマネージドルール） | ✕ 拒否 ||
| 14 | SQLインジェクション（AWSマネージドルール） | ✕ 拒否 ||
| 15 | Linux攻撃（AWSマネージドルール） | ✕ 拒否 ||
| 16 | UNIX攻撃（AWSマネージドルール） | ✕ 拒否 ||
| 17 | 許可IP | ◯ 許可 | 本番環境→**すべて許可** <br/> それ以外→開発・検証関係者のみ |
| | デフォルトアクション | ✕ 拒否 ||

## 参考情報

- [PCA ID ドメイン＆ベースURL定義](./pcaid-domain-and-baseurl.md)
- <https://pcajp.slack.com/archives/C046D8Z78G4/p1721812545569169>
  - [pcaplus-terraform/infrastructure/management/waf/variables.tf](https://github.com/pca-web/ap-shared-terraform/blob/develop/infrastructure/runtime/waf/variables.tf)
- [インターネットからの野良リクエストがどれぐらい AWS WAF のマネージドルールに一致するのか確かめてみた \- 電通総研 テックブログ](https://tech.dentsusoken.com/entry/waf_managed_rules_match_experiment)
- [AWS WAF について最初から知りたかったこと8選 \- 電通総研 テックブログ](https://tech.dentsusoken.com/entry/8_things_i_wanted_to_know_about_aws_waf)
