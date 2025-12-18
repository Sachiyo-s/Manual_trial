---
sidebar_position: 24
---

# 24. PCAサブスク 認証API

## 環境

- [PCA ID 外部システム接続先環境](./pcaid-external-accounts-list.md)

## トークン発行（/auth/token）

認証トークン（JWTトークン）を作成します。  
新しくトークンを作成しても、既存のトークンは無効化されません。  
成功したら HTTP ステータスコード 200 を返します。  

- アカウントロック機能
  - ログインに失敗した際、所定の試行回数を超えた失敗が続くと、一定期間ログイン出来ないようにします。
  - ロックアウトの閾値は、設定情報に従います。
  - 例）最大試行回数「5」のとき
    - 1～5回目 → パスワード不一致のエラー
    - 6回目～ → アカウントロックのエラー

### リクエスト

| パラメータ | 名前 | 必須 | データ型 | 備考 |
| --- | --- | :---: | --- | --- |
| grantType | 認証種類 | ◯ | string | `password` : 契約パスワードでの認証 <br/> `refresh_token`: トークンの更新 |
| id | 契約ID | - | string | 認証種類（grantType）： `password`で使用 |
| password | パスワード | - | string | 認証種類（grantType）： `password`で使用 |
| accessToken | アクセストークン | - | string | 有効期限切れでもOK <br/> 認証種類（grantType）： `refresh_token`で使用 |
| refreshToken | リフレッシュトークン | - | string | 有効期限切れ不許可 <br/> 認証種類（grantType）： `refresh_token`で使用 |

### 成功レスポンス

| パラメータ | 名前 | データ型 | 値の例 |
| --- | --- | --- | --- |
| accessToken | アクセストークン | string | xxxxxxxxxx |
| refreshToken | リフレッシュトークン | string | xxxxxxxxxx |
| expiresIn | アクセストークンの有効期限（秒） | int | 3600 |

### エラーレスポンス

| パラメータ | 名前 | データ型 | 値の例 |
| --- | --- | --- | --- |
| errorCode | エラーコード | string | WEBLS401101 |
| errorMessage | エラーメッセージ | string | アクセトークンの有効期限が切れています。 |
| status | HTTPステータスコード | string | 400 |
| traceId | トレースID | string | 80000002-0000-f300-b63f-84710c7967bb |
