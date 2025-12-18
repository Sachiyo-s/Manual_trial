---
sidebar_position: 1
---

# POST ユーザーインポート状態取得

```http
/users/import/tasks/{task_id}
```

## 説明

[ユーザーインポート](./../../POST%20ユーザーインポート非同期.md) の非同期実行に対して、タスクの実行状態を取得します。

- URLパスに指定される `task_id` でタスクを特定する
- タスク終了を返す際に、インポート結果CSVとしてユーザーごとの実行結果を取得できるようにする
  - インポート結果は、S3オブジェクトとして保存する
  - インポート結果は、署名付きURLによりダウンロード可能とする
  - インポート結果は、ライフサイクルルールを設定して自動削除する

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | [OAuthアクセストークン](/docs/common-dev/oidc-oauth-token-rules.md) | required | String | [管理APIの認可方法](/docs/common-dev/pcaid-management-api-granttype.md) |
| X-PCA-organization-id | 組織ID | conditional | String | ・[サービス区画](/docs/common/サービス区画.md)（X-PCA-service-partition）が未指定の場合は必須 <br/> ・任意の組織を指定して利用されることを想定する <br/> ・両方指定された場合はこの値を優先する |
| X-PCA-service-partition | [サービス区画](/docs/common/サービス区画.md) | conditional | String |  ・組織ID（X-PCA-organization-id）が未指定の場合は必須 <br/> ・原則として、呼び出し元のサービス区画を指定する |

```http
Authorization: Bearer xxxx-xxxx-xxxx-xxxx
```

```http
X-PCA-organization-id: xxxx-xx-xx-xxxx
```

```http
X-PCA-service-partition: pca.hub.tenant1
```

### BODY

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| なし | | | | |

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| task_id | タスクID | required | String | [ユーザーインポート](./../../POST%20ユーザーインポート非同期.md) が都度発行するID |
| csv_file_name | 参照CSVファイル名 | required | String | |
| task_status | タスク状態 | required | String | ・`importing` `finished` `stopped` のいずれか<br/>・状態ごとの説明は下記の表を参照 |
| created_at | タスク作成日時 | required | String | ISO8601 UTC 形式 |
| created_by | タスク作成アカウントID | required | String | ・ユーザーインポートの指示ユーザー<br/>・[PCAアカウント](/docs/common/PCAアカウント.md) の「アカウントID」を参照 |
| task_start_at | タスク開始日時 | required | String | ISO8601 UTC 形式 |
| task_end_at | タスク終了日時 | required | String | ISO8601 UTC 形式 |
| task_run_by | タスク実行クライアントID | required | String | 非同期にバックグラウンド実行する「インポート処理」に割り当てた[OAuthクライアントID](/docs/common-dev/oidc-oauth-clients.md) |
| imported_user_count | インポート完了ユーザー数 | required | Number | |
| failed_user_count | インポート失敗ユーザー数 | required | Number | |
| task_result_url | インポート結果URL | conditional | String | ・インポート結果を保存したCSVオブジェクトへの署名付きURLを発行する<br/>・タスクが終了してなければ null とする |

```json
{
  "task_id": "0b671c50-5dd8-11ee-b7de-4b16f1ee9b45",
  "csv_file_name": "sample.csv",
  "task_status": "finished",
  "created_at": "2024-04-10T15:00:00Z",
  "created_by": "4fdcc10f-3e65-48f4-a201-ea9e453cb279",
  "task_start_at": "2024-04-10T15:00:10Z",
  "task_end_at": "2024-04-10T15:03:30Z",
  "task_run_by": "b8445ab4-2867-4d25-b9bd-ew38eqh3q9we",
  "imported_user_count": 12,
  "failed_user_count": 0,
  "task_result_url": "https://storage/user/import/..."
}
```

### タスク状態の種類

| 種類 | 説明 |
| -- | -- |
| importing | インポートを実行中の状態で、この時点での処理済みユーザー数は正しく計算されている |
| finished | インポートが正常に終了した状態で、タスク全体の処理済みユーザー数は正しく計算されている |
| stopped | インポート中になんらかの原因でが異常に終了した状態で、処理済みユーザー数は正しく計算されていない可能性がある |

### インポート結果CSV（ユーザーごとの実行結果）

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| user_imported_at | インポート日時 | required | String | `yyyy/mm/dd hh:mm:ss` <br/> ※出力は日本時間（JST）<br/>※保存は世界時間（UTC） |
| user_import_status | インポート状態 | required | String | 成功：`success` <br/> 失敗： `failed` |
| user_import_error | インポートエラー | conditional | String | ・失敗時のエラー情報 <br/>・成功の場合は空 |
| account_id | アカウントID | not required | String | ・常に空（予備項目） |
| login_name | ログイン名 | required | String | インポート時の設定値 |
| email | メールアドレス | required | String | インポート時の設定値 |
| preferred_username | 表示名 | required | String | インポート時の設定値 |
| family_name | 姓 | required | String | インポート時の設定値 |
| given_name | 名 | optional | String | インポート時の設定値 |
| family_kana | 姓カナ | required | String | インポート時の設定値 |
| given_kana | 名カナ | optional | String | インポート時の設定値 |

```csv
Ver1.0
インポート日時,インポート状態,インポートエラー,アカウントID,ログイン名,メールアドレス,表示名,姓,名,姓カナ,名カナ
2024/04/10 15:00:00,sucess,,,yamada,yamada@email.com,総務部_山田太郎,山田,太郎,ヤマダ,タロウ
2024/04/10 15:00:10,sucess,,,sato,satou@email.com,総務部_佐藤花子,佐藤,花子,サトウ,ハナコ
2024/04/10 15:00:20,failed,ログイン名が重複しています。,,sato,satoh@email.com,営業部_佐藤二郎,佐藤,二郎,サトウ,ジロウ
```

- インポート時のユーザーCSVフォーマットに対して、ユーザーごとのインポート結果を付加した拡張フォーマットとする
  - [ユーザーCSVフォーマット](/docs/common/ユーザーCSVフォーマット.md)
  - インポートに失敗したユーザーだけを抜き出すことで、インポートを再実行できるようにする
    - 失敗行を抽出（または成功行を削除）する
    - エラーの原因に対処する
    - 先頭3列のインポート結果を取り除く
- ファイル名は `ユーザーインポート結果_{yy}-{mm}-{dd}_{hh}-{mm}-{ss}.csv` とする
- 署名付きURLの期限は60分とする
- インポート結果CSVオブジェクトの有効期限は1日とする（1日で自動削除される）
