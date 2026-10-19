---
id: ace-wiki-home
title: GCP ACE Wiki ホーム
type: overview
topics: [ACE, GKE, network]
status: verified
verified_at: 2026-10-19
source_ids: [SRC-CHAT-20261019-01]
---

# GCP ACE 学習Wiki

一問一答を蓄積するのではなく、**出題条件からサービス・操作を選べる** ように整理します。各記事に公式リンクと確認日を保存します。

## GKE

- [既存クラスタへのノードプール追加とPod配置](gke/node-pools-and-scheduling.md) — マシンタイプ変更とバージョンアップを区別
- [VPCネイティブクラスタのプライマリ・セカンダリIP](gke/vpc-native-ip-ranges.md) — Node、Pod、Serviceの範囲と例外

## 学習時の使い方

- 手順を問われたら、まず **既存リソースの維持要件、対象リソース、変えたい属性** を確認する。
- IP割り当ての問題では、**Node / Pod / Service** と、**Standard / Autopilot、GKEのバージョン** を区別する。
- 詳細は [スキーマ](../SCHEMA.md) および [更新ルール](../AGENTS.md) を参照。

## 出典・検証

- 初期資料: `SRC-CHAT-20261019-01`
- 検証: 2026-10-19。リンク先記事ごとに公式Google Cloud文書を明記。
