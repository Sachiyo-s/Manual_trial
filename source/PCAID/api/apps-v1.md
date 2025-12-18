---
sidebar_position: 4
---

# 4. アプリAPI - api/apps/v1

- [PCA ID 連携クライアント](/docs/common-dev/apps-integration-clients.md)向けに提供するAPIとなる
- 管理者によって許可された組織関連データのみを対象とする
  - アクセストークンに含まれる[組織管理者や製品責任者のロール](/docs/common/ロール（役割）.md)により対象データを制限する
  - 組織管理者によって許可されたアプリ（OIDC/OAuthクライアントのセット）のみが対象組織へアクセスできるように制限する
- [アプリ向けオーソライザー（pcaid-cloud-cloudAuthorizer）](/docs/common-dev/api-authorizers.md#4-%E3%82%A2%E3%83%97%E3%83%AA%E5%90%91%E3%81%91-api-%E3%82%AA%E3%83%BC%E3%82%BD%E3%83%A9%E3%82%A4%E3%82%B6%E3%83%BCpcaid-cloudauthorither)

| #  | スコープ           | URLパス                                        | HTTPメソッド | API名                                                                      |
| -: | ---                | ---                                           | ---         | ---                                                                        |
| 1  | アクセス管理        | /ensure                                       | POST        | [APIリクエスト条件確認](./ensure/POST%20APIリクエスト条件確認.md) |
| 2  | プロフィール情報    | /me                                            | GET        | [ユーザー情報取得](./me/GET%20ユーザー情報取得.md) |
| 3  | 　〃               | /me/images/profile                             | GET        | [プロフィール画像取得](./users/account_id/images/profile/GET%20プロフィール画像取得.md) |
| 4  | 組織管理           | /organizations                                 | GET         | [組織取得](./organizations/GET%20組織取得.md) |
| 5  | 　〃               | /organizations/images/logo                     | GET        | [組織画像取得](./organizations/images/logo/GET%20組織画像取得.md) |
| 6  | ユーザー管理        | /users                                        | GET         | [ユーザー一覧取得](./users/GET%20ユーザー一覧取得.md) |
| 7  | 　〃               | /users/query                                   | GET        | ユーザー一覧検索 |
| 8  | 　〃               | /users/{account_id}                            | GET        | [ユーザー取得](./users/account_id/GET%20ユーザー取得.md) |
| 9  | 　〃               | /users/{account_id}/images/profile             | GET        | [プロフィール画像取得](./users/account_id/images/profile/GET%20プロフィール画像取得.md) |
| 10 | サービス管理        | /users/{account_id}/service_roles              | POST       | [ユーザーロールの割り当て](./users/account_id/service_roles/POST%20ユーザーロールの割り当て.md) |
| 11 | 　〃               | /users/{account_id}/service_roles/unassign     | POST       | [ユーザーロールの割り当て解除](./users/account_id/service_roles/unassign/POST%20ユーザーロールの割り当て解除.md) |
| 12 | サービス契約        | /contracts/{service_partition}                 | GET        | [サービス契約情報の取得](./contracts/service_partition/GET%20サービス契約情報の取得.md) |
| 13 | 　〃               | /contracts/update                              | POST       | [サービス契約キャッシュ更新](./contracts/update/POST%20サービス契約キャッシュ更新.md) |
