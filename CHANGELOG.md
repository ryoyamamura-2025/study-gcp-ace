# 更新履歴

## 2026-10-24 — HA VPNとCloud RouterのBGP動的ルーティングを追加

- HA VPN + Cloud Routerによるオンプレミス接続、IPsecとBGPの役割分担、Regional/Global動的ルーティングを整理。
- Classic VPNのBGP新規作成不可（2025-08-01）、HA VPNの99.99% SLAのトンネル要件、VPC Peering/Cloud NATとの違いを公式資料で確認。


## 2026-10-24 — Organization PolicyとIAM・Firewall・Cloud NATを比較

- 外部IP付きVMの新規作成を禁止する `compute.vmExternalIpAccess` 制約を整理。
- ポリシーの継承・下位での上書き、既存VM非遡及、MIG/GKEへの影響を確認。


## 2026-10-24 — サービスアカウントの権限借用とADCを整理

- Service Account UserとToken Creatorの違い、短期認証情報とサービスアカウントキーのリスクを整理。
- gcloud CLI認証とADC、単発フラグとgcloud構成での権限借用を公式資料で検証し、判断軸を追加。


## 2026-10-24 — Pub/SubのFan-out、ACKと再配信を整理

- 1 Topic・複数SubscriptionによるFan-out、同一Subscription・複数Subscriberによる処理分担の違いを追加。
- ACK Deadline、再配信・べき等性、Exactly-onceの適用条件、BigQuery Subscriptionの使い分けをGoogle公式で確認。

## 2026-10-24 — ロードバランサ選定（L7/L4とTLS）を追加

- Application LBとProxy/Passthrough Network LB、外部/内部、グローバル/リージョンの選定軸を公式資料で確認。
- URLマップによるパス振り分け、MIGのヘルスチェック、TLS終端、Cloud DNS/Cloud NATとの違いを整理。
- Global external passthrough Network LBも存在する現行仕様を明記し、すべてのNLBがリージョン専用という誤解を防止。

## 2026-10-23 — Transfer Applianceによる2PBオフライン移行を追加

- 公式模試の2PB/100Mbps/最大1Gbpsという条件に基づくネットワーク転送の理論下限時間を整理。
- Transfer Applianceのオフライン転送、装置1台の容量上限、Cloud Interconnect/PSCとの目的の違いを比較。


## 2026-10-23 — Cloud DNSとCompute Engine内部DNSを比較

- 公式模試でのDNS公開の論点から、Cloud DNS公開ゾーン、Compute Engine内部DNS、A/CNAME、外部IP、エイリアスIP、VPCルートの選定を整理。
- 複数アプリのDNS名とHTTP転送の違い、静的IPとHAの違いを補足。


## 2026-10-23 — 公式ACE模擬試験の論点を整理

- 共有された20問の公式模試から、最小権限、運用負荷、費用対効果、サービス選定の判断軸を独自に抽出。設問原文は保存しない。
- 模試の正解と現行仕様を区別。BigQueryのdataViewerだけではSQLジョブを起動できない点、イベントトリガーと構成変更監査の範囲の違いなどを公式情報で確認。
- 公式模試の判断軸の記事、出典要約、BigQuery既存ページ、索引とREADMEを更新。

## 2026-10-20 — Compute Engine SSH公開鍵・OS Login・IAPを追加

- プロジェクト全体SSH公開鍵の意味、ユーザーごとの鍵ペア、複数VMへの適用と例外を公式資料で確認。
- 同じ公開鍵を複数ユーザーで保持するケースと、秘密鍵まで共有するリスクを区別。
- SSH認証と通信経路を分け、OS Login・IAP TCP転送・外部IPなしの要件との使い分けを追加。
- 公開用の学習論点ソース、Wiki索引、READMEを更新。

## 2026-10-20 — GKEのPod/ノードのオートスケーリング比較を追加

- HPA（Podレプリカ数）、VPA（PodのCPU/メモリ要求）、Cluster Autoscaler（Standardノード台数）の比較記事を追加。
- Podを増やすことで並列処理が増える理由、ノードのCPU総量は増えない点、余剰容量がない場合の連携を整理。
- 「新アプリが増えた際にノード容量を自動拡張」の設問判断軸と、resource requests/スケジューリングを基準にする注意点を追加。
- ソースの要点、Wiki索引、関連記事、READMEを更新。

## 2026-10-19 — BigQueryのSQL実行・データ操作ロールを追加

- `bigquery.jobUser`、`bigquery.dataViewer`、`bigquery.dataEditor`、`bigquery.user` の違いを現行公式資料で確認し、比較記事を追加。
- `jobs.create` と `tables.getData` / `tables.updateData` の組み合わせ、プロジェクトとデータセットの付与範囲、ACEでの選定条件を整理。
- Wiki索引と公開用会話ソースの要点を更新。

## 2026-10-19 — Cloud SQLバックアップ/PITRを追加

- Cloud SQLの自動バックアップ、オンデマンドバックアップ、PITRの使い分けを公式資料で検証しWiki記事に追加。
- MySQLのバイナリログ、PostgreSQLのWAL、SQL Serverのトランザクションログ、PITRの新規インスタンスへの復元を整理。
- Enterprise/Enterprise Plus・作成手段による既定値の差、保持期間、標準/拡張バックアップとインスタンス削除時の注意点を追記。
- 索引と公開用会話ソースの要点を更新。

## 2026-10-19 — 初期版

- 空のGitHubリポジトリに、LLM Wikiの3層（sources / wiki / schema）を基本とする運用規則を作成。
- 参照可能なプロジェクト会話の一部分を、著作物を転載せずに出典メモとして登録。
- Google Cloud公式情報で検証した2ページを登録:
  - GKE Standardでの別マシンタイプのノードプール追加。
  - VPCネイティブのNode/Pod/Service IPv4範囲とGKE管理Service CIDRの例外。
- 未対応: 過去チャット全履歴の一括取得、GitHub Actionsによる自動lint、講座の未提供設問の取り込み。
