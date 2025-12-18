---
sidebar_position: 3
---

# 3. アカウントAPI - api/account/v1

## API一覧

- 自身の関連データのみを対象とする
  - アクセストークンにより対象データを制限する
- 自身のアカウント設定の更新には、Keycloak のプロフィール API を利用する
  - アカウント設定
    - ユーザー名（表示名）
    - 姓・名（＋フリガナ）
    - ログイン名
    - メールアドレス
  - セキュリティ設定
    - パスワード
    - バックアップ
    - パスキー
- [アカウント設定向けオーソライザー（pcaid-account-Authorizer）](/docs/common-dev/api-authorizers.md#2-アカウント設定向け-api-オーソライザーpcaid-accountauthorizer)

| # | スコープ | URLパス | HTTPメソッド | API名 |
| ---: | --- | --- | --- | --- |
| 1 | プロフィール情報 | /me | GET | [ユーザー情報取得](./me/GET%20ユーザー情報取得.md) |
| 2 | 　　〃 | 〃 | PUT | ユーザー情報更新 |
| 3 | 　　〃 | /images/profile | POST | [プロフィール画像アップロード](./users/account_id/images/profile/POST%20プロフィール画像アップロード.md) |
| 4 | 　　〃 | 〃 | GET | [プロフィール画像取得](./users/account_id/images/profile/GET%20プロフィール画像取得.md) |
| 5 | 所属組織情報 | /organizations | GET | [組織取得](./organizations/GET%20組織取得.md) |
| 6 | 　　〃 | /organizations/images/logo | GET | [組織画像取得](./organizations/images/logo/GET%20組織画像取得.md) |
| 7 | セキュリティ | /security | GET | 自身のセキュリティ設定取得 |
| 8 | 　　〃 | /security/passkey/{credentialId}/label | POST | 自身のパスキー名変更 |
| 9 | 　　〃 | /security/passkey/{credentialId}/remove | POST | 自身のパスキー削除 |
| 10 | サービス利用状況 | /service_usage | GET | 自身のサービス利用状況取得 |
