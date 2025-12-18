---
sidebar_position: 9
---

# 9. CloudFront を経由するアクセス制御

## 目的

- PCA ID へのすべてパブリックアクセスは、常に CloudFront（本番環境なら `id.pca.jp`）を経由させる
  - [PCA ID システム構成図](./system-diagram.md)
- CloudFront に WAF を設置して、IP制限や各種インジェクション攻撃への対策などをおこなう

## PCA ID における CloudFront オリジンの種類

1. Keycloak（ALB + Fargate + ECS）
2. PCA ID 管理API（API Gateway + Lambda）
3. PCA ID フロントアプリ・画像・CSVオブジェクト（S3）

### 準備（`Referer`ヘッダー値）

- CloudFrontオリジンでのポリシー（アクセス条件）として利用するランダム値を用意しておく
- ランダム値は `Referer` ヘッダーを通じて利用する
  - `Referer` ヘッダーの値はローテーション可能にしておき、将来的には自動ローテーションを可能にする
- オリジン側で `Referer` ヘッダーをアクセス条件としてから、CloudFront 側で `Referer` ヘッダーを設定する

### 1. Keycloak（ALB + Fargate + ECS）

- ALB セキュリティグループにおいて「CloudFront マネージドプレフィックスリスト」からの HTTPS 通信のみ許可する

### 2. PCA ID 管理API（API Gateway + Lambda）

- CloudFront からの API Gateway オリジンへのアクセスに、ヘッダー `Referer` を利用したアクセス方法を設定する
  - API Gateway リソースポリシーにて、CloudFront で設定した `Referer` ヘッダー値のみ許可する
    - ローテーションを想定して、2つ以上の `Referer` ヘッダー値を許可できるようにする

### 3. PCA ID 画像・CSVオブジェクト（S3バケット）

- CloudFront からのS3バケットアクセスに Origin Access Control (OAC) を利用する
- **（重要）** S3バケットは「ブロックパブリックアクセス」を有効（デフォルト）のままとする

#### 参考情報

- [【アップデート】Amazon CloudFront を経由しないアクセスのブロックが簡単になりました \| DevelopersIO](https://dev.classmethod.jp/articles/amazon-cloudfront-managed-prefix-list/)
- [CloudFront のオリジンへ直接アクセスさせない方法まとめ \- サーバーワークスエンジニアブログ](https://blog.serverworks.co.jp/tech/2020/05/11/post-84652/)
- [Amazon CloudFrontの従来のOAIから新しくなったOACに移行してみた \| DevelopersIO](https://dev.classmethod.jp/articles/cloudfront-oai-to-oac/)
- [CloudFront\+OACでS3へのアクセスを制限してみた｜み](https://note.com/vast_curlew334/n/nb6ab3a6a4f59)
- [【Next\.js\+S3\+CloudFront】S3をパブリックかプライベートにアクセスする時の課題](https://zenn.dev/portinc/articles/next-s3-cloudfront)
- [API Gateway\(REST API\) のアクセス元を CloudFront のみに許可するには \| Grasswake‘s Blog](https://blog.grasswake.me/tech-posts/1646529783)
- [CloudFrontとWAFでフロントエンドとAPIを保護する｜ジロー@インフィニテック](https://note.com/hirozki/n/ncdece0191daa)

## PCA ID管理API（Lambda）からKeycloak（ALB）へのアクセス方法

- ALB へのアクセスは CloudFront 経由のみを認めるため、Lambda からの呼び出しでも CloudFront を通ることになる
  - ALBドメイン（`XXX-alb-XXX.ap-XXX.elb.amazonaws.com`）ではなく、外部公開ドメイン（例：`id.pca.jp`）でのアクセスとする
- Lambda から ALB へのアクセスは、外部向けの WAF ルールの影響を受ける
- アクセス制限のための WAF ルールは次のとおり
  > 環境パラメーターどおりの Referer ヘッダー値（のいずれか）が設定されている   ← Lambda から ALB へアクセスするケース
  > or
  > 許可された送信元IPが設定されている　← 上記以外のケース
- Cloudfront においては、自身で設定した `Referer` が存在していることを想定しておく必要がある
  - `Referer` ヘッダーが設定済みであれば、その値を維持する
