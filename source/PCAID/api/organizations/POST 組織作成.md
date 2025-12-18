---
sidebar_position: 1
---

# POST 組織作成

```http
/organizations
```

## 説明

[組織](/docs/common/組織.md)を作成します。

- 同名の既存組織がある状況でも、別サービスから利用することを想定する
- PCA Hub において、テナントを作成するとき、暗黙的に新規組織を作成する
- PCAクラウド等のサービスでは、フロントで新規組織か既存組織を選択させて、このAPIを利用することを想定する
  - 組織作成と同時に[サービス区画](/docs/common/サービス区画.md)も追加する
- 組織ごとに、既定ロールと、サービス区画内で利用する任意ロールをそれぞれ作成する
  - [ロール（役割）](/docs/common/ロール（役割）.md)
  - 既定ロールは、`pca.id.{organizaiton_id}` から始まるすべてのロールを対象とする
  - サービス区画内で利用する任意ロールは、例えば、PCA Hub テナント「tdi」であれば  `pca.hub.tdi/gs:admin` となる
- この組織に対して、上記の既定ロールのうち、`pca.id.{organizaiton_id}/user` を付与する
  - 所属ユーザーに対して、自動的に `pca.id.{organizaiton_id}/user` ロールが付与される（アクセストークンに含まれる）ようにする
- 対応する[組織予約](./organization_reservations/organization_name/POST%20組織予約.md)が存在しなければエラーとし、組織を作成したらその予約を削除する
- 組織名の再利用は不可としエラーとする
  - 過去に利用した組織名は履歴管理する

## INPUT

### HEADER

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| Authorization | [OAuthアクセストークン](/docs/common-dev/oidc-oauth-token-rules.md) | required | String | [管理APIの認可方法](/docs/common-dev/pcaid-management-api-granttype.md) |

```http
Authorization: Bearer xxxx-xxxx-xxxx-xxxx
```

### BODY

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| organization_name | [組織名](/docs/common/組織.md) | requried | String ||
| organization_display_name | 組織表示名 | conditional | String | ・新規組織では必須として未指定ならエラーとする <br/>・既存組織があるときは必須とせず値を無視する |
| service_partition | [サービス区画](/docs/common/サービス区画.md) | conditional | String | ・指定された値を追加し、追加済みなら何もしない<br/>・PCA Hubサービスからの作成時は必須 <br/>・原則として、利用を可能とする呼び出し元のサービスの区画（例：Hubならテナント名）を指定する |
| service_roles | サービスロール一覧 | optional | String | ・指定されたサービス区画で利用可能なロールとして作成する（service_partitionと結合する） <br/>・例：`pca.hub.tdi/gs:admin` <br/>・追加済みなら何もしない |

```json
{
  "organization_name": "tdi",
  "organization_display_name": "TOKYO DIGITAL IDEAS",
  "service_partition": "pca.hub.tdi",
  "service_roles": [
    "gs:admin", 
    "d:users"
  ]
}
```

## OUTPUT

| Field_Name | Field_Name_Description | Required | Type | Comment |
| -- | -- | -- | -- | -- |
| organization_id | 組織ID | requried | String | ・[組織](/docs/common/組織.md)の「組織ID」を参照 |

```json
{
  "organization_id": "xxxx-xx-xx-xxxx"
}
```
