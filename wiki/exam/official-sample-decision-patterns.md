---
id: ace-official-sample-decision-patterns
title: ACE公式模試から学ぶ選定判断軸と注意点
type: comparison
topics: [ACE, decision-making, least-privilege, IAM, Cloud Run, GKE, Pub/Sub, BigQuery, quotas, audit-logs, event-triggers]
status: verified
verified_at: 2026-10-23
source_ids: [SRC-OFFICIAL-SAMPLE-20261023-01]
---

# ACE公式模試から学ぶサービス選定の判断軸

## 要点

公式模試は、単に技術的に実現可能な方法でなく、**推奨手法・最小権限・少ない運用負荷・費用対効果**を満たす選択肢を見極める形式が目立つ。これは今回のサンプルから読み取れる傾向であり、本試験の出題範囲や難易度を保証しない。

## 仕組み・判断軸

| 問題文の条件 | まず検討するもの | 判断理由・対案 |
| --- | --- | --- |
| 人員の増減がある・最小権限 | グループへの事前定義IAMロール | 個人への直接付与・過度に広い基本ロールを避ける。適切な事前定義ロールがなければカスタムロール |
| コンテナ稼働・運用負荷を抑える | Cloud Run | Kubernetesの機能が必要ならGKE。既存KubernetesにはGKE Autopilotも比較 |
| 大量の非同期タスクをワーカーへ | Pub/Sub | 配信基盤とHTTP負荷分散を混同しない |
| 大規模時系列データの低レイテンシ読み書き | Bigtable | 分析SQLのBigQuery、分散トランザクションのSpannerと区別 |
| オブジェクトの経年移動と削除 | Cloud Storage Lifecycle | ストレージクラス変更と削除をルールで自動化 |
| リソースの過剰消費を抑制 | Quota | 通常の予算アラートは通知で、利用の自動停止ではない |
| 大量データに断続的にアドホックSQL | BigQuery | 常時稼働する分析クラスタと比較 |
| Pod入れ替え後も到達先を安定 | Kubernetes Service | PodのIPに依存しない |
| 複数環境を同じ手法で作成 | Terraformの再利用可能なテンプレート/モジュール | 既製ブループリントを含め一貫性と保守性で判断 |

問題文の「最小限」がコスト、設定、運用労力、権限のどれを意味するかを最初に確認する。

## 間違えやすい点

### BigQueryの閲覧権限とSQL実行権限は別

模試では監査者グループへのBigQuery Data Viewerが最善選択肢。ただしSQLクエリジョブを実行するなら通常、実行プロジェクトへのBigQuery Job User（bigquery.jobs.create）も必要。模試の出題意図は「グループ利用・事前定義ロール」。[詳しい比較](../bigquery/iam-query-roles.md)。

### イベントトリガーはすべての「構成変更」を捕捉しない

Firestoreイベントはドキュメントのcreate/update/delete等、Cloud Storageのイベント通知は主としてオブジェクトの作成・削除・メタデータ変更等。バケット設定などの管理操作をすべて網羅するものではない。構成変更の監査ならCloud Audit LogsのAdmin Activityやログルーティングを考える。

### Quotaは支出総額の保証された上限ではない

Quotaはリソース・API使用量に対する制限。通常のCloud Billing予算アラートは通知で、自動的な停止ではない。使用量の抑止と支出の把握を分離する。

### 固定IPと高可用性は別

静的外部IPはDNSを変更せず同じアドレスを使う手段だが、故障したVMを自動的に別VMへフェイルオーバーする機能ではない。実際の可用性要件はロードバランサや冗長化の設計も要確認。

### Terraformのパラメータ化自体は誤りではない

模試ではGoogle提供のブループリント採用が最善選択肢になっているが、環境差分を変数化する再利用可能な構成が一般に誤りという意味ではない。

## 関連ページ

- [Transfer ApplianceとInterconnectの選択](../network/transfer-appliance-vs-interconnect.md)

- [Wikiホーム](../index.md)
- [BigQueryの権限比較](../bigquery/iam-query-roles.md)
- [GKEのスケーリング](../gke/autoscaling.md)

## 出典・検証

2026-10-23にGoogle Cloud公式を確認。

- [ACE認定](https://cloud.google.com/learn/certification/cloud-engineer)
- [IAMセキュリティ推奨](https://docs.cloud.google.com/iam/docs/using-iam-securely)
- [IAMロール選定](https://docs.cloud.google.com/iam/docs/choose-role-type)
- [BigQueryクエリ実行権限](https://docs.cloud.google.com/bigquery/docs/running-queries)
- [Cloud Storage Lifecycle](https://docs.cloud.google.com/storage/docs/lifecycle)
- [Cloud Storageのオブジェクト通知](https://docs.cloud.google.com/storage/docs/pubsub-notifications)
- [Firestoreイベント](https://docs.cloud.google.com/run/docs/triggering/firestore-triggers)
- [Cloud Storage監査ログ](https://docs.cloud.google.com/storage/docs/audit-logging)
- [Cloud Quotas](https://docs.cloud.google.com/docs/quotas/quotas)
- [Cloud Billing予算・アラート](https://docs.cloud.google.com/billing/docs/how-to/budgets)
- [Cloud Runスケーリング](https://docs.cloud.google.com/run/docs/about-instance-autoscaling)

学習会話要約：SRC-OFFICIAL-SAMPLE-20261023-01。
