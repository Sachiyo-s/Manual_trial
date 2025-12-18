---
sidebar_position: 55
---

# 55. タイムアウトとリトライ設定

## AWSサービス（AWS SDK for JavaScript v3）

- リトライモード（retryMode）：standard 　※デフォルト
  - リトライ回数：３回（２回まで再試行）
  - リトライ間隔：エクスポネンシャルバックオフ（0, 1秒, 2秒, 4秒, 8秒, ...）
- 接続タイムアウト（connectionTimeout）： 2秒
- 要求タイムアウト（requestTimeout）： S3 のみ 15秒、その他：5秒

```js
// Example: setting timeouts.
import { NodeHttpHandler } from "@smithy/node-http-handler";

new DynamoDBClient({
  requestHandler: new NodeHttpHandler({
    // time limit (ms) for receiving response.
    requestTimeout: 5_000,

    // time limit (ms) for establishing connection.
    connectionTimeout: 2_000,
  }),
});
```

[AWS SDKを使ったAWSサービスへのアクセスについて、タイムアウトとリトライを設定する · Issue \#1063 · pca\-idp/pcaid](https://github.com/pca-idp/pcaid/issues/1063)

## Keycloak API（REST API 全般）

- リトライ回数：３回（２回まで再試行）
- リトライ間隔：エクスポネンシャルバックオフ（0, 1000ms, 2000ms, 40000ms, 8000ms, ...）
- タイムアウト：無期限

```js
const fetchWithRetry = async (input: RequestInfo, init: RequestInit, retries = 3) => {
    for (let i=0; i<retries; i++) {
        // exponential backoff algorithm, 0, 1000, 2000, 4000
        const delay = 1000 * 2 ** i
        i>0 ? console.info(`${i} retry`) : null
        try {
            const response = await fetch(input, init)
            return response
        } catch (error) {
            if ((error.code == 'UND_ERR_CONNECT_TIMEOUT' || error.code == 'UND_ERR_SOCKET' || error.code == 'ETIMEDOUT' || error.code == 'ECONNRESET') && i<retries-1) {
                await new Promise(resolve => setTimeout(resolve, delay))
            } else {
                throw error
            }
        }
    }
}
```

[Keycloak呼び出しのリトライ対応を検討する · Issue \#765 · pca\-idp/pcaid](https://github.com/pca-idp/pcaid/issues/765)
