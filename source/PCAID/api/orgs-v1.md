---
sidebar_position: 2
---

# 2. 組織API - api/orgs/v1

- 管理者権限をもつ組織関連データのみを対象とする
  - アクセストークンに含まれる[組織管理者ロール](/docs/common/ロール（役割）.md)により対象データを制限する
- [管理コンソール向けオーソライザー（pcaid-portal-portalAuthorizer）](/docs/common-dev/api-authorizers.md#3-管理コンソール向け-api-オーソライザーpcaid-portalauthorither)

| # | スコープ | URLパス | HTTPメソッド | API名 |
| ---: | --- | --- | --- | --- |
| 1 | 組織管理 | /organizations | GET | [組織取得](./organizations/GET%20組織取得.md) |
| 2 | 　〃 | 〃 | PUT | [組織更新](./organizations/PUT%20組織更新.md) |
| 3 | 　〃 | /organizations/rename | PUT | [組織名変更](./organizations/rename/PUT%20組織名変更.md) |
| 4 | 　〃 | /organizations/images/logo | GET | [組織画像取得](./organizations/images/logo/GET%20組織画像取得.md) |
| 5 | 　〃 | 〃 | PUT | [組織画像アップロード](./organizations/images/logo/PUT%20組織画像アップロード.md) |
| 6 | 　〃 | /organizations/customer-id | GET | 同一顧客IDの組織一覧取得 |
| 7 | 　〃 | 〃 | POST | 組織の顧客ID更新 |
| 8 | 　〃 | /auth/url | GET | サービス認証URL取得 |
| 9 | 　〃 | /connect_code | POST | [連携コード発行](./connect_code/POST%20連携コード発行.md) |
| 10 | 　〃 | /contracts/update | POST | サービス契約更新 |
| 11 | サービス区画管理 | /organizations/service_partition | GET | 組織内サービス区画取得 |
| 12 | 　〃 | 〃 | POST | [組織内サービス区画追加](./organizations/service_partitions/POST%20組織内サービス区画追加.md) |
| 13 | 　〃 | /organizations/service_partition/remove | POST | [組織内サービス区画削除](./organizations/service_partitions/remove/POST%20組織内サービス区画削除.md) |
| 14 | 　〃 | /organizations/service_partition/connect_code | POST | [連携コード発行](./connect_code/POST%20連携コード発行.md) |
| 15 | 　〃 | /organizations/service_partition/users | GET | サービス区画内ユーザー取得 |
| 16 | 　〃 | 〃 | POST | サービス区画内ユーザー一括追加 |
| 17 | 　〃 | /organizations/service_partition/{task_id} | GET | サービス区画内ユーザー一括追加状況 |
| 18 | 組織統合管理 | /organizations/service_partition/move | GET | 組織統合リクエスト取得 |
| 19 | 　〃 | 〃 | POST | 組織統合リクエスト作成 |
| 20 | 　〃 | /organizations/service_partition/move/approval | POST | 組織統合（非同期） |
| 21 | 　〃 | /organizations/service_partition/move/{task_id} | POST | 組織統合の処理状況取得 |
| 22 | ユーザー管理 | /users | GET | [ユーザー一覧取得](./users/GET%20ユーザー一覧取得.md) |
| 23 | 　〃 | 〃 | POST | [ユーザー作成](./users/POST%20ユーザー作成.md) |
| 24 | 　〃 | /users/query | GET | ユーザー一覧検索 |
| 25 | 　〃 | /users/permitted | POST | ユーザーのサービス区画追加フラグ登録 |
| 26 | 　〃 | /users/{account_id} | GET | [ユーザー取得](./users/account_id/GET%20ユーザー取得.md) |
| 27 | 　〃 | 〃 | PUT | [ユーザー更新](./users/account_id/PUT%20ユーザー更新.md) |
| 28 | 　〃 | /users/{account_id}/iamges/profile | GET | [プロフィール画像取得](./users/account_id/images/profile/GET%20プロフィール画像取得.md) |
| 29 | 　〃 | 〃 | PUT | [プロフィール画像アップロード](./users/account_id/images/profile/POST%20プロフィール画像アップロード.md) |
| 30 | 　〃 | /users/{account_id}/init | GET | ログインユーザー情報 |
| 31 | 　〃 | /users/{account_id}/invite | POST | ユーザー招待 |
| 32 | 　〃 | /users/{account_id}/verify | POST | ユーザー検証メール送信 |
| 33 | 　〃 | /users/{account_id}/password/reset | POST | パスワードリセット |
| 34 | 　〃 | /users/{account_id}/sessions/reset | POST | ログイン状態リセット |
| 35 | 　〃 | /users/{account_id}/admin_roles | POST | 組織管理者権限の付与 |
| 36 | 　〃 | /users/{account_id}/admin_roles/unassign | POST | 組織管理者権限の取り消し |
| 37 | 　〃 | /users/{account_id}/lifecycle/suspend | POST | [ユーザー無効化](./users/account_id/lifecycle/suspend/POST%20ユーザー無効化.md) |
| 38 | 　〃 | /users/{account_id}/lifecycle/unsuspend | POST | [ユーザー無効化解除（有効化）](./users/account_id/lifecycle/unsuspend/POST%20ユーザー無効化解除.md) |
| 39 | 　〃 | /users/{account_id}/remove | POST | [ユーザー削除](./users/account_id/remove/POST%20ユーザー削除.md) |
| 40 | 　〃 | /users/import | POST | [ユーザーインポート](./users/import/POST%20ユーザーインポート非同期.md) |
| 41 | 　〃 | /users/import/tasks/{task_id} | POST | [ユーザーインポート状態取得](./users/import/tasks/task_id/POST%20ユーザーインポート状態取得.md) |
| 42 | 　〃 | /users/export | POST | [ユーザーエクスポート](./users/export/POST%20ユーザーエクスポート.md) |
| 43 | サービス管理 | /users/{account_id}/service_partitions | POST | [ユーザーへのサービス区画追加](./users/account_id/service_partitions/POST%20ユーザーへのサービス区画追加.md) |
| 44 | 　〃 | /users/{account_id}/service_partitions/remove | POST | [ユーザーからのサービス区画削除](./users/account_id/service_partitions/remove/POST%20ユーザーからのサービス区画削除.md) |
| 45 | 　〃 | /users/{account_id}/sp_manager_roles | POST | サービス責任者権限の付与 |
| 46 | サービス利用状況管理 | /service_usage | GET | サービス利用状況取得 |
| 47 | 　〃 | /service_usage/login_time | POST | 最終ログイン日時取得 |
| 48 | 　〃 | /service_usage/tasks | POST | サービス利用状況CSV取得開始 |
| 49 | 　〃 | /service_usage/tasks/{task_id} | POST | サービス利用状況CSV取得状況確認 |
