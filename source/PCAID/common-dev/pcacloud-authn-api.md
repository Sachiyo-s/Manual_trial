---
sidebar_position: 23
---

# 23. PCAクラウド 認証API

## 通信方式

- 単純なREST API ではなく、SOAPプロトコルによる通信となっているため、専用ライブラリが必要です

## 接続先環境

- [PCA ID 外部システム接続先環境](./pcaid-external-accounts-list.md)

## サービス認証（login2）

IDとパスワードを使用してログインします。  
認証に成功した場合、認証用のCookieの有効期間を更新して返します。

### リクエスト

| パラメータ | 名前 | 必須 | データ型 | 備考 |
| --- | --- | :---: | --- | --- |
| identity | 識別情報 | - | string | 値は `null` 固定とする |
| parameter | ログインパラメータ | ◯ | string | パラメータ詳細は下記の表を参照 |

#### ログインパラメータ

- キー（key）と値（value）を「key=value」の形式でカンマ区切りで連結する
- キーや値に含まれる「,」は「,,」に置換する

```text
partner=PCA,id=12345678,pass=pca@2024,appdetail=orgs-new
```

| パラメータ | 名前 | 必須 | データ型 | 備考 |
| --- | --- | :---: | --- | --- |
| partner | パートナー | ◯ | string | 値は `PCA` 固定とする |
| id | サービスユーザーID | ◯ | string | 契約ID相当 |
| pass | パスワード | ◯ | string | 平文とする |
| appdetail | アプリケーション詳細情報 | ◯ | string | 値は `orgs-new` 固定とする |

### レスポンス

| パラメータ | 名前 | データ型 | 値の例 |
| --- | --- | --- | --- |
| applicationServiceUrl | アプリケーションの接続先URL | string | <https://app.pcacloud.jp/...> |
| certCookie | 認証クッキー | string | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx |
| certDate | 認証日付（YYYYMMDD） | int | 20241025 |
| remainderTime | サービス終了までの残り時間（分単位） | short | 0 |
| serviceUserID | サービスユーザーID | string | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx |
| certTokenID | 認証トークンID | string | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx |
| certInnerAddress | 認証サーバー内部アドレス | string | 172.16.0.1 |
| serviceStartTime | サービス開始時間（HHMM） | int | 0 |
| serviceEndTime | サービス終了時間（HHMM） | int | 0 |
| contractTypeName | 契約タイプ名 | string | Type12 |
| maxDBSize | 利用可能な最大DBサイズ(MB) | int | 1024 |
| status | ログイン結果の状態 | int | 1: ログインに成功 <br/> 3: アプリサーバー利用不可 <br/> 4: サービス利用時間外 <br/> 5: サービスユーザーが見つからない |
| validateSession | セッションが有効かどうか | bool | true |
