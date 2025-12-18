---
sidebar_position: 6
---

# 6. S3 データ仕様

| データ種類 | バケット名 | パス | 用途 | アクセスログ | バックアップ | 備考 |
| --- | --- | --- | --- | :---: | :---: | --- |
| [アプリログ](./app-log-spec.md) | pcaid-app-log-raw | | CloudWatchログストリーム | ◯ | ◯ | |
| 　〃 | pcaid-app-log-glue | | Glueテーブル <br/> Athenaメタデータ <br/> split.py のみ | | | |
| 　〃 | pcaid-app-log | | | | | 未使用 |
| 　〃 | pcaid-profilebucket | /app_log | Athenaクエリー保存先 | | | |
| [操作ログ](/docs/common/操作ログの種類と詳細.md) | pcaid-audit-log-raw | | CloudWatchログストリーム | ◯ | ◯ | |
| 　〃 | pcaid-audit-log-glue | | Glueテーブル <br/> Athenaメタデータ | | | |
| 　〃 | pcaid-audit-log | | | | | 未使用 |
| 　〃 | pcaid-profilebucket | /audit_log | Athenaクエリー保存先 | | | |
| 　〃 | pcaid-profilebucket | /storage/audit_log | リレーションデータ | | | |
| 　〃 | pcaid-profilebucket | /storage/audit_log_download | 署名付きURL発行先 | | | BOM付きCSV |
| Keycloakログ | pcaid-keycloak-log-raw | | CloudWatchログストリーム | ◯ | | [Keycloakログ設計](https://github.com/pca-idp/pcaid/issues/91) |
| [ユーザーCSV](/docs/api/users/import/POST%20ユーザーインポート非同期.md) | pcaid-profilebucket | | ユーザーデータ <br/> アップロード先 | ◯ | ◯ | [ユーザーCSVフォーマット](/docs/common/ユーザーCSVフォーマット.md) |
| [Hubテナント状態](/docs/api/hub_authn_switchings/GET%20Hubテナント移行状態取得.md) | pcaid-profilebucket | /storage/hub_authn_switch | 署名付きURL発行先 | ◯ | ◯ | JSON |
| [組織画像](/docs//api/organizations/images/logo/GET%20組織画像取得.md) | pcaid-profilebucket | /storage/organization/images | ユーザーデータ <br/> 署名付きURL発行先 | ◯ | ◯ | png, jpeg など |
| [プロフィール画像](/docs/api/users/account_id/images/profile/GET%20プロフィール画像取得.md) | pcaid-profilebucket | /storage/user/images | ユーザーデータ <br/> 署名付きURL発行先 | ◯ | ◯ | png, jpeg など |
| [ユーザーインポート結果](/docs/api/users/import/tasks/task_id/POST%20ユーザーインポート状態取得.md) | pcaid-profilebucket | /storage/user/import | 署名付きURL発行先 | ◯ | ◯ | BOM付きCSV |
| [システムログ](./log-policy.md) | pcaid-logbucket | /alb | ALB ログ | | | |
| 　〃 | pcaid-logbucket | /cloudfront | CloudFront アクセスログ | | | |
| 　〃 | pcaid-logbucket | /s3 | S3 サーバーアクセスログ | | | |
| 　〃 | pcaid-logbucket | /vpc | VPC フローログ | | | |
| 　〃 | aws-waf-logs-pcaid | | WAF ログ | | | |
| インフラ環境 | pcaid-pipelinebucket | | CDパイプライン | | | 未使用 |
| 　〃 | pcaid-wellknown | realms/pcaid/.well-known/openid-configuration | OpenID構成 | | | JSON |
| 　〃 | pcaid-ui | /account | アカウント設定 <br/>（フロント） | | | html, css, js など |
| 　〃 | pcaid-ui | /orgs | 管理コンソール <br/>（フロント） | | | html, css, js など |
| 　〃 | pcaid-ui | /orgs-new | 組織作成 <br/>（フロント） | | | html, css, js など |
| 　〃 | pcaid-ui-build | | フロントビルド成果物 | | | html, css, js など |
| 　〃 | pcaid-back | | Lambda関数 ビルド成果物パッケージ | | | zip |
