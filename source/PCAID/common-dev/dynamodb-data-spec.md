---
sidebar_position: 5
---

# 5. DynamoDB データ仕様

## テーブル一覧

### Ver1.0

| テーブル名 | パーティションキー | ソートキー | インデックス | ストリーム | TTL | 用途 |
| --- | --- | --- | :---: | :---: | :---: | --- |
| pcaid-hub_watch_log_constraints_table | constraint_item | | | | | PCA Hub 制約条件 |
| pcaid-import_users | task_id | sequence_id | | ◯ | | ユーザーインポートタスク |
| pcaid-login_event_record_table | account_id | | | | | 最終ログイン記録 |
| pcaid-organization_image | organization_id | | | | | 組織画像の保存 |
| pcaid-portal-audit_log_execution | task_id | | | ◯ | | 操作ログCSVダウンロード |
| pcaid-reserve_organization | organization_name | | | | | 組織名の予約 |
| pcaid-sqs_queue_event_delivery_record | message_id | | | | 180s | SQSトリガーLambdaの同時実行制御 |
| pcaid-user_profile_image | user_id | | | | | プロフィール画像の保存 |
| pcaid-users_convert_status_table | service_partition | | | | | HubユーザーID移行の管理 |

### Ver2.0

| テーブル名 | パーティションキー | ソートキー | インデックス | ストリーム | TTL | 用途 |
| --- | --- | --- | :---: | :---: | :---: | --- |
| pcaid-service_client_master_table | service_kind | | | | | サービス種類のクライント定義 |
| pcaid-new-orgs-auth_url_master | service_kind | | | | | サービス種類の認証エンドポイント |
| pcaid-new-orgs-new_org_prepare | receipt_session_id | | | | 1h | 組織作成セッション |
| pcaid-new-orgs-connect_code | service_contract_id | | connect_code | | | 連携コード管理 |
| pcaid-portal-contract_info | contract_id | | service_partition  | | 48h | サービス契約キャッシュ |
| pcaid-portal-service_info_customer_test | contract_id | | | | | サービス契約へのダミー顧客ID |
| pcaid-portal-move_service_partition | organization_id | | move_organization_id  | | 7d | 組織統合リクエスト |
| pcaid-portal-move_service_partition_status | task_id | | | ◯ | | 組織統合タスク状態 |
| pcaid-portal-add_all_users_tasks | task_id | | | ◯ | | サービス区画内ユーザーの一括追加 |
| pcaid-apps_watch_log_constraints_table | constraint_item | | | | | アプリAPI利用サービスの制約条件 |
| pcaid-portal-get_hard_bounce_task | task_id | | organization_id | | | バウンスメールの記録 |

## pcaid-hub_watch_log_constraints_table テーブル

- 用途
  - PCA Hub 制約条件
- キャパシティーモード
  - オンデマンド

| 属性名（システム） | 属性名 | 値の例 | 備考 |
| --- | --- | --- | --- |
| constraint_item (String) | PCA Hub の制約条件 | ip_list | 現状IPリストのみ（2024/10） |

### 項目（1件固定）

| constraint_item (String) | 値の例 | 備考 |
| --- | --- | --- |
| ip_list | [ { "S" : "12.23.34.56" }, { "S" : "23.34.45.56" } ] | PCA ID を呼び出す PCA Hub の 固定IPリスト |

## pcaid-import_users テーブル

- 用途
  - ユーザーインポートタスク
- キャパシティーモード
  - オンデマンド
- ストリーム
  - Lambda トリガー
    - pcaid-importUsersEventConsumer
    - シャードあたりの同時バッチ: 1
    - バッチウィンドウ: 1
    - バッチサイズ: 1
    - 再試行: 3
    - 障害時の送信先：なし
    - フィルタ条件: `INSERT`
- API仕様
  - [ユーザーインポート（非同期） POST /users/import](/docs/api/users/import/POST%20ユーザーインポート非同期.md)
  - [ユーザーインポート状態取得 POST /users/import/tasks/\{task\_id\}](/docs/api/users/import/tasks/task_id/POST%20ユーザーインポート状態取得.md)

| 属性名（システム） | 属性名 | 値の例 | 備考 |
| --- | --- | --- | --- |
| task_id | タスクID | 1831b1d0-7ccb-11ef-88c2-fdd949330946 | ユーザーインポート処理の実行ごとに付番する |
| sequence_id | |  |  |
| created_at | タスク作成日時 | 1727439720288 | UNIX時間 |
| created_by | タスク作成アカウント | a5a198bb-53bc-4e98-9f91-3b72d8e65f4a | PCAアカウントの `account_id` |
| csv_file_name | CSVファイル名 | import-user.csv | |
| external_customer_id | 顧客ID | 12345678 | |
| failed_user_count | インポート失敗ユーザー数 | 1 | |
| filed_users | インポートエラー情報 | エラー配列（JSON） | |
| imported_user_count | インポート完了ユーザー数 | 999 | |
| imported_users | インポート完了ユーザー | ユーザー配列（JSON）| |
| last_seq_end_at | | | |
| organization_id | 組織ID | 47d8e3dc-f2eb-4483-9384-098d10a1d2d4 | |
| organizatoin_name | 組織名 | fw-test1 | |
| send_invitation_mail | 招待メール送信フラグ | true | |
| service_partition | サービス区画 | pca.hub.test-jddqiarbpa | |
| task_start_at | タスク開始日時 | 1727159068333 | UNIX時間 |
| task_end_at | タスク終了日時 | 1727159072051 | UNIX時間 |
| task_run_by | タスク実行クライアント | fdcbd79f-56f4-49fd-bf8d-d3061cc2eae9 | バックグラウンドジョブ用のクライアントID |
| task_status | タスク状態 | finished | 以下のいずれかとなる <br/> `importing`: 実行中 <br/> `finished`: 正常終了 <br/> `stopped`: 異常終了 |
| total_count | インポート処理ユーザー件数 | 1000 | |
| trace_id | トレースID | 00-1b0bd3f2251cce3bfc0bcf84b8a2fec3-a4575542f751662c-00 | |

## pcaid-login_event_record_table テーブル

- 用途
  - 最終ログイン記録
- キャパシティーモード
  - オンデマンド

| 属性名（システム） | 属性名 | 値の例 | 備考 |
| --- | --- | --- | --- |
| account_id | アカウントID | 66c0cbff-21e0-4eba-8b4a-9f354d6a84f6 | PCAアカウント |
| login_time | ログイン日時 | 1721282279238 | UNIX時間 |
| refresh_time | 更新日時 | 1721282280213 | UNIX時間 |

## pcaid-organization_image テーブル

- 用途
  - 組織画像の保存
- キャパシティーモード
  - オンデマンド

| 属性名（システム） | 属性名 | 値の例 | 備考 |
| --- | --- | --- | --- |
| organization_id | 組織ID | 482aa102-821b-4574-8abf-ab6252852b5d | |
| s3_key | S3キー | storage/organization/images/482aa102-821b-4574-8abf-ab6252852b5d.png | |

## pcaid-portal-audit_log_execution テーブル

- 用途
  - 操作ログCSVダウンロード
- キャパシティーモード
  - オンデマンド
- ストリーム
  - Lambda トリガー
    - pcaid-portal-getOperationLogsCSVComsumer
    - シャードあたりの同時バッチ: 1
    - バッチウィンドウ: 1
    - バッチサイズ: 1
    - 再試行: 3
    - 障害時の送信先：なし

| 属性名（システム） | 属性名 | 値の例 | 備考 |
| --- | --- | --- | --- |
| task_id | タスクID | d8e7fd0a-aa5b-4b79-83bd-e5a696bb76e7 | |
| organization_id | 組織ID | 7e46db69-38ae-434a-bc09-c81f6fc05343 | |
| status | タスク状態 | SUCCEEDED | `SUCCEEDED`: 成功 <br/> `FAILED`: 失敗 |
| url | 署名付きURL | <https://id.pca.jp/storage/audit_log_download/...2024-10-03....csv?Expires=1727..&Key-Pair-Id=K1...L&Signature=wiz36....UJbmy> | |

## pcaid-reserve_organization テーブル

- 用途
  - 組織名の予約
- キャパシティーモード
  - オンデマンド

| 属性名（システム） | 属性名 | 値の例 | 備考 |
| --- | --- | --- | --- |
| organization_name | 組織名 | fwtest120 | |
| reserved_at | 予約日時 | 1715237086083 | UNIX時間 |
| status | 予約状態 | reserved | `reserved`: 予約済み <br/> `inUse`: 使用中 <br/> `replaced`: 変更済み |

## pcaid-sqs_queue_event_delivery_record テーブル

- 用途
  - SQSトリガーLambdaの同時実行制御
- キャパシティーモード
  - オンデマンド
- TTL有効
  - 180秒
- [PCA Hub Webhook 呼び出し仕様](./hub-webhook-integration.md)

| 属性名（システム） | 属性名 | 値の例 | 備考 |
| --- | --- | --- | --- |
| message_id | メッセージID | c4ef31db-9702-4a59-841b-96d29059aef7 | |
| expired_at | TTL | 1728860809 | UNIX時間 |

## pcaid-user_profile_image テーブル

- 用途
  - プロフィール画像の保存
- キャパシティーモード
  - オンデマンド

| 属性名（システム） | 属性名 | 値の例 | 備考 |
| --- | --- | --- | --- |
| account_id | アカウントID | 5ceece7d-0f72-4da6-87cb-1e9604b8baf8 | PCAアカウント |
| s3_key | S3キー | storage/user/images/5ceece7d-0f72-4da6-87cb-1e9604b8baf8.png | |

## pcaid-users_convert_status_table テーブル

- 用途
  - HubユーザーID移行の管理
- キャパシティーモード
  - オンデマンド
- API仕様
  - [Hubテナント移行開始準備 PUT /hub\_authn\_switchings/prepare](/docs/api/hub_authn_switchings/prepare/PUT%20Hubテナント移行開始準備.md)
  - [Hubテナント移行状態更新 PUT /hub\_authn\_switchings](/docs/api/hub_authn_switchings/PUT%20Hubテナント移行状態更新.md)
  - [Hubユーザー情報コンバート POST /hub\_authn\_switchings/users/convert](/docs/api/hub_authn_switchings/users/convert/POST%20Hubユーザー情報コンバート.md)

| 属性名（システム） | 属性名 | 値の例 | 備考 |
| --- | --- | --- | --- |
| service_partition | サービス区画 | pca.hub.fw-pcaidtest01 | |
| service_partition_id | サービス区画ID | eee6db21-0838-40d8-83f1-cf4261d5ba17 | KeycloakグループID |
| switching_status | 移行状態 | by_tenant_convert_done | `not_started`: 移行前 <br/> `by_tenant_convert_running`: 移行中 <br/> `by_tenant_convert_failed`: 移行失敗 <br/> `by_tenant_convert_done`: 移行完了 |
| pcaid_released_at | PCA ID リリース日時 | 2024-08-23T06:33:00Z | ISO8601 |
| switching_start_at | 移行開始日時 | 2024-05-01T10:00:00Z | ISO8601 |
| switching_end_at | 移行終了日時 | 0001-01-01T00:00:00 | ISO8601 |
| failed_reason | 移行失敗理由 | unexpected |  |
| reminder_email | 移行案内メール設定 | {"send_start_from":"2500-12-11T16:00:00Z",<br/>"send_frequency":"5d",<br/>"latest_sent_at":"1868-09-08T00:00:00Z",<br/>"email_template_id":"template1"} | |
| features_limit | 機能制限設定 | {"limit_start_from":"2024-07-29T05:00:00Z",<br/>"limit_scope":"Users.Create"} |  |
