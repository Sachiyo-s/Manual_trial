---
sidebar_position: 5
---

# 5. 組織作成API - api/orgs-new/v1

- 組織作成を対象とする
  - OTP認証および受付セッションIDでアクセスを制限する
  - [組織作成 - 共通](/docs/apps/orgs-new/orgs-new-common.md)
- [組織作成向けオーソライザー（pcaid-new-orgs-newOrgsAuthorizer）](/docs/common-dev/api-authorizers.md#5-%E7%B5%84%E7%B9%94%E4%BD%9C%E6%88%90%E5%90%91%E3%81%91-api-%E3%82%AA%E3%83%BC%E3%82%BD%E3%83%A9%E3%82%A4%E3%82%B6%E3%83%BCpcaid-neworgauthorither)

| # | スコープ | URLパス | HTTPメソッド | API名 |
| ---: | --- | --- | --- | --- |
| 1 | 組織作成 | /organizations/prepare | POST | [組織作成準備](./organizations/prepare/POST%20組織作成準備.md) |
| 2 | 　〃 | /organizations/prepare | GET | [組織作成準備](./organizations/prepare/GET%20組織作成準備.md) |
| 3 | 　〃 | /auth/url | GET | サービス認証URL取得 |
| 4 | 　〃 | /connect_code | POST | [連携コード発行](./connect_code/POST%20連携コード発行.md) |
| 5 | 　〃 | /service_partition/check | PUT | サービス区画確認 |
| 6 | 　〃 | /organizations/with_admin | POST | 組織および管理者作成 |
| 7 | 　〃 | /contracts/update | POST | サービス契約更新 |
