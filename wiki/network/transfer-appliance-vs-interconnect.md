---
id: gcp-transfer-appliance-vs-interconnect
title: 大容量データ移行：Transfer Applianceとネットワーク転送の選定
type: comparison
topics: [ACE, Transfer Appliance, Cloud Storage, offline transfer, online transfer, PB, bandwidth, Interconnect, Storage Transfer Service, Private Service Connect]
status: verified
verified_at: 2026-10-23
source_ids: [SRC-OFFICIAL-SAMPLE-20261023-01]
---

# Transfer Applianceとネットワーク転送の使い分け

## 要点

オンプレミスからGoogle Cloudに**ペタバイト級**のデータを一括移行し、利用できるWAN回線の帯域幅が限られている場合は、**Transfer Applianceを使ったオフライン物理輸送**を検討する。既存回線の帯域幅とデータ量を比較し、移行期間、輸送、コスト、地域のサービス可用性を判断する。

ACE公式模試では「2 PBの初回移行、100 Mbps、第三者経由でも最大1,000 Mbps、迅速かつ費用対効果を重視」でTransfer Applianceが正解。

## 仕組み・判断軸

1. Googleから大容量物理ストレージ装置（Transfer Appliance）を受け取る。
2. オンプレミス側で装置へデータをコピーする。オフラインモードではこの段階のクラウドWAN転送は不要。
3. 暗号化データを収めた装置をGoogle側へ発送する。
4. Googleが装置からCloud Storageバケットにデータを取り込む。検証後、装置のデータは消去される。

**回線だけを使う理論上の転送期間：** 2 PB（10進法で2×10^15バイト）を、100 Mbpsで常時転送すれば約1,852日（約5.1年）、1 Gbpsで常時転送しても約185日。プロトコルオーバーヘッドや中断を無視した下限の試算であり、実際はさらに長くなる可能性がある。

| 選択肢 | 向く状況 | 注意 |
| --- | --- | --- |
| Transfer Appliance オフライン | 膨大な初期データ移行、狭い回線、一度にまとめて移す | 発送・装置上限・コピー速度・対応地域・費用を見積もる |
| Storage Transfer Service等のネットワーク転送 | WANの帯域が十分、継続同期、データが相対的に小さい | ネットワークのボトルネックを受ける |
| Partner Interconnect | 接続パートナー経由でGoogle Cloudと長期に接続 | 回線契約・設備・ルーティング・帯域確保が必要 |
| Dedicated Interconnect | Googleネットワークに専用の物理回線で接続可能 | コロケーション等が必要。高帯域だが設定・契約の負担 |
| Private Service Connect | VPC等からGoogle APIや公開サービスに非公開接続 | データコピー・物理輸送自体をするサービスではない |

### 物理装置の容量

2026年10月の仕様文書では、製品により最大40 TB、100 TB、300 TBなどの容量がある。**2 PBは単一の300 TB装置には入らない**。300 TB級なら理論上7台分以上の容量が必要で、複数装置や複数回転送を使う設計になる。実際の時間はコピー環境や物流にも依存する。

## 間違えやすい点

- **2 PBを数値として見る**。2 PBは約2,000 TB。高帯域の1 Gbpsですら200日近い理論時間。
- **100台に分割すれば速いとは限らない**。同じWAN回線の総帯域を共有する限り総転送速度のボトルネックは変わらない。
- **Interconnectが常に不正解ではない**。大規模で継続的なデータ同期や十分な実効帯域が必要なら有力。試験文の「短期一括移行・帯域制約・費用対効果」が鍵。
- **Transfer Applianceにオンラインモードも存在する**。公式模試の正答は物理発送するオフラインモードの利用。
- **Private Service ConnectとInterconnectは目的が異なる**。前者はサービスへの非公開アクセスの仕組み。後者はネットワーク間の接続。

## 関連ページ

- [Wikiホーム](../index.md)
- [公式模試から学ぶ選定判断軸](../exam/official-sample-decision-patterns.md)

## 出典・検証

Google Cloud公式情報を2026-10-23に確認。

- [Transfer Applianceの概要](https://docs.cloud.google.com/transfer-appliance/docs/4.0/overview)
- [Transfer Applianceの仕様](https://docs.cloud.google.com/transfer-appliance/docs/4.0/specifications)
- [Transfer Applianceの利用手順](https://docs.cloud.google.com/transfer-appliance/docs/4.0/procedure-guide)
- [オンライン・オフラインモード](https://docs.cloud.google.com/transfer-appliance/docs/4.0/online-transfer)
- [Partner Interconnect概要](https://docs.cloud.google.com/network-connectivity/docs/interconnect/concepts/partner-overview)
- [Dedicated Interconnect概要](https://docs.cloud.google.com/network-connectivity/docs/interconnect/concepts/dedicated-overview)
- [Private Service Connect](https://docs.cloud.google.com/vpc/docs/private-service-connect)

出題の論点: SRC-OFFICIAL-SAMPLE-20261023-01。模試原文・選択肢は転載しない。
