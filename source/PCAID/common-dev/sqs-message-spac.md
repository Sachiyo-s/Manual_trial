---
sidebar_position: 7
---

# 7. SQS メッセージ仕様

| データ種類 | キュー名 | 用途 | 可視性タイムアウト | 保持期間 | DLQ <br/>(retry) | 備考 |
| --- | --- | --- | --- | --- | :---: | --- |
| イベント通知 | pcaid-event-hook-queue | Keycloak イベント受付 | ・デフォルト 30秒 <br/> ・リトライ時 5, 10, 20, 40, 80s | 1時間 | - | Ver2.0~ |
| 　〃 | pcaid-event-pcahub-queue | PCA Hub 連携 | ・デフォルト 30秒 <br/> ・リトライ時 10, 300, 600, 1800, 6000s | 12時間 | あり<br/>（5回） | |
| 　〃 | pcaid-event-pcahub-dead-letter-queue | PCA Hub 連携 DLQ | - | 14日 | - |  |
| 　〃 | pcaid-event-internal-system | 社内システム連携 | 60秒 | 14日 | - | 現在未使用（2025/03） |
| メール送信 | pcaid-email-sender-event-queue | メール送信量制御 | 30秒 | 10分 | - | Ver2.0~ |
