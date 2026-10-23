---
id: ace-load-balancer-selection
title: Google Cloudロードバランサの選定とURLマップ・TLS終端
type: comparison
topics: [ACE, load-balancing, Application Load Balancer, Network Load Balancer, URL map, TLS, health-check, global, regional]
status: verified
verified_at: 2026-10-24
source_ids: []
---

# ロードバランサの選定：L7/L4、外部/内部、グローバル/リージョン

## 要点

- **Application Load Balancer** はHTTP/HTTPSのL7ロードバランサ。URLパスやHTTPヘッダー等の条件でバックエンドを選べる。
- **Network Load Balancer** はL4。**プロキシ型**（TCP、対応する外部構成ではTLS/SSLオフロード可能）と**パススルー型**（TCP/UDPなど、クライアントの元のパケットをバックエンドへ転送）がある。通常のHTTP URLパスマッチングは行わない。
- インターネットから公開するなら**外部**、VPCや接続ネットワークから使うなら**内部**。
- 世界中の利用者と複数リージョンのHTTP(S)バックエンドを単一グローバルIPで扱うなら**グローバル外部Application Load Balancer**が有力。

## 仕組み・判断軸

| 判断軸 | 要件 | 選択候補 |
| --- | --- | --- |
| プロトコル | HTTPS、ホスト名・パス別振り分け | Application LB |
| プロトコル | HTTP以外のTCPをプロキシ分散、必要なら外部でTLSオフロード | Proxy Network LB |
| プロトコル | UDP、TCP等の元のパケットを維持 | Passthrough Network LB |
| 利用者 | インターネットからのアクセス | External |
| 利用者 | VPC/接続ネットワーク内だけ | Internal |
| 配置 | 複数リージョンのHTTP(S)バックエンドを1つのグローバル入口で使用 | Global external Application LB |
| 配置 | 一つのリージョン内の処理 | Regionalタイプを検討 |

### Application LBの典型的なリクエスト経路

**グローバル外部IP・転送ルール → ターゲットHTTPSプロキシ（クライアント側TLS終端） → URLマップ → バックエンドサービス → 正常なMIGなど**。

- URLマップはパス・ホストなどに基づいてバックエンドサービス／バックエンドバケットを選ぶ。たとえば `/api/*` はAPI用バックエンド、`/images/*` は画像配信用バックエンド。
- バックエンドサービスに関連付けたヘルスチェックは応答可能なバックエンドを確認する。ヘルスチェックを通ったことはアプリ全機能の正常を保証しない。プローブを受け入れるファイアウォール設定なども必要。
- TLSをロードバランサで終端した後、バックエンドへの通信が自動的にHTTPSになるとは限らない。バックエンド側のプロトコルと証明書設定も区別して確認する。

### 外部・内部とグローバル・リージョン

- **External** はインターネット経由の利用者向け、**Internal** はVPCまたはVPN/Interconnectなどで接続したネットワークの利用者向け。
- **Global external Application LB** はPremium TierのグローバルAnycast IPを利用し、複数リージョンのバックエンドにアクセスできる。
- **Regional external passthrough Network LB** は同じリージョンのバックエンドを対象とする。
- **例外に注意**：Network LBがすべてリージョン専用というわけではない。**Global external passthrough Network LB** や **Global external proxy Network LB** もある。ただしURLパスのL7分岐はApplication LBの役割。

## 間違えやすい点

- 「HTTPSを受けられる」だけでApplication LBを一意に決めない。L4パススルーではバックエンドがTLS終端でき、外部Proxy Network LBの一部ではSSLオフロードできる。ただしHTTPパス振り分け要件ならApplication LB。
- Cloud DNSの名前解決はURLパスに基づくHTTPルーティングではない。
- Cloud NATは主に外向き通信のためのNATであり、内部LBをインターネットから公開する機能ではない。
- グローバルIPを維持することと、異常なVMを検出して健全なバックエンドに転送することは別。後者には適切なヘルスチェックとバックエンド構成が必要。

### ACE試験での決め方

1. HTTP(S)のパス/ヘッダーによる振り分けが必要か？ → Application LB。
2. インターネットからアクセスされるか？ → External。非公開のVPC間などか？ → Internal。
3. バックエンドが複数リージョンか、一つのリージョンか？ → Global/Regionalを選ぶ（各製品の対応範囲も確認）。
4. TLS終端をどこで行うか、ヘルスチェックがどこを検査するかを判断する。

## 関連ページ

- [Wikiホーム](../index.md)
- [Cloud DNSの名前解決とA/CNAME](dns-public-internal-a-cname.md)

## 出典・検証

Google Cloud公式、2026-10-24確認。

- [Choose a load balancer](https://docs.cloud.google.com/load-balancing/docs/choosing-load-balancer)
- [Cloud Load Balancing overview](https://docs.cloud.google.com/load-balancing/docs/load-balancing-overview)
- [External Application Load Balancer overview](https://docs.cloud.google.com/load-balancing/docs/https)
- [Proxy Network Load Balancer overview](https://docs.cloud.google.com/load-balancing/docs/proxy-network-load-balancer)
- [Passthrough Network Load Balancer overview](https://docs.cloud.google.com/load-balancing/docs/passthrough-network-load-balancer)
- [Global external passthrough Network Load Balancer overview](https://docs.cloud.google.com/load-balancing/docs/network/global-networklb-architecture)
- [URL maps overview](https://docs.cloud.google.com/load-balancing/docs/url-map-concepts)
