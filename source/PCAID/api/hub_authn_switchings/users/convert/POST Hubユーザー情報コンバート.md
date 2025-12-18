# POST Hubユーザー情報コンバート

```http
/hub_authn_switchings/users/convert
```

## 説明

PCA Hubのユーザー移行として、PCA Hubユーザー情報を元にして、PCAアカウントとなるユーザーを作成します。

- 指定されたPCA Hubのサービス区画（`pca.hub.{tenant_name}`）から組織を特定する
- 別組織で作成済みのPCAアカウントに対して、このAPIによる作成要求があり得ることを想定する
- 移行中において、作成済みのPCAアカウントに対して、繰り返し作成要求があり得ることを想定する
- アカウントが作成済みかどうかはメールアドレスで判定する
  - このAPIではメールアドレスを変更できない
- アカウント状態 `account_status` は、アカウントの新規作成時なら `active` とし、作成済みなら変更しない
- メールアドレス状態 `email_status` は、アカウントの新規作成時なら `enable` とし、作成済みなら変更しない
- 割当済みロール一覧 `pcahub_roles` で指定されたPCA Hub テナント内の任意ロールは、次の扱いとする
  - 指定されたロールが存在しなければ作成する
  - このアカウントのユーザーに対して指定されたロールを割り当てる
  - `pca.hub.{tenant_name}/gs:admin` が割り当てられている場合は `pca.id.{organization_id}/admin` も割り当てる
    - `pca.hub.{tenant_name}/gs:admin` はPCA Hub テナント管理者を指す
    - `pca.id.{organization_id}/admin` は組織管理者を指し、組織作成時に既定ロールとして作成する
      - [組織作成API POST /organization](./../../../organizations/POST%20組織作成.md)
  - [ロール（役割）](/docs/common/ロール（役割）.md)
- AES-GCMの利用については次の仕様とする
  - AES-256-GCM アルゴリズムを使用する
  - PCA ID と PCAAuth で事前に共有鍵を共有する
  - nonce(IV) は専用アルゴリズムで都度ランダム値を計算する
  - [AAD（追加認証データ）](https://qiita.com/jkomatsu/items/baf25bf86f815f6da8b3#aad)  は、サービス区画を利用する（例：pca.hub.tenant1）
  - 文字列はすべてUTF-8のバイト列にする
- API呼び出しログは、成功・失敗とともに、少なくともログイン名 `login_name` を記録する
- 移行先となる新しい組織は、移行完了まで管理画面からアクセスできないものとする
  - 管理画面は移行状態でアクセス可能かを判定する
- 以下の場合はエラーとする
  - 存在しないPCA Hubのサービス区画が指定された

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | [OAuthアクセストークン](/docs/common-dev/oidc-oauth-token-rules.md) | required | String | [管理APIの認可方法](/docs/common-dev/pcaid-management-api-granttype.md) |
| X-PCA-service-partition | [サービス区画](/docs/common/サービス区画.md) | required | String |  ・呼び出し元のサービス区画を指定する |

```http
Authorization: Bearer xxxx-xxxx-xxxx-xxxx
```

```http
X-PCA-service-partition: pca.hub.tenant1
```

### BODY

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| nonce | ノンス | requried | String | ・AES-GCMのnonceとして利用するランダム値 <br/>・Base64エンコード文字列とする |
| tag | 認証タグ | requried | String | ・AES-GCMのtagとして利用する認証値 <br/>・Base64エンコード文字列とする |
| encrypted_data | 暗号化したユーザー情報 | requried | String | ・下記のユーザー情報のJSONデータをAES-GCMにより暗号化した値 <br/>・Base64エンコード文字列とする |

```json
{
  "nonce": "...",
  "tag": "...",
  "encrypted_data": "..."
}
```

### encrypted_data の暗号化前データ

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| login_name | ログイン名 | requried | String | ・[PCAアカウント](/docs/common/PCAアカウント.md) の「ログイン名」を参照 <br/> ・内部的には組織IDとセットで管理する <br/> ・未所属の組織IDならユーザー属性の追加となる |
| password_hash  | パスワードのハッシュ値 | requrired | String | ・salt はバイト列の先頭に含まれている <br/> ・Base64エンコード文字列とする |
| backup_code | バックアップコード | required | String | ・セミコロン区切りで1～10個 |
| email | メールアドレス | requried | String |・作成済みアカウントに対する変更不可 |
| preferred_username | 表示名 | requried | String |・アカウント作成済みで、別組織に所属済みなら上書きしない <br/> ・例外として、どのサービスも利用していない場合は上書きする |
| family_name | 姓 | requried | String |・上書きルールは `preferred_username` と同様 |
| given_name | 名 | optional | String |・上書きルールは `preferred_username` と同様 |
| family_kana | 姓カナ | requried | String |・上書きルールは `preferred_username` と同様 |
| given_kana | 名カナ | optional | String |・上書きルールは `preferred_username` と同様 |
| pcahub_roles | PCA Hub 割当済みロール一覧 | required | String | ・ロール定義は組織作成と同時に作成する <br/>・ライセンスの割当状況把握の利用を想定する |

```json
{
  "longin_name": "yamada",
  "password_hash": "password_hash",
  "backup_code": "664870ec;c3b877bf;11cbd178",
  "email": "email",
  "preferred_username": "総務部_山田太郎",
  "family_name": "山田",
  "given_name": "太郎",
  "family_kana": "ヤマダ",
  "given_kana": "タロウ",
  "pcahub_roles": [
    "pca.hub.tenant1/gs:admin",
    "pca.hub.tenant1/d:users"
  ]
}
```

[AspNetCore Identityにおけるパスワードのハッシュ値計算](https://github.com/dotnet/aspnetcore/blob/e9c1d8ac923a722e82112cd968fdc26569a52d95/src/Identity/Extensions.Core/src/PasswordHasher.cs#L220)

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| account_id | アカウントID | requried | String |・[PCAアカウント](/docs/common/PCAアカウント.md) の「アカウント属性項目」を参照 |
| organization_id | 組織ID | requried | String | |

```json
{
  "account_id": "xxxx-xx-xx-xxxx", 
  "organization_id": "xxxx-xx-xx-xxxx"
}
```
