---
id: ace-wiki-home
title: GCP ACE Wiki ホーム
type: overview
topics: [ACE, GKE, network, Cloud SQL, backup, PITR, BigQuery, IAM]
status: verified
verified_at: 2026-10-20
source_ids: [SRC-CHAT-20261019-01]
---

# GCP ACE 学習Wiki

一問一答を蓄積するのではなく、**出題条件からサービス・操作を選べる** ように整理します。各記事に公式リンクと確認日を保存します。

## GKE

- [HPA・VPA・Cluster Autoscalerの違いとPodを増やす意味](gke/autoscaling.md) — Podレプリカ数、リソース要求、VM台数の判断軸
- [既存クラスタへのノードプール追加とPod配置](gke/node-pools-and-scheduling.md) — マシンタイプ変更とバージョンアップを区別
- [VPCネイティブクラスタのプライマリ・セカンダリIP](gke/vpc-native-ip-ranges.md) — Node、Pod、Serviceの範囲と例外

## Cloud SQL

- [自動・オンデマンドバックアップとPITR](cloud-sql/backups-and-pitr.md) — MySQLバイナリログ、ログ保持、復元先と既定設定を含む

## BigQuery

- [SQLジョブ実行とデータ操作のIAMロール](bigquery/iam-query-roles.md) — `jobUser` / `user` と `dataViewer` / `dataEditor` の違い、組み合わせ、最小権限

## 学習時の使い方

- 手順を問われたら、まず **既存リソースの維持要件、対象リソース、変えたい属性** を確認する。
- GKEの拡張問題では、**アプリのPodレプリカ数 / Podの要求CPU・メモリ / ノードの台数** を分ける。
- IP割り当ての問題では、**Node / Pod / Service** と、**Standard / Autopilot、GKEのバージョン** を区別する。
- Cloud SQLの復元方式では、**定期取得 / 明示取得 / 特定時刻の復元**を区別する。
- BigQueryのSQL権限では、**ジョブを実行できるか / 対象データを参照・編集できるか** を別々に確認する。
- 詳細は [スキーマ](../SCHEMA.md) および [更新ルール](../AGENTS.md) を参照。

## 出典・検証

- 初期資料: `SRC-CHAT-20261019-01`
- 検証: 2026-10-20（一部初期記事は2026-10-19）。リンク先記事ごとに公式Google Cloud文書を明記。
