---
sidebar_position: 26
---

# 26. 組織で利用可能なサービス

## サービス一覧

- サービスは、[アプリAPI（/api/apps/v1）](../api/apps-v1.md)を利用できる
- アプリAPIの呼び出しは、リクエストヘッダー `X-PCA-service-partition` に[サービス区画](../common/サービス区画.md)の設定が必須となる
- 事前に対象組織にサービス区画を追加することで、サービスからのアクセスを許可する

| サービス名 | サービス種類 <br/>（servie_kind） | 区画情報 | [利用できるクライアント](./oidc-oauth-clients.md) <br/> （client_id） |
| --- | --- | --- | --- |
| お客様サポート | pca.support | `customerid` <br/> ※固定値 | pca-apps-download |
| PCAクラウド | pca.cloud | 連携コード | pcaapp20-ui |
|  |  |  | pcacloud-devsite |
|  |  |  | pcacloud-api |
| PCAサブスク | pca.subsc | 連携コード | pcaapp20-ui |
| PCA Hub | pca.hub | テナント名 | pcahub-authn-ui |
|  |  |  | pcahub-management-api |
|  |  |  | xronosid-auth |
|  |  |  | xronos-ui |
|  |  |  | xronos-ui-mobile |
|  |  |  | xronos-management-api |
| PCA Arch | pca.arch | 組織ID | pca-portal-ui |
|  |  |  | pca-portal-api |
|  |  |  | pca-ai-assistant-api |
| ドリームホップ Res-Q | dreamhop.resq | 連携コード | dreamhop-resq |

## サービス詳細

### お客様サポート（pca.support）

- アクセス範囲は「すべての組織」とする
  - 組織へのサービス区画の追加は不要とし、すべての組織へのアクセスが許可済みとして扱う
- アクセス可能な項目は「[顧客ID](../common/組織.md#組織属性項目)」のみとする
- [指定可能なスコープ](./apps-integration-clients.md#oidcoauth-スコープ)は、以下の項目のみとする
  - openid
  - email
  - profile
  - id.customer_ids.read

### プロダクト系サービス

- アクセス範囲は「許可された組織」のみとする
  - 組織に対象サービスの「サービス区画」が追加されていればアクセスが許可される
- アクセス範囲内であれば、任意の項目にアクセスできる
- [任意のスコープ](./apps-integration-clients.md#oidcoauth-スコープ)を指定できる
- PCAクラウド／PCAサブスク、および任意の外部サービスは、区画情報に「連携コード」を使用する
  - それ以外のサービスは、それぞれの種類に応じた特殊な扱いとなる
