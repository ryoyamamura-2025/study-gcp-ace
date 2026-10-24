---
id: ace-wiki-home
title: GCP ACE Wiki ホーム
type: overview
topics: [ACE, official-sample, GKE, network, load-balancing, Pub/Sub, Cloud SQL, backup, PITR, BigQuery, IAM, Compute Engine, SSH, OS Login, IAP]
status: verified
verified_at: 2026-10-24
source_ids: [SRC-CHAT-20261019-01, SRC-CHAT-20261020-SSH-01, SRC-OFFICIAL-SAMPLE-20261023-01]
---

# GCP ACE 学習Wiki

一問一答を蓄積するのではなく、**出題条件からサービス・操作を選べる** ように整理します。各記事に公式リンクと確認日を保存します。

## 公式模試の出題傾向

- [公式模試から学ぶサービス選定・判断軸と注意点](exam/official-sample-decision-patterns.md) — 最小権限・最小運用・費用対効果、模試の正解と実運用の条件の区別

## ネットワーク / DNS

- [ロードバランサ選定：L7/L4、外部/内部、グローバル/リージョン](network/load-balancer-selection.md) — URLマップ、ヘルスチェック、TLS終端とNLBプロキシ/パススルー

- [2PBのデータ移行: Transfer ApplianceとInterconnectの違い](network/transfer-appliance-vs-interconnect.md) — オフライン物理輸送、回線帯域、容量、PSCとの使い分け

- [外部公開DNS、内部DNS、A/CNAME、エイリアスIPの違い](network/dns-public-internal-a-cname.md) — 公開ゾーンと内部DNS、固定外部IPと名前解決、DNSとルーティングの区別

## Pub/Sub

- [Fan-out・同一Subscription内の負荷分散・ACKと再配信](pubsub/fan-out-load-balancing-ack.md) — Topic/Subscription/Subscriber、At-least-once、Exactly-once、BigQuery Subscription

## GKE

- [HPA・VPA・Cluster Autoscalerの違いとPodを増やす意味](gke/autoscaling.md) — Podレプリカ数、リソース要求、VM台数の判断軸
- [既存クラスタへのノードプール追加とPod配置](gke/node-pools-and-scheduling.md) — マシンタイプ変更とバージョンアップを区別
- [VPCネイティブクラスタのプライマリ・セカンダリIP](gke/vpc-native-ip-ranges.md) — Node、Pod、Serviceの範囲と例外

## Cloud SQL

- [自動・オンデマンドバックアップとPITR](cloud-sql/backups-and-pitr.md) — MySQLバイナリログ、ログ保持、復元先と既定設定を含む

## BigQuery

- [SQLジョブ実行とデータ操作のIAMロール](bigquery/iam-query-roles.md) — `jobUser` / `user` と `dataViewer` / `dataEditor` の違い、組み合わせ、最小権限

## Compute Engine / SSHアクセス

- [SSH公開鍵・プロジェクト全体鍵・OS Login・IAP](compute-engine/ssh-public-keys-os-login-iap.md) — A・Bの鍵共有、複数VMへの適用、認証と通信経路の違い

## 学習時の使い方

- Pub/Subは **複数のSubscriptionに同じメッセージを配信するFan-out / 1つのSubscriptionの複数Workerで分担する負荷分散** を区別する。

- LBの選定は **HTTP(S)のURL単位の制御が要るか / インターネット公開か / 複数リージョンか** を分ける。

- 公式模試の設問では、Googleの推奨手法、運用工数、コスト、最小権限のうち何が要求されているかを先に読む。

- 手順を問われたら、まず **既存リソースの維持要件、対象リソース、変えたい属性** を確認する。
- GKEの拡張問題では、**アプリのPodレプリカ数 / Podの要求CPU・メモリ / ノードの台数** を分ける。
- IP割り当ての問題では、**Node / Pod / Service** と、**Standard / Autopilot、GKEのバージョン** を区別する。
- Cloud SQLの復元方式では、**定期取得 / 明示取得 / 特定時刻の復元**を区別する。
- BigQueryのSQL権限では、**ジョブを実行できるか / 対象データを参照・編集できるか** を別々に確認する。
- SSHの設問では、**公開鍵による認証 / IAMによるOSログイン権限 / VMへのネットワーク経路**を分けて判断する。
- 詳細は [スキーマ](../SCHEMA.md) および [更新ルール](../AGENTS.md) を参照。

## 出典・検証

- 初期資料: `SRC-CHAT-20261019-01`、追加会話の公開用要点: `SRC-CHAT-20261020-SSH-01`
- 検証: 2026-10-23（一部初期記事は2026-10-19〜20）。リンク先記事ごとに公式Google Cloud文書を明記。
