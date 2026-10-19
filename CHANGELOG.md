# 更新履歴

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
