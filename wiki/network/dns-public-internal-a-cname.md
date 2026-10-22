---
id: dns-public-internal-a-cname
title: DNSの公開ゾーン・内部DNS・A/CNAMEと外部IP
type: comparison
topics: [Cloud DNS, internal DNS, public zone, private zone, A, AAAA, CNAME, NS, alias IP, external IP]
status: verified
verified_at: 2026-10-23
source_ids: [SRC-OFFICIAL-SAMPLE-20261023-01]
---

# ACE: DNS名でCompute Engineのアプリを公開する

## 要点

インターネット上でサービスを**名前で公開**するなら、権威DNSの一般公開ゾーンにAレコード（ドメイン名→外部IPv4）等を作る。Google Cloudで管理する場合は**Cloud DNSの公開ゾーン**が候補。VMの**Compute Engine内部DNS**はVPC内での内部IPv4の名前解決であり、一般公開とは異なる。

## 仕組み・判断軸

| 目的 | 選定 |
| --- | --- |
| 公開サイトの名前を外部IPv4に対応づけたい | Cloud DNSの**公開ゾーン + Aレコード**。ドメインの委任先NS設定、VM/ロードバランサの外部IP、ファイアウォール等も確認 |
| 外部IPv6へ名前を対応づけたい | AAAAレコード |
| 別のドメイン名への別名を作りたい | CNAME（IPそのものを指定しない。ゾーンapexでは制約あり） |
| VPC内部だけで独自名を使いたい | Cloud DNSの**限定公開ゾーン** |
| VM同士でVM名によって内部通信したい | Compute Engineの**内部DNS**（VMのprimary nic0 internal IPv4に対して自動作成） |
| VMのIPを恒久的に変えたくない | 静的**外部**IPを予約し、公開ゾーンのAレコードを向ける。固定IPそれ自体はHAを提供しない |

### 複数アプリケーションの場合

同一VM上の複数のアプリ名（例: app1.example.com、app2.example.com）を同じ外部IPに解決させることはできる。ただし**DNSはURLパス、ポート、HTTP Hostによる宛先アプリの振り分けを行わない**。同じポートでアプリごとにルーティングする場合は、リバースプロキシやロードバランサなどを別途構成する。

## 間違えやすい点

- **Compute Engine内部DNS**は自動生成されるが、インターネット公開に使う公開DNSではない。
- **エイリアスIP**はVMに追加の内部IPを割り当てられるが、追加IPにDNS名が自動付与されるわけではない。外部向けのDNS公開手段ではない。
- **VPCルート**はIPパケットの転送先制御であり、ドメイン名を公開する機能ではない。
- **Cloud DNS**と**Cloud Domains**は異なる。前者はDNSゾーンとレコード管理、後者はドメイン登録の管理に用いる。
- **DNSレコードを作るだけでサイトにアクセスできるわけではない**。インターネットから到達可能な外部IP（または公開ロードバランサ）、ファイアウォール、サーバー設定、ドメインの委任が必要。
- サーバーを交換してもIPを変えたくない場合は静的外部IPの予約が役立つが、**それだけで冗長化や無停止切り替えにはならない**。

## 関連ページ

- [Wikiホーム](../index.md)
- [GKEのVPCネイティブIP範囲](../gke/vpc-native-ip-ranges.md)
- [公式模試の選定判断軸](../exam/official-sample-decision-patterns.md)

## 出典・検証

2026-10-23にGoogle Cloud公式ドキュメントを確認。

- [Cloud DNS概要](https://docs.cloud.google.com/dns/docs/overview)
- [DNSゾーン・公開と限定公開](https://docs.cloud.google.com/dns/docs/dns-overview)
- [DNSレコード型](https://docs.cloud.google.com/dns/docs/records-overview)
- [Cloud DNSとCompute Engineを用いたドメイン設定](https://docs.cloud.google.com/dns/docs/tutorials/create-domain-tutorial)
- [Compute Engineの内部DNS](https://docs.cloud.google.com/compute/docs/internal-dns)
- [エイリアスIP](https://docs.cloud.google.com/vpc/docs/alias-ip)
- [静的外部IP](https://docs.cloud.google.com/compute/docs/ip-addresses/configure-static-external-ip-address)

公式模試の学習上の論点: SRC-OFFICIAL-SAMPLE-20261023-01。問題文・選択肢の原文は掲載しない。
