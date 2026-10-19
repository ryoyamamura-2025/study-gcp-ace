---
id: gke-autoscaling-comparison
title: GKEのPodとノードのオートスケーリング（HPA・VPA・Cluster Autoscaler）
type: comparison
topics: [GKE, Kubernetes, HPA, VPA, Cluster Autoscaler, Pod, node, resource-requests, scaling, Standard, Autopilot]
status: verified
verified_at: 2026-10-20
source_ids: [SRC-CHAT-20261019-01]
---

# GKE: Podを増やす意味とノードを増やす意味

## 要点

- **HPA（Horizontal Pod Autoscaler）**：ワークロードの**Podレプリカ数**を負荷などの指標に応じて増減し、処理を複数のPodに分担させる。
- **VPA（Vertical Pod Autoscaler）**：Pod内のコンテナの**CPU・メモリの要求量（requests）など**を調整する。Podのレプリカ数を増やす仕組みではない。
- **Cluster Autoscaler**：GKE Standardの**ノードプールのノード数**を、Podの配置に必要な容量に応じて増減する。
- **Pod増＝VMのCPU増ではない**。既存ノードに空きリソースがあるときは、Pod数を増やすだけで処理を並列化できる。一方、既存の物理・仮想CPU容量を使い切っているなら、Podだけ増やしても計算資源は増えない。

## 仕組み・判断軸

### 1. ノード・Pod・アプリの関係

- ノードはPodを実行するマシン（GKE Standardでは通常Compute Engine VM）。
- Podは1個以上のコンテナを持つKubernetesの実行・配置単位。1つのDeploymentのレプリカとして同じアプリのPodを複数動かせる。
- WebアプリでPodレプリカを増やし、Serviceなどでリクエストを複数Podに振り分けると、十分な計算資源がある範囲では並列処理能力や可用性を改善できる。
- ただしPod数に比例してスループットが増えるとは限らない。CPU上限、DB、共有リソース、処理特性などがボトルネックになりうる。

### 2. オートスケーラーの役割を対比する

| 機能 | 何を変える？ | 主な判断材料 | 増えないもの |
| --- | --- | --- | --- |
| HPA | 対象ワークロードのPodレプリカ数 | CPU/メモリ使用率、カスタム/外部指標など | ノードの計算資源 |
| VPA | Pod内のコンテナのCPU・メモリrequests/limitsなど | リソース使用履歴・推奨値 | Podレプリカ数 |
| Cluster Autoscaler | ノードプールのノード台数 | 配置不能Podの要求リソース、制約、配置可能性 | アプリ自体のPodレプリカ数 |

### 3. HPAとCluster Autoscalerの典型的な連携

1. Webアプリへのアクセスが増え、HPAがDeploymentのPodを3個から8個へ増やそうとする。
2. **既存ノードに空きがあれば**、増えたPodはそこに配置される。**ノードの追加は不要**。
3. **Podのresource requests等を満たす配置先がない**と、一部Podがスケジュール不能（Pending / PodUnschedulable）になる。
4. Cluster Autoscalerを有効化しており、設定した最小・最大サイズ、クォータ、ノード構成などの条件を満たせば、Podを配置できるようノードの追加を検討する。
5. 新しいノードにPodが配置される。なおCluster Autoscalerが動いても、すべての配置不能Podが必ず解消するとは限らない（1 Podが新規ノードにも入らない、制約やクォータがある等）。

**重要**：Cluster Autoscalerは単純な「ノードのCPU使用率が高いので増やす」仕組みではない。Podのresource requestsとスケジューリング可能性が中心。実際のCPU使用率が高くても、追加Podがなく配置可能なら増えないことがある。

### 4. 「Podを増やすだけ」で性能が伸びる条件

例：ノードに余裕があり、同じアプリを実行するPodを2個から4個へ増やせる場合。トラフィックを分散できれば、処理待ちを減らせる可能性がある。

反対に、ノードのCPUが既に飽和している場合、同じノード上にPodを追加してもCPU総量は増えないため、4倍の処理能力にはならない。ノードを追加するなどして計算資源を確保する必要がある場合がある。

※ CPU 4コア＝「1コア要求するPodを厳密に4個置ける」ではない。Kubernetesの配置にはPodの**requests**とノードの**allocatable**リソースが関係し、システム予約や他の制約がある。

### 5. 新しいアプリが続々デプロイされる場合（ACEの典型問題）

要件が「新しいアプリの追加によって**クラスタ容量が不足したら、手動でノードを増やさず対処したい**」なら、GKE Standardでは**ノードプールでCluster Autoscalerを有効化し最小・最大ノード数を設定**するのが基本。

- HPAは各ワークロードの負荷に応じてPod数を増減する。新しいアプリが加わるだけでクラスタ容量を自動拡張する仕組みではない。
- VPAは各Podのリソース要求の最適化であり、ノード台数の自動調整を主目的としない。
- アプリごとに専用ノードプールを手作業で作るのは「手動介入を避けたい」に反する。
- **Autopilot**ではノードの管理と必要容量のプロビジョニングがGKE側で自動化される。Standardのノードプール設定を前提にした説明と区別する。

## 間違えやすい点

- **誤**：「Podを3倍にすればCPUが3倍になる」→ **正**：Podの並列実行単位が増えるだけ。計算資源が不足すれば頭打ち。
- **誤**：「ノードを3倍にすればアプリも3倍起動する」→ **正**：ノードを増やすだけでDeploymentのレプリカ設定が増えるわけではない。
- **誤**：「HPAがPodを増やしたら必ずノードも増える」→ **正**：既存ノードに配置できればノード追加は不要。
- **誤**：「Cluster AutoscalerはCPU使用率が80%を超えると増える」→ **正**：主に配置できないPodの有無とresource requests・制約を判断する。
- **誤**：「VPAはHPAと同じくPodを増やす」→ **正**：VPAはPodのサイズ（requests/limits）を調整する。更新モードによってはPod再作成を伴う。
- HPAとVPAの同一ワークロードへの併用にはメトリクス等の制約があるため、無条件に併用可能とは考えない。

## ACE試験の選定キーワード

- 「アクセス増、アプリのレプリカ数を自動増減」→ **HPA**
- 「PodのCPU・メモリ割当要求を適正化」→ **VPA**
- 「新しいPodを配置できない／クラスタ容量の自動拡張」→ **Cluster Autoscaler（Standardのノードプール）**
- 「別マシンタイプのノードを既存Podを維持しながら追加」→ **別ノードプール**（[ノードプールとPod配置](node-pools-and-scheduling.md)）

## 関連ページ

- [GKEのノードプールとPodの配置](node-pools-and-scheduling.md)
- [GKE VPCネイティブのIP範囲](vpc-native-ip-ranges.md)
- [Wikiホーム](../index.md)

## 出典・検証

Google Cloud公式文書を **2026-10-20** に確認:

- [Horizontal Pod autoscaling](https://docs.cloud.google.com/kubernetes-engine/docs/concepts/horizontalpodautoscaler)
- [Vertical Pod autoscaling](https://docs.cloud.google.com/kubernetes-engine/docs/concepts/verticalpodautoscaler)
- [About GKE cluster autoscaling](https://docs.cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler)
- [Troubleshoot cluster autoscaler scale-up](https://docs.cloud.google.com/kubernetes-engine/docs/troubleshooting/cluster-autoscaler-scale-up)

会話由来の学習論点: `SRC-CHAT-20261019-01`（本題に対する理解上の混同点と、Podを増やす意味についてのやり取り）。講座の設問原文は未収録。
