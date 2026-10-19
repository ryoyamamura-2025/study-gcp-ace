# 更新履歴

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
