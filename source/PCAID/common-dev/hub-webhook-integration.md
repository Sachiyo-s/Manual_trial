---
sidebar_position: 14
---

# 14. PCA Hub Webhook 呼び出し仕様

※ *この文書は長南さんが担当しています*

## 現在の実装

単純なAPIキーでの保護だけで､IP制限など別の仕組みによる保護の必要性があります｡  
IP制限はNatゲートウェイの追加などでAWSの費用面で高いのと､今からやるには実装面でも面倒なことが懸念されます｡

## どうしたいのか

PCA HubのプライベートAPI(PCA Hubサービス間で利用する秘密のAPI)の方法に変更したいと考えています｡

### メリット

- Natゲートウェイ不要｡
- プログラム的には年末調整の成果物や､Hub側の成果物も使いまわし出来るため､実装インパクトは少なく対応が出来る
- セキュリティ的にも今よりは安全(キーの定期的な入れ替えも不要のはず)

### デメリット

- デメリットはOTPの計算が必要なので､開発環境であっても気軽に呼び出しにくい点です｡(メリットでもある)
  - 面倒なだけで頑張れば呼び出せる
- 多少だが､PCA Hub､PCA IDで実装の修正が必要
- PCA ID側の環境変数が増える

## 認証仕様

以下のリクエストヘッダーを伴って､ALBのパブリックDNS経由で Wevhook API を呼び出します｡  
ALBパブリックDNS経由のため､IP制限に引っかかることはありません｡  
X-PCA-TENANT以外は全ての環境(開発・検証・本番)で異なる点に注意してください｡  

この認証で必要なリクエストヘッダー

- Authorization
  - OTP方式(計算方法､SaltはPCA Hubに合わせる)で生成するワンタイムコード
- X-PCA-SHARED-INTERNAL-KEY
  - 秘密の合言葉(全PCA HubサービスとPCA IDが知っている)
- X-PCA-ISSUER
  - PCA Hubの Issuer
  - フォーマットは `https://{テナント名}.{環境毎に異なるPCA Hubドメイン}/auth`
- X-PCA-TENANT
  - テナント名(例: tenant1)
  - サービス区画名では無い点に注意

Webhook API固有で必要なヘッダーは変わらず必要

- X-PCA-Event-APIKey (キーとしては無くても良い気がし始めてるが､PCA Plus と PCA IDしか知らないという点では意味がある)
- X-PCA-organization-id
- X-PCA-service-partition
- X-PCA-Event-Type
- X-PCA-Event-Sender

## 実装方針

### PCA Hub側

認証属性を2つ利用するだけ｡  
※最終的なユーザーとしては､PcaIdCallbackAuthorizeが作成する PCA ID イベント用ユーザー(GOD権限)になる  
※呼び出しURLのパスが /g/**private**/api/v1/pcaidwebhooks/user-events となることに注意  

```cs
[PcaAuthAuthorize(PcaRoles.PcaInternalAdmin)]
[PcaIdCallbackAuthorize]
public class PcaIdWebhooksController : ControllerBase
```

#### 暫定対応について

PCA Hub側は PcaAuthAuthorizeを暫定的に外すことで､OTP無しでの呼び出しが可能です｡  
TDI様側の実装追いつくまではOTP無しで､以下のエンドポイントでイベントAPIが利用できるようにしてあります｡  
※必要なパラメータなどは今まで通り + `X-PCA-TENANTヘッダー` のみ  

```http
https://{ALBパブリックDNS}/g/private/api/v1/pcaidwebhooks/user-events
```

### PCA ID

年調からOTPの実装を移植して､呼び出し時のリクエストヘッダーはOTP用ヘッダー+Webhook API固有のヘッダーで呼び出す｡  
既存の実装の使い回すことで実装インパクトは少ないはず？(TDI様に要確認)  

## その他修正依頼

### PCA ID 側環境変数の持ち方

Hub側のエンドポイント組織､ユーザーなどの単位でエンドポイントを分けたいと考えています｡  
目的としては､同一エンドポイントで複数リクエストボディを受け取るのは避けたいというのが意図です｡  
※現状存在する､組織の作成イベントはHub側で処理しないため､Hubでは組織系イベントは利用されないです｡  

**PCA Hub固有**要件として､以下のように変更していただきたいと考えています｡  

#### 環境パラメーター

エンドポイントルートとパスの組み合わせで､呼び出し先を決定して頂きたいです｡  
例) ユーザー系イベントのエンドポイントとしては `https://{ALBパブリックDNS}/g/private/api/v1/pcaidwebhooks/user-events` となる｡

- Webhookエンドポイントルート
  - webhooks-endpoint-root: https://{ALBパブリックDNS}/g/private/api/v1/pcaidwebhooks
- ユーザー系イベントのパス
  - webhooks-user-path: /user-events
- 組織系イベントのパス
  - webhooks-organization-path: /organization-events

### Hub側Webhook API 失敗時の動作

1～2時間のメンテナンスはありえるため､それを見越したリトライ実装を入れて欲しいです｡  
最大5回のリトライを以下のルールでリトライしてください｡  
※ジッターは1～10のランダムな秒数  

```text
1回目:10秒後 + ジッター
2回目:300秒後 + ジッター
3回目:600秒後 + ジッター
4回目:1800秒後 + ジッター
5回目:6000秒後 + ジッター
5回試しても駄目ならデッドレターキューに移動
```

#### リトライ対象とする条件

- PCA Hub の API が ステータスコードが 500 エラー の場合
- PCA Hub の API が ステータスコードが 409 エラー の場合
  - 楽観的ロックによる同時実行制御エラーになると `409 Conflict` が返される
  - 他のイベントとセットになる `user.email-verified` イベント通知で発生することがある
- PCA Hub の API が ステータスコード 200 以外で、以下の **PCA Hub 標準のエラーレスポンス形式ではない場合**
  - CloudFront や ALB 等において、一時的にサービスが不安定になる AWS 由来のエラーを想定している

```json
{
  "status": 0,
  "code": "string",
  "message": "string",
  "domain": "string",
  "trace": "string"
}
```

### WebhookAPI呼び出し前のPCA119 APIの確認

キューを処理するLambda関数では､`https://{テナント名}.{PCA Hubドメイン}/g/api/pca119` を呼び出して､PCA Hub が正常に動作しているかを確認してください｡  
PCA119 API が 以下のレスポンスの場合だけWebhook APIを呼び出してください｡  
条件を満たさない場合は､即時にデッドレターキューにキューを積んでください｡  

- PCA119 API のステータスコードが 200 かつ､レスポンスの Status キー($.Status)の 値が 2 の場合

※ 必ずしもPCA119だけでメンテナンス状態かどうかを判定できませんが､
インターネット経由でアクセスされたくない状態であれば PCA119 API もエラーになります(データ不整合を防ぐのが目的です)

### キューのリトライ制御方法

- SQS のリトライ回数はキューがメッセージを返した回数([ApproximateReceiveCount](https://docs.aws.amazon.com/ja_jp/AWSSimpleQueueService/latest/APIReference/API_ReceiveMessage.html#API_ReceiveMessage_RequestParameters))を使ってカウント
- リトライの必要があれば可視性タイムアウトを更新

※キューが同時にLambda関数に拾われることについてはDynamoDBの[attribute_not_exists](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/Expressions.OperatorsAndFunctions.html#Expressions.OperatorsAndFunctions.Functions)条件で対応する([参考](https://engineering.dena.com/blog/2021/05/email-cloud-migration/#:~:text=%E3%81%9D%E3%81%93%E3%81%A7%E3%80%81DynamoDB%20%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E8%87%AA%E5%89%8D%E3%81%A7%E9%87%8D%E8%A4%87%E6%8E%92%E9%99%A4%E3%83%AD%E3%82%B8%E3%83%83%E3%82%AF%E3%82%92%E5%AE%9F%E8%A3%85%E3%81%97%E3%81%BE%E3%81%99%E3%80%82%20%E3%81%93%E3%82%8C%E3%81%AF%20PutItem%20%E3%83%AA%E3%82%AF%E3%82%A8%E3%82%B9%E3%83%88%E3%81%AB%20attribute_not_exists%20%E6%9D%A1%E4%BB%B6%E3%82%92%E3%81%A4%E3%81%91%E3%81%9F%E4%B8%8A%E3%81%A7%E3%80%81SQS%20%E3%81%AE%E3%83%A1%E3%83%83%E3%82%BB%E3%83%BC%E3%82%B8%20ID%20%E3%82%92%20DynamoDB%20%E3%81%AB%20PUT%20%E3%81%97%E3%81%A6%E3%81%84%E3%81%8F%E3%81%A0%E3%81%91%E3%81%A7%E5%AE%9F%E7%8F%BE%E3%81%A7%E3%81%8D%E3%81%BE%E3%81%99%E3%80%82(%E9%87%8D%E8%A4%87%E3%81%97%E3%81%A6%E3%81%84%E3%81%9F%E3%82%89%20PUT%20%E3%81%AB%E5%A4%B1%E6%95%97%E3%81%99%E3%82%8B%E3%81%AE%E3%81%A7%E3%81%9D%E3%82%8C%E3%82%92%E6%A4%9C%E7%9F%A5%E3%81%99%E3%82%8C%E3%81%B0%E3%81%84%E3%81%84)%20DynamoDB%20%E3%81%AB%E3%81%AF%20TTL%20%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%97%E3%80%81%E5%8F%A4%E3%81%84%E3%83%A1%E3%83%83%E3%82%BB%E3%83%BC%E3%82%B8%20id%20%E3%81%AF%E8%87%AA%E5%8B%95%E5%89%8A%E9%99%A4%E3%81%95%E3%82%8C%E3%82%8B%E3%81%9F%E3%82%81%E4%BF%9D%E5%AD%98%E3%82%B3%E3%82%B9%E3%83%88%E3%81%AE%E5%BF%83%E9%85%8D%E3%82%82%E3%81%82%E3%82%8A%E3%81%BE%E3%81%9B%E3%82%93%E3%80%82))

#### DynamoDB での排他ロック制御解除について

DynamoDB のTTLの他､関数を抜ける際に､明示的なロック解除実装もお願いします｡  
※予期せぬ例外が発生した場合についても考慮した実装(finally句で実装する)をお願いします  

- DynamoDB のTTL は180秒
- ttlを3分(180秒)として､バグっても5分または10分(2回目､3回目のリトライ)待てば良くなる
  - ※ttlは即時反映では無い場合もあります｡
  - バグの場合は5分間or10分間機能不全を許容｡
- バグが無いときは10秒後のリトライで実行される
  - 大体はこのタイミングで実行されて､たまたま失敗した場合のUXも許容範囲内と想定

なお､明示的ロック解除を実装する理由は､Hub側の都合で､想定よりセッションを長く維持できるようになっていた場合にロックが機能しなくなる可能性があるためです｡  
→ nginxコンテナの更新で設定が変わっていたなど(検証から漏れやすい部分なのが懸念点)

### Webhook API を呼び出すLamba関数のアプリケーションログ

Webhook API を呼び出すLamba関数のアプリケーションログを取るようにして頂きたいです。  
※PCA Hub の Webhook APIのログの、トレースIDを使って、PCA IDのアプリケーションログを調査できるようにするのが目的です。  

- mt に `Call PCA Hub Webhook (tenantname:{テナント名}, accountid:{対象ユーザーのアカウントID})` を指定をお願いします。
- PCA Hub の Webhook API の呼び出し結果が200以外の場合のレスポンスボディを mt に含めて欲しいです。
  - `Error PCA Hub Webhook (tenantname:{テナント名}, accountid:{対象ユーザーのアカウントID}, res:{レスポンスボディ})`

### PCA Hub 側 のWehook API を呼び出す場合の注意

Hub側の同一ユーザーに対して連続して Hub 側の Webhook API を呼び出す場合は2秒の間隔を開けてください。  
この対応を行わないと、Hub側のユーザー更新で、楽観的同時実行制御エラーになって失敗します。  
