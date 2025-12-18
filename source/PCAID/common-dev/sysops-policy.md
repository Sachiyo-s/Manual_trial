---
sidebar_position: 50
---

# 50. システム運用方針

[2024/08/28 AWSディスカッション · pca\-idp/pcaid · Discussion \#809](https://github.com/pca-idp/pcaid/discussions/809)

## 特権ユーザー運用（ベースライン）

- 特権ユーザー
  - AWSアカウントのルートユーザー
  - Keycloak管理コンソールの管理者ユーザー
    - <https://id.pca.jp/admin>
- CBC のみ利用可能

### 特権ユーザー保護

- MFAを必須とする
  - 可能ならパスキーを使用する
- 接続元を可能な限り制限する
  - [WAF ルール](./waf-rules.md)
- ユーザーの利用をすべて記録する
- ログイン通知メールを送信する

## Aurora PostgreSQL バージョン利用方針

- [Aurora PostgreSQL 長期サポート \(LTS\) リリース \- Amazon Aurora](https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Updates.LTS.html)
- 最新または3番目のLTSバージョンを採用する
  - 15.10 (15.10.x)
  - 14.6 (14.6.x)
  - 13.9 (13.9.x)

## Kaycloak バージョン利用方針

- [keycloak.org](https://www.keycloak.org/)
- 最新または3番目までのバージョンを採用する
  - v26 (26.x.x)
  - v24 (24.x.x)
  - v23 (23.x.x)
- パッチバージョン（セキュリティパッチ）は単独で適用する
  - PCA ID バージョンの更新を待たない
  - 自動テストを実施する

## Node.js バージョン利用方針

- [Node\.js — Run JavaScript Everywhere](https://nodejs.org)
- 最新または2番目のLTSバージョンを採用していく
  - v22
  - v20
- PCA ID バージョンアップのときに最新化していく

## バックアップ

AWS Backup の活用を中心に考える

### 1. 定期バックアップ

- 目的：DR対策と顧客トラブル対応
- 対象リソース：Aurora, S3, DynamoDB
  - Aurora クラスター → フル＋増分バックアップ
  - S3 バケット → フル＋増分バックアップ
  - DynamoDB テーブル → フルバックアップ
- 遠隔コピー：同一アカウントの大阪リージョンへコピー
  - 任意時刻でリストアして、顧客単位で調査・回復措置を実施可能にする
    - リソース名には例えば -prod-copy サフィックスを使う
  - 復元テストの活用を検討する
- 実行頻度：毎日（夜間 2:00）
- 保持期間：90日間
  - AWS仕様としては最大100年

### 2. 継続的バックアップ

- 目的：致命的システム障害からの回復
- 対象リソース：Aurora, S3
  - DynamoDBは AWS Backup による管理不可なので単独管理する
- 保持期間：10日間
  - AWS仕様としては最大35日

### AWSバックアップ参考情報

- [Amazon DynamoDBにおけるバックアップ戦略 \| Amazon Web Services ブログ](https://aws.amazon.com/jp/blogs/news/backup-strategies-for-amazon-dynamodb/)
- [AWS Backup で S3 の継続的または定期的なバックアップを作成する \| AWS re:Post](https://repost.aws/ja/knowledge-center/backup-s3-continuous-periodic-backups)
- [Amazon AuroraのバックアップをAWS Backupで取得する方法と動作 \#AWSBackup \- Qiita](https://qiita.com/n_53_xx/items/376ddb275da7cf7a46fd)

## モニタリング

### 役割分担

#### 開発（FW+Hub）：CloudWatchへログデータを転送する

- Terraformで構築する
- モニタリング用のログデータの保持期間は３ヶ月以上とする

#### 運用（CBC）：CloudWatchのログデータを活用する（アラーム等）

- AWSコンソール等で構築する

### Aurora

- 有効化する機能（Terraform）
  - RDS拡張モニタリング
  - RDSパフォーマンスインサイト
- メトリクスの対象となる主要項目
  - CPU
  - メモリ
  - ディスク容量
  - ネットワーク
  - データベース接続数

## ECS

- 有効化する機能（Terraform）
  - ECSコンテナインサイト
  - GuardDuty Agent
- メトリクスの対象となる主要項目
  - サービス（タスク平均）のCPU
  - サービス（タスク平均）のメモリ
  - 実行タスク数

## ECR

- 有効化する機能（Terraform）
  - ECRベーシック
  - ECRインスペクター
- 開発環境では yamory も併用する

## Lambda

- 有効化する機能（Terraform）
  - CloudWatch Lambda インサイト
    - Lambda拡張モニタリング
- メトリクスの対象となる主要項目
  - CPU
  - メモリ
  - 実行回数
  - 実行時間
  - エラー数

## DynamoDB

- 有効化する機能（手動）
  - CloudWatchアラーム
  - CloudWatch Contributor Insights for DynamoDB は必要なら使う
- メトリクスの対象となる主要項目
  - 持続的な読み取り・読み取りスロットル
    - ReadThrottleEvents / ConsumedReadCapacityUnits
    - WriteThrottleEvents / ConsumedWriteCapacityUnits
  - キャパシティユニットの消費率
    - ConsumedReadCapacityUnits / ProvisionedReadCapacityUnits
    - ConsumedWriteCapacityUnits / ProvisionedWriteCapacityUnits

## SQS

- スループット
- デッドレターキューへのメッセージ件数

### モニタリング参考情報

- [運用上の認識を高めるための Amazon DynamoDB のモニタリング \| Amazon Web Services ブログ](https://aws.amazon.com/jp/blogs/news/monitoring-amazon-dynamodb-for-operational-awareness/)

## 設定変更手順

### AWS

- Terraform
- AWSコンソールから手動設定

### Keycloak

- Keycloak設定（環境別）
  - pcaid-keycloak リポジトリ
- WinDiff等でJSON設定ファイルを目視で比較する

### 環境パラメーター

- ◯ SSMパラメーターストアで管理する？
  - SecretManager との使い分けを検討する
- ✕ 社内ネットワーク内で管理する？
