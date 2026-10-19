---
id: gke-vpc-native-ip
title: GKE VPCネイティブのプライマリ・セカンダリIP
type: concept
topics: [GKE, VPC-native, primary-range, secondary-range, alias-IP, service-CIDR]
status: verified
verified_at: 2026-10-19
source_ids: [SRC-CHAT-20261019-01]
---

# GKE: VPCネイティブのプライマリIPとセカンダリIP

## 要点

**プライマリIP範囲は主にGKEノード用、セカンダリIP範囲はPod用**。ServiceのClusterIPは、別のセカンダリ範囲を使う構成に加え、**GKE管理のService IP範囲を使う構成**もある。

VPCネイティブはエイリアスIPを使うGKEネットワーク方式。新規クラスタではデフォルトのモード。

## 仕組み・判断軸

| 対象 | IPv4アドレスの取得元 | 押さえる点 |
| --- | --- | --- |
| Node | VPCサブネットの**プライマリ**IPv4範囲 | ノード自体が持つIP |
| Pod | VPCサブネットの**セカンダリ**IPv4範囲（Pod用） | GKEは各ノードへPod用エイリアスIP範囲を割り当てる |
| Service (ClusterIP) | サブネットのService用**別セカンダリ**範囲、または **GKE管理範囲** | Pod用範囲と混同しない |

### Service IPの重要な例外

GKE **Autopilot 1.27以降** と **Standard 1.29以降** のクラスタでは、ServiceのIPにGoogle管理の `34.118.224.0/20` が**デフォルトで**使われる構成がある。既存クラスタや明示的な設定ではサブネットのService用セカンダリ範囲を使う場合がある。

したがって、旧来の「Node=プライマリ、PodとService=セカンダリ」という暗記だけでは、すべての現行構成を正確に表せない。

### 見分けるためのイメージ

- **プライマリ**: サブネットが標準で持つ、VMなどのネットワークインターフェース用IPv4範囲。
- **セカンダリ**: サブネットに追加する別のIPv4範囲。GKEのPodアドレス空間などに割り当てられる。
- **エイリアスIP**: ノードのネットワークインターフェースに関連付けられ、同じノードのPodが利用するIP範囲を扱う仕組み。ServiceのClusterIPをそのまま「ノードに付けたエイリアスIP」と説明しない。

## 間違えやすい点

- Pod用とService用のIP範囲は同一だとは限らない。従来のサブネット管理方式では別のセカンダリ範囲を使う。
- Nodeが増えればプライマリ範囲、Podが増えればPod用セカンダリ範囲のIP容量が課題になる。
- Serviceの取得元は、**GKEモード、バージョン、作成時設定**を確認して判断する。
- エイリアスIP方式の **VPCネイティブ** と、静的ルートを使う **routes-based** クラスタは異なる方式。

## 関連ページ

- [ノードプール追加とPod配置](node-pools-and-scheduling.md)
- [Wikiホーム](../index.md)

## 出典・検証

公式文書を **2026-10-19** に確認:

- [VPC-native clusters / IPv4 address allocation](https://docs.cloud.google.com/kubernetes-engine/docs/concepts/alias-ips)

会話由来の論点: `SRC-CHAT-20261019-01`。講座原文は未収録。
