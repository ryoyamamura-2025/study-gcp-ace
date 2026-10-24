---
id: pubsub-fanout-load-balancing-ack
title: Pub/SubのFan-out、Subscriber負荷分散、ACKと再配信
type: comparison
topics: [ACE, Pub/Sub, topic, subscription, subscriber, fan-out, load-balancing, ACK, ack-deadline, retry, dead-letter-topic, idempotency, exactly-once, BigQuery-subscription]
status: verified
verified_at: 2026-10-24
source_ids: []
---

# Pub/Sub: Fan-out、負荷分散、ACK/再配信

## 要点

- **Topic**: Publisherがメッセージを送信する宛先。
- **Subscription**: 特定のTopicに紐づく独立したメッセージ配信経路と処理状態。1つのTopicに複数Subscriptionを作ればそれぞれに同じメッセージを配信できる（**Fan-out**）。
- **Subscriber**: SubscriptionからPullする、またはPushで受ける実際の処理アプリ。1つのPull Subscriptionに複数Subscriberが接続すると、通常はメッセージを分担する（**負荷分散**）。
- **ACK**: そのSubscriptionに対する処理完了通知。別Subscriptionの配信・ACKには影響しない。
- **ACK Deadline**: ACK期限。期限切れまたはNACKで再配信対象となる。
- 標準では**At-least-once**配信。重複配信・重複処理に備えた**べき等性**が重要。

## 仕組み・判断軸

### Fan-outと負荷分散の違い

たとえばTopic `orders` に注文M1、M2、M3が発行され、在庫管理と分析の双方が全注文を必要とする場合:

- Subscription A（在庫）とSubscription B（分析）を`orders`にそれぞれ作成する。各SubscriptionがM1/M2/M3を独立して受け取る。
- 在庫側のSubscription AにWorker 1/Worker 2が接続する。通常、各Workerはメッセージのサブセットを処理する。配分割合・順序は保証されない。
- 在庫側がACKに失敗しても、分析側Subscription BのACK状態とは独立。別アプリを同じSubscriptionに接続するとメッセージを分担し、両者に全件を渡せない。

### ACK、NACK、ACK Deadline

Pull Subscriberはメッセージを処理してACKする。ACK期限内にACKされなければ再配信が試行され、同じSubscriptionの別Workerに届くこともある。NACKも再配信の対象。

- サブスクリプションのACK Deadlineの通常の既定値は**10秒**、指定可能範囲は**10〜600秒**。
- 個々のメッセージは`ModifyAckDeadline`によって期限を延長でき、高水準クライアントライブラリは期限延長（lease management）を自動で行う場合がある。
- Exactly-once enabledでACK期限を作成時に省略した場合には**60秒**という既定値の例外がある。
- 標準の再試行ポリシーは即時再試行。指数バックオフやDead-letter Topicを設定できる。
- ACK成功後であっても標準At-least-once配信では稀な重複があり得る。注文ID等による重複検出、べき等なデータ更新が必要。

### Exactly-onceの条件

Pub/SubのExactly-once deliveryは**Pull/StreamingPull Subscriptionのみ**対応し、同一リージョンのサービスエンドポイントを利用する場合に保証される。Push/Export Subscriptionは非対応。ACK期限切れ/NACKによる正規の再配信までは排除しない。外部DBへの副作用の完全な1回実行を自動保証しないので、処理進捗やべき等性を設計する。

### BigQuery連携

データ加工が不要なら**BigQuery Subscription（Export）**でSubscriberアプリを用意せずBigQueryテーブルに直接書ける。複雑な変換・集計はDataflowを検討する。BigQuery SubscriptionはAt-least-once配信のため重複対策が必要な場合がある。

## 間違えやすい点

- Topic自体に各ワーカーのACK状態を置くわけではない。ACKはSubscriptionごと。
- 同じSubscriptionを複数アプリで共有しても、全アプリへ全件が複製されるわけではない。
- ACKは「別のSubscriberに送信せよ」という合図ではなく「このSubscriptionでは処理完了」という合図。
- 「複数Subscription」だけではWorker単位の並列負荷分散を説明できない。両者は組み合わせ可能。
- 標準Pub/Subは厳密な一度だけの業務処理や全体順序を自動保証しない。順序保証はordering key等の要件がある。

### ACE試験の選定基準

| 要件 | まず考える構成 |
| --- | --- |
| 同一イベントを異なるアプリで独立に処理 | 1 Topic + アプリごとにSubscription（Fan-out） |
| 大量イベントの処理能力を増やす | 1 Subscription + 複数Pull Subscriber |
| ACK前にWorkerが停止 | ACK Deadline経過 → 再配信を試みる |
| 何度も失敗するメッセージを隔離 | Dead-letter Topic |
| 重複する注文処理を回避 | べき等な処理、必要に応じExactly-once機能 |
| BigQueryにそのまま書く | BigQuery Subscription |
| 複雑な変換・集計を伴う処理 | Dataflow |

## 関連ページ

- [Wikiホーム](../index.md)
- [公式模試のサービス選定ポイント](../exam/official-sample-decision-patterns.md)

## 出典・検証

Google Cloud公式資料を2026-10-24に確認:

- [Pub/Sub service overview](https://docs.cloud.google.com/pubsub/docs/pubsub-basics)
- [Subscription overview](https://docs.cloud.google.com/pubsub/docs/subscription-overview)
- [Subscription properties](https://docs.cloud.google.com/pubsub/docs/subscription-properties)
- [Lease management](https://docs.cloud.google.com/pubsub/docs/lease-management)
- [Exactly-once delivery](https://docs.cloud.google.com/pubsub/docs/exactly-once-delivery)
- [BigQuery subscriptions](https://docs.cloud.google.com/pubsub/docs/bigquery)
