---
sidebar_position: 1
---

# 1. 管理API - api/admin/v1

- すべての管理対象データを扱うことができる
  - クライアントクレデンシャルによりアクセスを制限する
  - 特別に許可したクライアントのみが利用でき、現時点（2024/11）では以下のシステムのみアクセスを認めている
    - PCA Hub
    - 社内システム（情シス管轄）
- [管理 API オーソライザー（pcaid-authorizer）](/docs/common-dev/api-authorizers.md#1-管理-api-オーソライザーpcaid-authorizer)

| # | スコープ | URLパス | HTTPメソッド | API名 |
| ---: | --- | --- | --- | --- |
| 1 | 組織管理 | /organizations | GET | [組織取得](./organizations/GET%20組織取得.md) |
| 2 | 　〃 | 　〃 | POST | [組織作成](./organizations/POST%20組織作成.md) |
| 3 | 　〃 | 　〃 | PUT | [組織更新](./organizations/PUT%20組織更新.md) |
| 4 | 　〃 | /organizations/images/logo | GET | [組織画像取得](./organizations/images/logo/GET%20組織画像取得.md) |
| 5 | 　〃 | 　〃 | PUT | [組織画像アップロード](./organizations/images/logo/PUT%20組織画像アップロード.md) |
| 6 | 　〃 | /organizations/organization_reservations/{organization_name} | POST | [組織予約](./organizations/organization_reservations/organization_name/POST%20組織予約.md) |
| 7 | 　〃 | /organizations/organization_reservations/{organization_name}/status | GET | [組織予約状態](./organizations/organization_reservations/organization_name/status/GET%20組織予約状態.md) |
| 8 | 　〃 | /organizations/organization_reservations/{organization_name}/rename | PUT | [組織予約変更](./organizations/organization_reservations/organization_name/rename/PUT%20組織予約変更.md) |
| 9 | 　〃 | /organizations/organization_reservations/{organization_name}/remove | POST | [組織予約削除](./organizations/organization_reservations/organization_name/remove/POST%20組織予約削除.md) |
| 10 | 　〃 | /organizations/rename | PUT | [組織名変更](./organizations/rename/PUT%20組織名変更.md) |
| 11 | 　〃 | /organizations/reset | POST | [組織削除](./organizations/reset/POST%20組織削除.md) |
| 12 | 　〃 | /contracts/update | POST | [サービス契約キャッシュ更新](./contracts/update/POST%20サービス契約キャッシュ更新.md) |
| 13 | サービス区画管理 | /organizations/service_partitions | POST | [組織内サービス区画追加](./organizations/service_partitions/POST%20組織内サービス区画追加.md) |
| 14 | 　〃 | /organizations/service_partitions | GET | [組織内サービス区画一覧取得](./organizations/service_partitions/GET%20組織内サービス区画一覧取得.md) |
| 15 | 　〃 | /organizations/service_partitions/contract | PUT | [組織内サービス区画の顧客更新](./organizations/service_partitions/contract/PUT%20組織内サービス区画の顧客更新.md) |
| 16 | 　〃 | /organizations/service_partitions/remove | POST | [組織内サービス区画削除](./organizations/service_partitions/remove/POST%20組織内サービス区画削除.md) |
| 17 | ユーザー管理 | /users | GET | [ユーザー一覧取得](./users/GET%20ユーザー一覧取得.md) |
| 18 | 　〃 | 　〃 | POST | [ユーザー作成](./users/POST%20ユーザー作成.md) |
| 19 | 　〃 | /users/import | POST | [ユーザーインポート](./users/import/POST%20ユーザーインポート非同期.md) |
| 20 | 　〃 | /users/import/tasks/{task_id} | POST | [ユーザーインポート状態取得](./users/import/tasks/task_id/POST%20ユーザーインポート状態取得.md) |
| 21 | 　〃 | /users/export | POST | [ユーザーエクスポート](./users/export/POST%20ユーザーエクスポート.md) |
| 22 | 　〃 | /users/{account_id} | GET | [ユーザー取得](./users/account_id/GET%20ユーザー取得.md) |
| 23 | 　〃 | 　〃 | PUT | [ユーザー更新](./users/account_id/PUT%20ユーザー更新.md) |
| 24 | 　〃 | /users/{account_id}/invite | POST | [招待メール送信](./users/account_id/invite/POST%20招待メール送信.md) |
| 25 | 　〃 | /users/{account_id}/verify | POST | [メールアドレス検証メール送信](./users/account_id/verify/POST%20メールアドレス検証メール送信.md) |
| 26 | 　〃 | /users/{account_id}/images/profile | GET | [プロフィール画像取得](./users/account_id/images/profile/GET%20プロフィール画像取得.md) |
| 27 | 　〃 | 　〃 | PUT | [プロフィール画像アップロード](./users/account_id/images/profile/POST%20プロフィール画像アップロード.md) |
| 28 | 　〃 | /users/{account_id}/lifecycle/suspend | POST | [ユーザー無効化](./users/account_id/lifecycle/suspend/POST%20ユーザー無効化.md) |
| 29 | 　〃 | /users/{account_id}/lifecycle/unsuspend | POST | [ユーザー無効化解除（有効化）](./users/account_id/lifecycle/unsuspend/POST%20ユーザー無効化解除.md) |
| 30 | 　〃 | /users/{account_id}/remove | POST | [ユーザー削除](./users/account_id/remove/POST%20ユーザー削除.md) |
| 31 | 　〃 | /users/account_registrations/{email}/status | GET | [ユーザーアカウント登録状態確認](./users/account_registrations/GET%20ユーザーアカウント登録状態確認.md) |
| 32 | サービス区画管理 | /users/{account_id}/service_partitions | POST | [ユーザーへのサービス区画追加](./users/account_id/service_partitions/POST%20ユーザーへのサービス区画追加.md) |
| 33 | 　〃 | /users/{account_id}/service_partitions/remove | POST | [ユーザーからのサービス区画削除](./users/account_id/service_partitions/remove/POST%20ユーザーからのサービス区画削除.md) |
| 34 | サービスロール管理 | /users/{account_id}/service_roles | POST | [ユーザーロールの割り当て](./users/account_id/service_roles/POST%20ユーザーロールの割り当て.md) |
| 35 | 　〃 | /users/{account_id}/service_roles/unassign | POST | [ユーザーロールの割り当て解除](./users/account_id/service_roles/unassign/POST%20ユーザーロールの割り当て解除.md) |
| 36 | Hubテナント状態管理 | /hub_authn_switchings | GET | [Hubテナント移行状態取得](./hub_authn_switchings/GET%20Hubテナント移行状態取得.md) |
| 37 | 　〃 | 　〃 | PUT | [Hubテナント移行状態更新](./hub_authn_switchings/PUT%20Hubテナント移行状態更新.md) |
| 38 | 　〃 | /hub_authn_switchings/users/prepare | PUT | [Hubテナント移行開始準備](./hub_authn_switchings/prepare/PUT%20Hubテナント移行開始準備.md) |
| 39 | 　〃 | /hub_authn_switchings/users/convert | POST | [Hubユーザー情報コンバート](./hub_authn_switchings/users/convert/POST%20Hubユーザー情報コンバート.md) |
| 40 | 　〃 | /hub_authn_switchings/users/search | POST | [Hubユーザー移行状態検索](./hub_authn_switchings/users/search/POST%20Hubユーザー移行状態検索.md) |
| 41 | 　〃 | /hub_authn_switchings/summary | GET | [Hubテナント移行状態全体サマリー](./hub_authn_switchings/summary/GET%20Hubテナント移行状態全体サマリー.md) |
| 42 | 呼び出し不可　　　　　 | /event_hook | POST | Keycloak イベント処理 |
