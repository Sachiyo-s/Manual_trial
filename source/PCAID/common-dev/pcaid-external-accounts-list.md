---
sidebar_position: 3
---

# 3. PCA ID 外部システム接続先環境

## 接続先エンドポイント

- [PCA ID ドメイン＆ベースURL定義](./pcaid-domain-and-baseurl.md)
- [別アカウントDBサーバーアクセス](https://github.com/pca-idp/pcaid/discussions/974)

## 接続先環境の組み合わせ

| [PCA ID](./pcaid-domain-and-baseurl.md) | [PCAクラウド](./pcaid-external-accounts-list#pca%E3%82%AF%E3%83%A9%E3%82%A6%E3%83%89) | [PCAサブスク](./pcaid-external-accounts-list#pca%E3%82%B5%E3%83%96%E3%82%B9%E3%82%AF) | [顧客DBサーバー](./pcaid-external-accounts-list#%E9%A1%A7%E5%AE%A2db%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC) |
| --- | --- | --- | --- |
| PoC環境 | [48環境](./pcaid-external-accounts-list#pca%E3%82%AF%E3%83%A9%E3%82%A6%E3%83%89-%E9%96%8B%E7%99%BA%E7%92%B0%E5%A2%8348%E7%92%B0%E5%A2%83)＋手動データ | [feature環境](./pcaid-external-accounts-list#pca%E3%82%B5%E3%83%96%E3%82%B9%E3%82%AF-%E3%83%95%E3%83%AC%E3%83%BC%E3%83%A0%E3%83%AF%E3%83%BC%E3%82%AF%E9%96%8B%E7%99%BA%E7%92%B0%E5%A2%83feature%E7%92%B0%E5%A2%83)＋手動データ | [ステージング環境](./pcaid-external-accounts-list#%E9%A1%A7%E5%AE%A2db%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC-%E3%82%B9%E3%83%86%E3%83%BC%E3%82%B8%E3%83%B3%E3%82%B0%E7%92%B0%E5%A2%83)＋手動データ |
| 開発環境 | [48環境](./pcaid-external-accounts-list#pca%E3%82%AF%E3%83%A9%E3%82%A6%E3%83%89-%E9%96%8B%E7%99%BA%E7%92%B0%E5%A2%8348%E7%92%B0%E5%A2%83) | [feature環境](./pcaid-external-accounts-list#pca%E3%82%B5%E3%83%96%E3%82%B9%E3%82%AF-%E3%83%95%E3%83%AC%E3%83%BC%E3%83%A0%E3%83%AF%E3%83%BC%E3%82%AF%E9%96%8B%E7%99%BA%E7%92%B0%E5%A2%83feature%E7%92%B0%E5%A2%83) | [ステージング環境](./pcaid-external-accounts-list#%E9%A1%A7%E5%AE%A2db%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC-%E3%82%B9%E3%83%86%E3%83%BC%E3%82%B8%E3%83%B3%E3%82%B0%E7%92%B0%E5%A2%83)＋手動データ |
| 検証環境 | [70環境](./pcaid-external-accounts-list#pca%E3%82%AF%E3%83%A9%E3%82%A6%E3%83%89-%E6%A4%9C%E8%A8%BC%E7%92%B0%E5%A2%8370%E7%92%B0%E5%A2%83) | [dev環境](./pcaid-external-accounts-list#pca%E3%82%B5%E3%83%96%E3%82%B9%E3%82%AF-%E8%A3%BD%E5%93%81%E9%96%8B%E7%99%BA%E6%A4%9C%E8%A8%BC%E7%92%B0%E5%A2%83dev%E7%92%B0%E5%A2%83) | [ステージング環境](./pcaid-external-accounts-list#%E9%A1%A7%E5%AE%A2db%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC-%E3%82%B9%E3%83%86%E3%83%BC%E3%82%B8%E3%83%B3%E3%82%B0%E7%92%B0%E5%A2%83) |
| 本番環境 | [00, 10, 20, 40環境](./pcaid-external-accounts-list#pca%E3%82%AF%E3%83%A9%E3%82%A6%E3%83%89-%E6%9C%AC%E7%95%AA%E7%92%B0%E5%A2%8300-10-20-40%E7%92%B0%E5%A2%83) | [本番環境](./pcaid-external-accounts-list#pca%E3%82%B5%E3%83%96%E3%82%B9%E3%82%AF-%E6%9C%AC%E7%95%AA%E7%92%B0%E5%A2%83) | [本番環境](./pcaid-external-accounts-list#%E9%A1%A7%E5%AE%A2db%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC-%E6%9C%AC%E7%95%AA%E7%92%B0%E5%A2%83) |
| サンドボックス環境 | [19, 29環境](./pcaid-external-accounts-list#pca%E3%82%AF%E3%83%A9%E3%82%A6%E3%83%89-%E6%9C%AC%E7%95%AA%E3%83%86%E3%82%B9%E3%83%88%E7%92%B0%E5%A2%8319-29%E7%92%B0%E5%A2%83) | [本番テスト環境](./pcaid-external-accounts-list#pca%E3%82%B5%E3%83%96%E3%82%B9%E3%82%AF-%E6%9C%AC%E7%95%AA%E3%83%86%E3%82%B9%E3%83%88%E7%92%B0%E5%A2%83) | [ステージング環境](./pcaid-external-accounts-list#%E9%A1%A7%E5%AE%A2db%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC-%E3%82%B9%E3%83%86%E3%83%BC%E3%82%B8%E3%83%B3%E3%82%B0%E7%92%B0%E5%A2%83) |



## PCAクラウド

### 管理文書（Notes）

- 接続情報
  - [Ⅹシリーズ会議室 - クラウドAWS開発環境メモ](notes://IBM5580Y.pca.co.jp/492570490006EA39/840CE3A13BC98C8549256A1600245813/272E14EF2210BAFB4925856F00247813)
  - [Ⅹシリーズ会議室 -本番DCサーバー（検証区画）](notes://IBM5580Y.pca.co.jp/492570490006EA39/840CE3A13BC98C8549256A1600245813/2CB5ED0E086B3EE549257F330026F681)
- アカウント情報
  - [SaaS開発会議室 - 開発部用クラウドテストアカウント割り当て](notes://IBM5580Y.pca.co.jp/49257371003A186A/840CE3A13BC98C8549256A1600245813/CFDC907ED560741149257F7F000EA100)
  - [SaaS開発会議室 - 検証/開発用クラウドAWSテスト用アカウント割り当て](notes://IBM5580Y.pca.co.jp/49257371003A186A/840CE3A13BC98C8549256A1600245813/42E31EC3CE63072D4925861D003086C2)

### API情報

- [PCAクラウド認証API](https://github.com/pca-idp/pcaid/discussions/976)

### 環境情報

#### PCAクラウド 開発環境（48環境）

| カテゴリ | 項目 | 値 |
| -- | -- | -- |
| API | スキーマURL | <http://cloud2-dev-alb-auth-2102122029.ap-northeast-1.elb.amazonaws.com:8080/PcaCertService/services/Certify?wsdl> |
| DBサーバー | host | pcaid-{env}-rdsproxy-endpoint-cloud2-dev.endpoint.proxy-ctteex0if3cc.ap-northeast-1.rds.amazonaws.com |
| DBサーバー | port | 5432 |
| DBサーバー | database | pcadb |
| DBサーバー | username | pcaid_dbuser |

```cmd
# PoC環境
psql --host=pcaid-poc-rdsproxy-endpoint-cloud2-dev.endpoint.proxy-ctteex0if3cc.ap-northeast-1.rds.amazonaws.com --port=5432 --dbname=pcadb --username=pcaid_dbuser
# 開発環境
psql --host=pcaid-dev-rdsproxy-endpoint-cloud2-dev.endpoint.proxy-ctteex0if3cc.ap-northeast-1.rds.amazonaws.com --port=5432 --dbname=pcadb --username=pcaid_dbuser
```

#### PCAクラウド 検証環境（70環境）

| カテゴリ | 項目 | 値 |
| -- | -- | -- |
| API | スキーマURL | <https://cert.cbc-demo.info/PcaCertService/services/Certify?wsdl> |
| DBサーバー | host | pcaid-rdsproxy-endpoint.endpoint.proxy-cbs9kdp7eehb.ap-northeast-1.rds.amazonaws.com |
| DBサーバー | port | 5432 |
| DBサーバー | database | pcadb |
| DBサーバー | username | pcaid_dbuser |

```cmd
# 検証環境
psql --host=pcaid-rdsproxy-endpoint.endpoint.proxy-cbs9kdp7eehb.ap-northeast-1.rds.amazonaws.com --port=5432 --dbname=pcadb --username=pcaid_dbuser
```

#### PCAクラウド 本番環境（00, 10, 20, 40環境）

| カテゴリ | 項目 | 値 |
| -- | -- | -- |
| API | スキーマURL（デフォルト） | 20環境と同じ |
| API | スキーマURL（10） | <https://west01.pcacloud.jp/cert02/PcaCertService/services/Certify?wsdl> |
| API | スキーマURL（20） | <https://east02.pcacloud.jp/cert03/PcaCertService/services/Certify?wsdl> |
| API | スキーマURL（40） | <https://cert.pcacloud.jp/PcaCertService/services/Certify?wsdl> |
| DBサーバー | host | 非公開 |
| DBサーバー | port | 非公開 |
| DBサーバー | database | 非公開 |
| DBサーバー | username | 非公開 |

#### PCAクラウド 本番テスト環境（19, 29環境）

| カテゴリ | 項目 | 値 |
| -- | -- | -- |
| API | スキーマURL（デフォルト） | 29環境と同じ |
| API | スキーマURL（19） | <https://west01.pcacloud.jp/saas02011/PcaCertService/services/Certify?wsdl> |
| API | スキーマURL（29） | <https://east02.pcacloud.jp/saas03011/PcaCertService/services/Certify?wsdl> |
| DBサーバー | host | 非公開 |
| DBサーバー | port | 非公開 |
| DBサーバー | database | 非公開 |
| DBサーバー | username | 非公開 |

## PCAサブスク

### 管理文書（Notes）

- 接続情報・アカウント情報
  - [Ⅹシリーズ会議室 - LS01.構成](notes://IBM5580Y.pca.co.jp/492570490006EA39/840CE3A13BC98C8549256A1600245813/332FD3610D81C00749258402000DBDD6)
  - [Ⅹシリーズ会議室 - LM090.開発・検証方法](notes://IBM5580Y.pca.co.jp/492570490006EA39/1B330C2CFE24A30C49256B510034F398/E61D07D05E0F941649258480002C7C83)
  - [Ⅹシリーズ会議室 - LM900.WebOperator（ライセンスサーバーDBデータ変更ツール）](notes://IBM5580Y.pca.co.jp/492570490006EA39/840CE3A13BC98C8549256A1600245813/7C464D39DEB58FDB4925847B0024398E)

### API情報

- [PCAサブスク認証API](https://github.com/pca-idp/pcaid/discussions/975)

### 環境情報

#### PCAサブスク フレームワーク開発環境（feature環境）

| カテゴリ | 項目 | 値 |
| -- | -- | -- |
| API | ベースURL | <https://www.ls-feature.pca-dev.net/fe/api/v1> |
| 画面 | 契約ユーザーサイト | <https://www.ls-feature.pca-dev.net/sites/mgr/#/> |
| 画面 | 社内管理サイト | <https://www.ls-feature.pca-dev.net/sites/inhouse/#/> |
| 開発・検証 | Swagger UI | <https://www.ls-feature.pca-dev.net/fe/swagger> |
| 開発・検証 | DBデータ管理 | <https://www.ls-feature.pca-dev.net/operator> |
| DBサーバー | host | pcaid-{env}-rdsproxy-endpoint-ls-feature.endpoint.proxy-crmqgu4wysxn.ap-northeast-1.rds.amazonaws.com |
| DBサーバー | port | 5432 |
| DBサーバー | database | lmdb_feature |
| DBサーバー | username | pcaid_dbuser |

```cmd
# PoC環境
psql --host=pcaid-poc-rdsproxy-endpoint-ls-feature.endpoint.proxy-crmqgu4wysxn.ap-northeast-1.rds.amazonaws.com --port=5432 --dbname=lmdb_feature --username=pcaid_dbuser

# 開発環境
psql --host=pcaid-dev-rdsproxy-endpoint-ls-feature.endpoint.proxy-crmqgu4wysxn.ap-northeast-1.rds.amazonaws.com --port=5432 --dbname=lmdb_feature --username=pcaid_dbuser
```

#### PCAサブスク 製品開発・検証環境（dev環境）

| カテゴリ | 項目 | 値 |
| -- | -- | -- |
| API | ベースURL | <https://www.ls-dev.pca-dev.net/dev/api/v1> |
| 画面 | 契約ユーザーサイト | <https://www.ls-dev.pca-dev.net/sites/mgr/#/> |
| 画面 | 社内管理サイト | <https://www.ls-dev.pca-dev.net/sites/inhouse/#/> |
| 開発・検証 | Swagger UI | <https://www.ls-dev.pca-dev.net/dev/swagger> |
| 開発・検証 | DBデータ管理 | <https://www.ls-dev.pca-dev.net/operator> |
| DBサーバー | host | pcaid-{env}-rdsproxy-endpoint-ls-dev.endpoint.proxy-crmqgu4wysxn.ap-northeast-1.rds.amazonaws.com |
| DBサーバー | port | 5432 |
| DBサーバー | database | lmdb |
| DBサーバー | username | pcaid_dbuser |

```cmd
# 検証環境
psql --host=pcaid-stg-rdsproxy-endpoint-ls-dev.endpoint.proxy-crmqgu4wysxn.ap-northeast-1.rds.amazonaws.com --port=5432 --dbname=lmdb --username=pcaid_dbuser
```

#### PCAサブスク 本番環境

| カテゴリ | 項目 | 値 |
| -- | -- | -- |
| API | ベースURL | <https://www.pcalic.jp/license/api/v1> |
| 画面 | 契約ユーザーサイト | <https://www.pcalic.jp/sites/mgr/#/> |
| 画面 | 社内管理サイト | <https://www.pcalic.jp/sites/inhouse/#/> |
| 開発・検証 | Swagger UI | <https://www.pcalic.jp/license/swagger> |
| 開発・検証 | DBデータ管理 | なし |
| DBサーバー | host | 非公開 |
| DBサーバー | port | 非公開 |
| DBサーバー | database | 非公開 |
| DBサーバー | username | 非公開 |

#### PCAサブスク 本番テスト環境

| カテゴリ | 項目 | 値 |
| -- | -- | -- |
| API | ベースURL | <https://www.pcalic.jp/license_test/api/v1> |
| 画面 | 契約ユーザーサイト | <https://www.pcalic.jp/sites_test/mgr/#/> |
| 画面 | 社内管理サイト | <https://www.pcalic.jp/sites_test/inhouse/#/> |
| 開発・検証 | Swagger UI | <https://www.pcalic.jp/license_test/swagger> |
| 開発・検証 | DBデータ管理 | なし |
| DBサーバー | host | 非公開 |
| DBサーバー | port | 非公開 |
| DBサーバー | database | 非公開 |
| DBサーバー | username | 非公開 |

## 顧客DBサーバー

### 環境情報

#### 顧客DBサーバー ステージング環境

| 項目 | 値 |
| -- | -- |
| host | pca-stg-database01.cvy4uewmiv2z.ap-northeast-1.rds.amazonaws.com |
| port | 5432 |
| database | cti05stg |
| username | pcaid_readonly |

```cmd
psql --host=pca-stg-database01.cvy4uewmiv2z.ap-northeast-1.rds.amazonaws.com --port=5432 --dbname=cti05stg --username=pcaid_readonly
```

#### 顧客DBサーバー 本番環境

| 項目 | 値 |
| -- | -- |
| host | pca-prd-database01.cvy4uewmiv2z.ap-northeast-1.rds.amazonaws.com |
| port | 5432 |
| database | cti05 |
| username | pcaid_readonly |

```cmd
psql --host=pca-prd-database01.cvy4uewmiv2z.ap-northeast-1.rds.amazonaws.com --port=5432 --dbname=cti05 --username=pcaid_readonly
```
