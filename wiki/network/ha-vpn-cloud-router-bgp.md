---
id: ace-ha-vpn-cloud-router-bgp
title: HA VPN・Classic VPN・Cloud RouterとBGP動的ルーティング
type: comparison
topics: [ACE, HA VPN, Classic VPN, Cloud Router, BGP, dynamic-routing, static-routing, Cloud NAT, VPC Peering, Cloud Interconnect]
status: verified
verified_at: 2026-10-24
source_ids: []
---

# HA VPN、Classic VPN、Cloud RouterとBGPの選び方

## 要点

- オンプレミスとVPCの間で、**インターネット上のIPsec暗号化通信・高可用性・BGPによる経路自動更新**が必要なら、**HA VPN + Cloud Router**。
- HA VPNが暗号化されたVPNトンネルを提供し、Cloud RouterがBGPで経路情報を交換する。Cloud Routerそのものはパケットを転送しない。
- **Classic VPN**は静的ルーティング向け。Classic VPNのBGP動的ルーティングは**2025-08-01に非推奨となり、新規作成は不可**。既存のBGP構成はサポート対象外でSLAがない。
- **VPC Peering**はVPC同士の接続であり、単体ではオンプレミスまでの暗号化VPNを構築しない。**Cloud NAT**はIPアドレス変換でありIPsec接続を構築しない。

## 仕組み・判断軸

| 条件 | 選択候補 | 理由 |
| --- | --- | --- |
| オンプレとGoogle Cloudのプライベートネットワークをインターネット経由で暗号化接続 | Cloud VPN | サイト間IPsec |
| 高可用性と経路自動変更、対向がBGP対応 | HA VPN + Cloud Router | 冗長トンネルとBGP |
| 対向がBGP未対応で静的経路が必須 | Classic VPN | 現在は静的ルート構成 |
| 大きな帯域の専用接続 | Dedicated/Partner Interconnect | VPNとは接続方式が異なる。暗号化が必要ならHA VPN over Cloud Interconnectも検討 |
| 異なるGoogle Cloud VPC同士の接続 | VPC Network Peering | VPC同士の到達性。別途VPNにはならない |
| VMなどの外向き通信のNAT | Cloud NAT | アドレス変換でありBGP VPNトンネルを作らない |

### HA VPN

HA VPNゲートウェイは2つのインターフェースとそれぞれ外部IPを持ち、2つのインターフェースのそれぞれにトンネルを構成することで高可用性を実現する。

- オンプレミスのVPNルーターがBGPに対応していることが前提。
- 適切なピア側構成・2トンネル等の条件を満たすトポロジでは最大**99.99% SLA**。1トンネルだけではこのSLA条件を満たさない。HA VPNなら無条件に99.99%とは言わない。
- **Classic VPN**は1インターフェース、静的ルーティング、99.9% SLA。

### BGPによる経路交換

例：オンプレ側ルーターから`10.30.0.0/24`をBGPで広報すると、対応するCloud Routerが学習し、VPCの動的ルートに反映できる。経路変更時にも新しいVPNトンネルの作成や毎回の手動ルート追加が不要になる。

**条件**: 新しいサブネットがオンプレミスルーター側でBGPの広報対象になること。VPN構成だけでオンプレ側のあらゆるネットワークを自動検出するわけではない。

Cloud Routerが担うのはBGPセッション/経路広告/学習であり、実データパケットのルーティング・転送自体はGoogle Cloudネットワークが行う。

### Regional / Global dynamic routing

VPCの動的ルーティングモードを`REGIONAL`（デフォルト）と`GLOBAL`から選ぶ。

- `REGIONAL`: 学習ルートが基本的にVPN/Cloud Routerのリージョンへ適用。
- `GLOBAL`: 他リージョンにも学習ルートが適用される。オンプレ側へ広告するサブネット範囲にもモードによる差がある。
- 同じVPCの全Cloud Routerの動作に影響するモードであり、Cloud Routerごとに独立した動的ルーティングモードを設定するわけではない。

### VPC Peeringを使う際の補足

PeeringはVPC同士を接続するサービス。Peering相手のVPCに既存Cloud VPN/Interconnectがある場合、**カスタムルートのimport/exportとCloud Routerのカスタム広告**などを設定すれば、オンプレミスとの到達性を広げられる場合がある。しかしPeering自体はVPNトンネルを確立せず、VPC間Peering自体は非推移的（A↔B、B↔CからA↔Cが自動的に通信できるわけではない）。

### Cloud NATとCloud Routerの注意点

Cloud NATもCloud Routerのコントロールプレーン機能を使うが、**BGPセッション管理には使わない**。Cloud NATとCloud Routerを作っただけではVPNにも経路自動交換にもならない。

## 間違えやすい点

- 「HA VPNを設定するだけで99.99% SLA」という主張は誤り。対向構成、複数インターフェース/トンネルなどの条件を確認。
- 古い教材ではClassic VPNのBGPが紹介されるが、**2025-08-01以降の新規BGP構成では利用できない**。
- BGPによる経路学習はオンプレ側が経路を広告していることが前提。
- Cloud Routerがパケットを中継しているという理解は誤り。
- Cloud VPNはCloud NATとは異なる。Cloud NATはVPN暗号化を提供しない。
- VPC Peeringだけではオンプレのルーターと接続できない。

### ACE試験での判断基準

- オンプレ、IPsec、インターネット経由→ Cloud VPN。
- 高可用性、BGP、動的ルート→ HA VPN + Cloud Router。
- 高可用性のSLA→ 2トンネル等の構成条件を確認。
- 新しいサブネットの自動反映→ BGP経路広告/学習。
- 同じVPCの複数リージョンへ動的ルート→ Global dynamic routing。
- 専用接続が必要→ Cloud Interconnect。
- 2 VPC間のプライベート通信→ VPC Peering（または要件によってHA VPNなど）。

## 関連ページ

- [Wikiホーム](../index.md)
- [DNSとA/CNAME](dns-public-internal-a-cname.md)
- [大規模移行とCloud Interconnect](transfer-appliance-vs-interconnect.md)

## 出典・検証

Google Cloud公式ドキュメント、2026-10-24確認。

- [Cloud VPN overview](https://docs.cloud.google.com/network-connectivity/docs/vpn/concepts/overview)
- [Classic VPN BGP deprecation](https://docs.cloud.google.com/network-connectivity/docs/vpn/deprecations/classic-vpn-deprecation)
- [HA VPN topologies](https://docs.cloud.google.com/network-connectivity/docs/vpn/concepts/topologies)
- [Cloud Router overview](https://docs.cloud.google.com/network-connectivity/docs/router/concepts/overview)
- [Choosing networks and routing](https://docs.cloud.google.com/network-connectivity/docs/vpn/concepts/choosing-networks-routing)
- [Set dynamic routing modes](https://docs.cloud.google.com/network-connectivity/docs/router/how-to/create-network-set-modes)
- [VPC Peering](https://docs.cloud.google.com/vpc/docs/vpc-peering)
- [Cloud NAT overview](https://docs.cloud.google.com/nat/docs/overview)
