---
sidebar_position: 27
---

# 27. サービス契約データの扱い

## サービス契約データの保存場所

### 元データ

- サービス契約ごとのDBサーバーにて管理している
  - 顧客DB（顧客IDと契約ID）
    - PCA Arch の契約情報も含まれる
  - PCAクラウド（契約IDと製品ライセンス）
  - PCAサブスク（契約IDと製品ライセンス）
- それぞれのDBサーバーにアクセスしてデータを取得する
  - [別アカウントDBサーバーアクセス](./pcaid-external-dbservers-access.md)
  - **【重要】サーバーアクセスが常に成功することを前提にしない**
    - アクセス不能のときはCloudWatchログに目印（キーワード）が記録される
      - `[db.error]`
      - → モニタリング環境にて検出して、速やかにSlack通知されるようにする
  - → サービス契約の取得データは自身のアカウント内でキャッシュする
    - [サービス契約データのキャッシュ方針](#サービス契約データのキャッシュ方針)

### キャッシュデータ

- [DynamoDB](./dynamodb-data-spec.md) を利用してキャッシュを保持する
  - `pcaid-portal-contract_info-{env}` テーブル
- サービス契約サーバーへのアクセス不能を想定して、キャッシュ保持期間は **2日間（48時間）** とする
- 最新の元データに更新するまでの、キャッシュ更新間隔は **1時間** とする
  - [アプリAPI](../api/apps-v1.md)の許可チェックにおいて、キャッシュデータを参照する状況を想定する
  - 最新データを取得できずキャッシュが古くなっても、キャッシュを保持している限りは認可チェックに利用する
    - [アプリ向けAPIオーソライザー](./api-authorizers.md#4-アプリ向け-api-オーソライザーpcaid-cloudauthorither)
- 管理コンソールや組織作成においては、定期更新を待たずに **強制的に更新** することがある
  - [サービス契約データのキャッシュ方針](#サービス契約データのキャッシュ方針)
  - 利用サービスの追加・更新
    - 「追加」の他に、「更新」ボタンを配置して強制更新を可能にする
    - 画面上に「最終更新日時」を表示する
- キャッシュ保持期間とキャッシュ更新間隔は、環境変数として設定する
  - PoC環境においては以下の動作とする
    - キャッシュデータの保持期間は無期限とし、自動更新は不可とする（更新間隔 0 とする）
    - サービス契約サーバーへはアクセスしない
      - 管理コンソールからの取得要求があっても強制更新せず「最終更新日時」は変わらない

### テスト用データ

- テスト用データも本番データ同様にキャッシュする
- 本番環境以外では、サービス契約の元データには、PCAクラウド／PCAサブスクのテスト用データを使用する
  - [PCA ID 外部システム接続先環境](./pcaid-external-accounts-list.md)
  - テスト用のサービス契約データは、開発・検証の都合で、自由に作成・変更・削除できる
    - ただし、CBCが管理する環境については、CBCに依頼する必要がある
- 顧客DBのテスト用データは、[DynamoDB](./dynamodb-data-spec.md) で管理する独自データを使用する
  - `pcaid-portal-service_info_customer_test-{env}` テーブル
    - 契約IDに対して、ダミーの「顧客ID」と「Arch登録ID」を設定できる
  - テスト用データの作成・更新・削除には、AWSが提供する管理ツールを利用する
    - [DynamoDBデータ管理ツール（NoSQL Workbench）](https://github.com/pca-idp/pcaid/discussions/1107)

### DynamoDBテーブル（サービス契約のキャッシュデータ）

`pcaid-portal-contract_info-{env}` テーブルレコードの例（「PCAクラウド」の場合）

#### サービス契約情報

| パラメータ | 名前 | データ型 | 値の例 | 備考 |
| --- | --- | --- | --- | --- |
| customer_id | 顧客ID | string | 10009999 | ・契約済み <br/> ・Arch体験利用（仮顧客） |
| service_partition | サービス区画 | string | pca.cloud.org-abcd-1234 | |
| service_contract_type | サービス種類 | string | pca.cloud | `pca.hub`：PCA Hub <br/> `pca.cloud`：PCAクラウド <br/> `pca.subsc`：PCAサブスク |
| service_contract_id | サービス契約ID | string | 40001234 | |
| service_contract_company | サービス契約会社 | string | イイダバシ株式会社 | |
| service_contract_status | サービス契約状態 | string | valid | `valid`：有効 <br/> `invalid`：無効 |
| arch_registration_id | Arch登録ID | string | 123456 | ・Archプランで有効 <br/> ・未登録なら `null` |
| cache_expiry | キャッシュ期限 | integer | 1753612542 | |
| cache_updated_at | キャッシュ更新日時 | string | 2024-11-01T10:20:00.123ZS | |

#### 製品ライセンス情報（app_licenses）

| パラメータ | 名前 | データ型 | 値の例 | 備考 |
| --- | --- | --- | --- | --- |
| app_id | アプリID | string | Acc20 <br/> SPay20 | 製品を特定するID |
| app_edition_id | アプリエディションID | string | Acc20 <br/> SPay20SQL | |
| app_name | アプリ名 | string | PCAクラウド 会計 dx <br/> PCAサブスク 給与 hyper | |
| app_usage_status | アプリ利用状態 | string | valid | `valid`：有効 <br/> `invalid`：無効 |
| app_usage_cal | アプリ利用CAL | integer | 3 <br/> null | 未設定なら `null` |
| app_usage_period_start | アプリ利用開始日 | string | 2024-04-01 | 未設定なら `0001-01-01` |
| app_usage_period_end  | アプリ利用終了日 | string | 2024-03-31 <br/> 0001-01-01 | 未設定なら `0001-01-01` |

#### 製品サポート情報（app_supports）

本番環境の体験利用では、契約前により顧客DBに製品サポート情報が未登録なので、app_supports は空配列になる

| パラメータ | 名前 | データ型 | 値の例 | 備考 |
| --- | --- | --- | --- | --- |
| app_id | アプリID | string | Acc20 <br/> SPay20 | app_licenses 配列の app_id と紐づく |
| app_registerno | 登録番号 | integer | 21607285 | |
| app_serialno | 製造番号 | string | 1780 | |
| app_category1 | 大分類 | integer | 133 | 202：PCA Hub <br/> 133：PCAクラウド <br/> 199：PCAサブスク |
| app_category2 | 小分類 | integer | 1437 | |
| app_plan_name | 製品プラン | string | PCAクラウド 会計 dx 月額プラン <br/> PCAサブスク 給与 hyper 年額プラン | |
| app_status_name | 契約種別 | string | 月額利用中 <br/> 年額利用中 | |
| app_support_period_start | 契約開始日 | string | 2025-04-01 | 未設定なら `0001-01-01` |
| app_support_period_end | 契約終了日 | string | 2026-03-31 | 未設定なら `0001-01-01` |

```json
{
  "customer_id": "c12345678", 
  "service_partition": "pca.cloud.org-abcd-1234",
  "service_contract_type": "pca.cloud",
  "service_contract_id": "12345",
  "service_contract_company": "イイダバシ株式会社",
  "service_contract_status": "valid",
  "arch_registration_id": "123456",
  "cache_expiry": 1753612542,
  "cache_updated_at": "2024-11-01T10:20:00.123ZS",
  "app_licenses": [
    {
      "app_id": "Acc20",
      "app_edition_id": "Acc20",
      "app_name": "PCAクラウド 会計 dx",
      "app_usage_status": "valid", 
      "app_usage_cal": 10, 
      "app_usage_period_start": "2024-04-01",
      "app_usage_period_end": "2025-03-31",
    },
    {
      "app_id": "SPay20",
      "app_edition_id": "SPay20",
      "app_name": "PCAクラウド 給与 hyper",
      "app_usage_status": "valid", 
      "app_usage_cal": 3, 
      "app_usage_period_start": "2020-04-01",
      "app_usage_period_end": "0001-01-01",
    },
  ],
  "app_supports": [
    {
      "app_id": "Acc20",
      "app_registerno": 22000111,
      "app_serialno": "12345678",
      "app_category1": 133,
      "app_category2": 1437,
      "app_plan_name": "PCAｸﾗｳﾄﾞ 会計dx 買取ﾌﾟﾗﾝ",
      "app_status_name": "年額利用中",
      "app_support_period_start": "2015-04-01",
      "app_support_period_end": "2026-03-31",
    },
    {
      "app_id": "Acc20",
      "app_registerno": 22000222,
      "app_serialno": "23456789",
      "app_category1": 133,
      "app_category2": 1439,
      "app_plan_name": "PCAｸﾗｳﾄﾞ ｿﾌﾄ利用ﾗｲｾﾝｽ会計dx",
      "app_status_name": "月額利用中",
      "app_support_period_start": "2015-04-01",
      "app_support_period_end": "2026-03-31",
    },
    {
      "app_id": "SPay20",
      "app_registerno": 22000333,
      "app_serialno": "34567890",
      "app_category1": 133,
      "app_category2": 1452,
      "app_plan_name": "PCAｸﾗｳﾄﾞ 給与dx 買取ﾌﾟﾗﾝ",
      "app_status_name": "年額利用中",
      "app_support_period_start": "2015-04-01",
      "app_support_period_end": "2026-03-31",
    },
  ],
}
```

## サービス製品マスター

- 別アカウントからのデータからは `app_id` を取得できないので、関連データから `app_id` を特定する
- 製品名については、独自にマスターデータを管理して、関連データから `app_name` を特定する
  - 製品名はサービスごとに表現に「ゆらぎ」があり、統一した製品名が存在していない

### 製品の特定方法（製品マスターの利用方法）

#### app_licenses

- [PCAクラウドSQL](./pcaid-external-dbservers-access.md#pcaクラウドsql) または [PCAサブスクSQL](./pcaid-external-dbservers-access.md#pcaサブスクsql) を使って組み立てる
- 結果セットに含まれる `app_edition_id` から、製品マスターを参照して `app_id` と `app_name` を特定する

#### app_supports

- [顧客契約詳細SQL](./pcaid-external-dbservers-access.md#顧客契約詳細sql) を使って組み立てる
  - 体験利用の未契約ユーザーの場合は、データを取得できない（app_licenses とは整合しない）ことに注意する
- 結果セットに含まれる `app_category2` から、製品マスターを参照して `app_id` を特定する

[PCAサービス製品マスター.xlsx](./materials/PCAサービス製品マスター.xlsx)

| app_grade | app_id | app_edition_id  | app_name                                  | app_category2 |
| :-------- | :----- | :-------------- | :---------------------------------------- | :------------ |
| dx        | Acc20  | Acc20           | PCAクラウド 会計                          | 1437          |
|           | Acc20  | Acc20           | PCAクラウド 会計                          | 1438          |
|           | Acc20  | Acc20           | PCAクラウド 会計                          | 1439          |
|           | Acc20  | Acc20Jiman      | PCAサブスク 会計 jiman                    | 1713          |
|           | Acc20  | Acc20SystemB    | PCAサブスク 会計 dx                       | 1714          |
|           | Acc20  | Acc20SQL        | PCAサブスク 会計 dx nw                    | 1748          |
|           | Jk20   | Jk20            | PCAクラウド 人事管理                      | 1467          |
|           | Jk20   | Jk20            | PCAクラウド 人事管理                      | 1468          |
|           | Jk20   | Jk20            | PCAクラウド 人事管理                      | 1469          |
|           | Jk20   | Jk20Jiman       | PCAサブスク 人事管理 jiman                | 1740          |
|           | Jk20   | Jk20SystemB     | PCAサブスク 人事管理 dx                   | 1741          |
|           | Jk20   | Jk20SQL         | PCAサブスク 人事管理 dx nw                | 1750          |
|           | Kon20  | Kon20Kon        | PCAクラウド 商魂                          | 1455          |
|           | Kon20  | Kon20Kon        | PCAクラウド 商魂                          | 1456          |
|           | Kon20  | Kon20Kon        | PCAクラウド 商魂                          | 1457          |
|           | Kon20  | Kon20Kon        | PCAクラウド 商魂                          | 1574          |
|           | Kon20  | Kon20Kon        | PCAクラウド 商魂                          | 1575          |
|           | Kon20  | Kon20KonJiman   | PCAサブスク 商魂 jiman                    | 1751          |
|           | Kon20  | Kon20KonSystemB | PCAサブスク 商魂 dx                       | 1737          |
|           | Kon20  | Kon20KonSQL     | PCAサブスク 商魂 dx nw                    | 1738          |
|           | Kon20  | Kon20Kan        | PCAクラウド 商管                          | 1458          |
|           | Kon20  | Kon20Kan        | PCAクラウド 商管                          | 1459          |
|           | Kon20  | Kon20Kan        | PCAクラウド 商管                          | 1460          |
|           | Kon20  | Kon20KanJiman   | PCAサブスク 商管 jiman                    | 1732          |
|           | Kon20  | Kon20KanSystemB | PCAサブスク 商管 dx                       | 1733          |
|           | Kon20  | Kon20KanSQL     | PCAサブスク 商管 dx nw                    | 1749          |
|           | Kon20  | Kon20Lot        | PCAクラウド 商管 ロット管理オプション       | 1461          |
|           | Kon20  | Kon20Lot        | PCAクラウド 商管 ロット管理オプション       | 1462          |
|           | Kon20  | Kon20Lot        | PCAクラウド 商管 ロット管理オプション       | 1463          |
|           | Kon20  | Kon21Lot        | PCAサブスク 商管 ロット管理オプション       | 1734          |
|           | Kon20  | Kon20Conveni    | PCAクラウド 商魂 コンビニ収納代行オプション         | 1814          |
|           | Kon20  | Kon20Conveni    | PCAサブスク 商魂 コンビニ収納代行オプション         | 1810          |
|           | Kon20  | Kon20SykNyk     | PCAクラウド 商魂・商管 売上・仕入同時入力オプション  | 1815          |
|           | Kon20  | Kon20SykNyk     | PCAサブスク 商魂・商管 売上・仕入同時入力オプション  | 1736          |
|           | Kon20  | Kon20JucHac     | PCAクラウド 商魂・商管 受注・発注同時入力オプション  | 1816          |
|           | Kon20  | Kon20JucHac     | PCAサブスク 商魂・商管 受注・発注同時入力オプション  | 1735          |
|           | Pay20  | Pay20           | PCAクラウド 給与                          | 1452          |
|           | Pay20  | Pay20           | PCAクラウド 給与                          | 1453          |
|           | Pay20  | Pay20           | PCAクラウド 給与                          | 1454          |
|           | Pay20  | Pay20Jiman      | PCAサブスク 給与 jiman                    | 1719          |
|           | Pay20  | Pay20SystemB    | PCAサブスク 給与 dx                       | 1720          |
|           | Pay20  | Pay20SQL        | PCAサブスク 給与 dx nw                    | 1747          |
|           | Dep20  | Dep20           | PCAクラウド 固定資産                      | 1464          |
|           | Dep20  | Dep20           | PCAクラウド 固定資産                      | 1465          |
|           | Dep20  | Dep20           | PCAクラウド 固定資産                      | 1466          |
|           | Dep20  | Dep20SystemB    | PCAサブスク 固定資産 dx                   | 1723          |
|           | Dep20  | Dep20SQL        | PCAサブスク 固定資産 dx nw                | 1724          |
|           | Psc20  | Psc20           | PCAクラウド 公益法人会計                  | 1545          |
|           | Psc20  | Psc20           | PCAクラウド 公益法人会計                  | 1546          |
|           | Psc20  | Psc20           | PCAクラウド 公益法人会計                  | 1547          |
|           | Psc20  | Psc20SystemB    | PCAサブスク 公益法人会計 dx               | 1727          |
|           | Psc20  | Psc20SQL        | PCAサブスク 公益法人会計 dx nw            | 1728          |
|           | Ssc20  | Ssc20           | PCAクラウド 社会福祉法人会計              | 1548          |
|           | Ssc20  | Ssc20           | PCAクラウド 社会福祉法人会計              | 1549          |
|           | Ssc20  | Ssc20           | PCAクラウド 社会福祉法人会計              | 1550          |
|           | Ssc20  | Ssc20SystemB    | PCAサブスク 社会福祉法人会計 dx           | 1729          |
|           | Ssc20  | Ssc20SQL        | PCAサブスク 社会福祉法人会計 dx nw        | 1730          |
|           | Ken20  | Ken20           | PCAクラウド 建設業会計                    | 1754          |
|           | Ken20  | Ken20           | PCAクラウド 建設業会計                    | 1755          |
|           | Ken20  | Ken20SystemB    | PCAサブスク 建設業会計 dx                 | 1758          |
|           | Ken20  | Ken20SQL        | PCAサブスク 建設業会計 dx nw              | 1785          |
|           | Kob20  | Kob20           | PCAクラウド 個別原価会計                  | 1756          |
|           | Kob20  | Kob20           | PCAクラウド 個別原価会計                  | 1757          |
|           | Kob20  | Kob20SystemB    | PCAサブスク 個別原価会計 dx               | 1759          |
|           | Kob20  | Kob20SQL        | PCAサブスク 個別原価会計 dx nw            | 1786          |
|           | Leg20  | Leg20           | PCAクラウド 法定調書                      | 1653          |
|           | Leg20  | Leg20           | PCAクラウド 法定調書                      | 1654          |
|           | Leg20  | Leg20SystemB    | PCAサブスク 法定調書 dx                   | 1745          |
|           | Leg20  | Leg20SQL        | PCAサブスク 法定調書 dx nw                | 1746          |
| hyper     | SAcc20 | SAcc20          | PCAクラウド 会計 hyper                    | 1668          |
|           | SAcc20 | SAcc20          | PCAクラウド 会計 hyper                    | 1669          |
|           | SAcc20 | SAcc20SystemB   | PCAサブスク 会計 hyper                    | 1715          |
|           | SAcc20 | SAcc20SQL       | PCAサブスク 会計 hyper nw                 | 1716          |
|           | SDep20 | SDep20          | PCAクラウド 固定資産 hyper                | 1670          |
|           | SDep20 | SDep20          | PCAクラウド 固定資産 hyper                | 1671          |
|           | SDep20 | SDep20SystemB   | PCAサブスク 固定資産 hyper                | 1725          |
|           | SDep20 | SDep20SQL       | PCAサブスク 固定資産 hyper nw             | 1726          |
|           | SPay20 | SPay20          | PCAクラウド 給与 hyper                    | 1698          |
|           | SPay20 | SPay20          | PCAクラウド 給与 hyper                    | 1699          |
|           | SPay20 | SPay20SystemB   | PCAサブスク 給与 hyper                    | 1721          |
|           | SPay20 | SPay20SQL       | PCAサブスク 給与 hyper nw                 | 1722          |
|           | SJk20  | SJk20           | PCAクラウド 人事管理 hyper                | 1696          |
|           | SJk20  | SJk20           | PCAクラウド 人事管理 hyper                | 1700          |
|           | SJk20  | SJk20SystemB    | PCAサブスク 人事管理 hyper                | 1742          |
|           | SJk20  | SJk20SQL        | PCAサブスク 人事管理 hyper nw             | 1743          |
|           | Cad20  | Cad20Cre        | PCAクラウド 会計 hyper 債権管理オプション | 1695          |
|           | Cad20  | Cad20Cre        | PCAクラウド 会計 hyper 債権管理オプション | 1701          |
|           | Cad20  | Cad20CreSQL     | PCAサブスク 会計 hyper 債権管理オプション | 1717          |
|           | Cad20  | Cad20Deb        | PCAクラウド 会計 hyper 債務管理オプション | 1697          |
|           | Cad20  | Cad20Deb        | PCAクラウド 会計 hyper 債務管理オプション | 1702          |
|           | Cad20  | Cad20DebSQL     | PCAサブスク 会計 hyper 債務管理オプション | 1718          |

## サービス契約データのキャッシュ方針

### キャッシュ実装の種類

#### (A) キャッシュ作成

- キャッシュが有効なら何もしない
  - キャッシュが有効とは、期限内のキャッシュが存在する状態
  - キャッシュを有効と見なす期限は「**1時間**」とする
- キャッシュが無効なら、元データから契約情報の読み込んでキャッシュを作成する
  - キャッシュが無効とは、存在しない（未作成、自動削除済み）か、期限切れの状態
  - 無効状態のキャッシュが、DynamoDBによって自動削除されるまでの期間は「**48時間**」とする
  - キャッシュが無効でも、元データへのアクセスが不可となっている状況では、古いキャッシュを参照することを想定する
    - 最大で48時間までは、サービス契約に関連する PCA ID 機能を継続させる

#### (B) 強制キャッシュ作成

- キャッシュの状態に関係なく、元データから契約情報の読み込んでキャッシュを作成する
- 既存のキャッシュが存在したら上書きする

#### (C) キャッシュ参照

- キャッシュから契約情報の読み込む

### キャッシュ作成の詳細 (A) (B)

1. サービス種類と契約IDを受け取る
   - PoC環境では何もせずに終了する
2. 強制作成 (B) ならキャッシュ検索（手順3）をスキップして元データへアクセスする（手順4へ）
3. `contract_id` でキャッシュを検索して、有効データがヒットすれば終了する
   - `contract_id` は「 **{サービス種類}#{契約ID}** 」形式とする
     - 例：`pca.cloud#12345678`
   - service_partitoin と区別するため、異なる形式としている
     - 例：`pca.cloud.org-abcd-1234`
4. 元データから契約情報を組み立てる
   - サービス種類に応じたクエリーを使って、製品ライセンス情報を取得する
   - 顧客DBクエリーを使って、顧客IDを取得する
5. 契約情報を組み立てることができたら、キャッシュとして書き込む

### キャッシュ実装のユースケース

#### ケース１. 組織作成・利用サービスの追加

1. ユーザー入力としてサービス種類と契約IDを受け取る
2. **契約情報のキャッシュを強制的に作成する (B)**
3. 契約情報を参照する (C)
4. サービス種類に応じた連携コードを生成して、サービス区画を組み立てて作成する
5. サービス区画の属性値を設定する
   - 契約ID
   - 顧客ID

#### ケース２. API認可チェック

1. APIリクエストとしてサービス区画を受け取る
2. 組織情報やサービス区画を参照して必要なチェックをする
3. 契約情報のキャッシュを作成する (A)
   - キャッシュが古くても許容する（有効期限は1時間）
   - 契約状態（利用サービスの有効状態）の反映にタイムラグが発生する
4. 契約情報を参照する (C)
5. 契約の状態をチェックして、valid でなければエラーとする

#### ケース３. 組織管理の表示

1. 組織に紐づくサービス区画に対して、契約情報のキャッシュを作成する (A)
   - キャッシュが古くても許容する（有効期限は1時間）
   - 顧客IDや製品ライセンス情報、契約状態（利用サービスの有効状態）の表示にタイムラグが発生する
2. 契約情報を参照する (C)
3. 契約IDや顧客IDは、Keycloakに保存してあるサービス区画のデータを表示する
   - 顧客IDに変更があればKeycloakに反映する
   - サービス区画ごとの顧客IDと主たる顧客IDの両方を対象とする
4. 契約の状態は、参照した契約情報（C）を表示する
   - 契約状態「無効」なら分かるように表示する

#### ケース４. 組織管理の更新

1. 組織に紐づくサービス区画に対して、**契約情報のキャッシュを強制的に作成する (B)**
2. 契約情報を参照する (C)
3. 顧客IDは、Keycloakに保存してあるサービス区画のデータを更新する
4. 契約IDや顧客IDは、Keycloakに保存してあるサービス区画のデータを表示する
5. 契約の状態は、参照した契約情報（C）を表示する
   - 契約状態（利用サービスの有効状態）が「無効」なら分かるように表示する
