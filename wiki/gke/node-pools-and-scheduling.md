---
id: gke-node-pools
title: GKEで別マシンタイプのPodを追加する
type: procedure
topics: [GKE, Standard, node-pool, scheduling, machine-type, gcloud]
status: verified
verified_at: 2026-10-19
source_ids: [SRC-CHAT-20261019-01]
---

# GKE: 稼働中のクラスタに異なるマシンタイプのPodを追加

## 要点

**GKE Standardで既存Podを稼働させたまま異なるマシンタイプのノードが必要な場合は、対象マシンタイプの新しいノードプールを追加し、新しいPodを配置する** のが基本。既存ノードプールを無理に変更する必要はない。

例: 既存の `n1-standard-2` ノード上にアプリが稼働し、新たに `n2-highmem-16` ノードを必要とするワークロードを追加するとき。

## 仕組み・判断軸

- **クラスタ**: Kubernetesの管理単位。
- **ノードプール**: ノード群を管理する単位。Standardではノードプール作成時に `--machine-type` でマシンタイプを指定できる。
- **Pod**: Kubernetesスケジューラが利用可能なノードへ配置する。特定のノードプールへ置きたいときは、GKEが付与する `cloud.google.com/gke-nodepool` ラベルを `nodeSelector` などで指定できる。
- 別のプールを足す方式は、既存プールを維持したままリソースを増やせる点が利点。必要なvCPU・メモリ割当量、クォータ、リージョン/ゾーンのマシン可用性、IPアドレス枠を確認する。
- アプリケーションの**無停止**は、単にノードを追加しただけで自動保証されない。既存ワークロードを変更する場合はレプリカ、readiness probe、Deploymentの更新戦略、PodDisruptionBudgetなども影響する。

## 間違えやすい点

| 手段 | 本来の目的 | 今回の使い方 |
| --- | --- | --- |
| `gcloud container node-pools create` | 既存Standardクラスタへノードプールを追加 | 新マシンタイプのプール追加に適する |
| `gcloud container clusters upgrade` | コントロールプレーン/ノードプールのKubernetesバージョン更新 | **別マシンタイプのプールを追加する操作ではない** |
| 既存プールのマシン属性変更 | 既存ノード群を別構成へ移行 | 可能なケースはあるが、既存Podをそのまま残して追加する要件では優先しない |

※ 現行GKE Standardでは既存ノードプールのマシン属性変更も可能。したがって「マシンタイプは絶対に変更できない」という説明も誤り。

## 関連ページ

- [HPA・VPA・Cluster AutoscalerとPod/ノード数](autoscaling.md) — Pod数とノード数の自動拡張を区別
- [VPCネイティブIP範囲](vpc-native-ip-ranges.md) — 新しいノードやPodのIPが不足すると追加に失敗しうる
- [Wikiホーム](../index.md)

## 出典・検証

公式文書を **2026-10-19** に確認:

- [Add and manage node pools](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/node-pools)
- [gcloud container node-pools create](https://docs.cloud.google.com/sdk/gcloud/reference/container/node-pools/create)
- [Manually upgrade a cluster](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/upgrading-a-cluster)
- [Migrate nodes to a different machine type](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/migrate-nodes)

会話由来の論点: `SRC-CHAT-20261019-01`。講座原文は未収録。
