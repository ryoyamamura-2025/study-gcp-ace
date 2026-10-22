---
id: bigquery-iam-query-roles
title: BigQueryのクエリ実行権限とデータ操作権限
type: comparison
topics: [BigQuery, IAM, jobUser, user, dataViewer, dataEditor, jobs.create, tables.getData, least-privilege]
status: verified
verified_at: 2026-10-23
source_ids: [SRC-CHAT-20261019-01, SRC-OFFICIAL-SAMPLE-20261023-01]
---

# BigQuery: SQL実行とデータアクセスのIAMロール

## 要点

**クエリジョブを開始する権限**と**対象データを読み書きする権限**は別。たとえば既存の売上テーブルをSQLで `SELECT` するには、通常、実行プロジェクト側の `roles/bigquery.jobUser` と、参照先テーブル/データセット側の `roles/bigquery.dataViewer` を組み合わせる。

| IAMロール | 主な責務 | 代表的な権限 | SQLジョブ開始権限 `bigquery.jobs.create` |
| --- | --- | --- | --- |
| `roles/bigquery.jobUser` | ジョブ実行 | `bigquery.jobs.create` | あり |
| `roles/bigquery.dataViewer` | データ閲覧 | `bigquery.tables.getData` | なし |
| `roles/bigquery.dataEditor` | データの読み書き、テーブル作成・更新等 | `bigquery.tables.getData`, `bigquery.tables.updateData`, `bigquery.tables.create` | なし |
| `roles/bigquery.user` | ジョブ実行 + データセット作成など | `bigquery.jobs.create`, `bigquery.datasets.create` | あり |

## 仕組み・判断軸

### 2つの独立した権限

1. **SQLをジョブとして実行する**: `bigquery.jobs.create` が必要。ジョブを起動する**プロジェクト**に対して `jobUser`（または `user` 等）を付与する。
2. **既存テーブルを読む/変更する**: 対象データに対する別の権限が必要。参照には `bigquery.tables.getData`、更新には `bigquery.tables.updateData` などを用いる。権限は必要な**データセットやテーブル**へ最小範囲で付与する。

SQL実行プロジェクトと、参照先データのあるプロジェクトは**異なっていてもよい**。前者にジョブ作成権限、後者の参照先データにアクセス権限が必要。

### 組み合わせ例（既存テーブル、追加の権限なし）

| 付与したロール | `SELECT` クエリ | `UPDATE` クエリ |
| --- | --- | --- |
| `jobUser` のみ | 不可（参照権限がない） | 不可 |
| `dataViewer` のみ | 不可（SQLジョブ作成権限がない） | 不可 |
| `dataEditor` のみ | 不可（SQLジョブ作成権限がない） | 不可 |
| `jobUser` + `dataViewer` | 可 | 不可 |
| `jobUser` + `dataEditor` | 可 | 可 |

**注**: ここでの可否は「BigQuery SQLクエリジョブ」としての実行を指す。`dataViewer` はジョブ実行権限がなくても、適切なAPIやプレビュー機能等を通じてデータを閲覧できる場合がある。`dataEditor` もSQLジョブとは別の方法でデータ操作できる場合がある。行レベルセキュリティ等の別制約、クエリ種類固有の権限がある場合は追加確認が必要。

### `bigquery.user` と `bigquery.jobUser` の違い

- `bigquery.user` を**プロジェクトレベル**で付与すると、SQLジョブ実行に加えて、**新しいデータセットの作成**も可能。作成したデータセットには作成者に `bigquery.dataOwner` が付与される。
- `bigquery.jobUser` は主にジョブ実行に限定されるため、**既存データへのSQL実行だけ**を必要とする場合の最小権限設計に向く。
- **`bigquery.user` だけでは、他者が作成した既存データセット内のテーブルを何でも読み取れるわけではない**。別途、対象テーブルの閲覧権限が必要。
- `bigquery.dataEditor` は**ジョブ実行の代替ではない**。`INSERT`/`UPDATE`/`DELETE` などのSQLには、ジョブ実行権限と対象データの変更権限の双方が必要。

### ACE試験の見分け方

- 「SQLジョブを実行できるように」→ **`jobUser`**（既存データを使うならデータ権限も検討）。
- 「既存テーブルをSQLで読み取る」→ **`jobUser` + `dataViewer`**。
- 「SQLでテーブルのデータを変更する」→ **`jobUser` + `dataEditor`**。
- 「新規データセットも作成する」→ **`bigquery.user`** を検討（ただしデータセットを作成させるのが妥当かも評価）。
- 「グループにSQLを実行させたい」→ グループへ必要ロールを付与できる。**実行権限とデータ権限を分けて判断**する。

## 間違えやすい点

- 「`dataViewer` があれば `SELECT` を実行できる」→ **ジョブ作成権限が別に必要**。
- 「`dataEditor` はSQL実行もできる」→ **データ編集権限とジョブ実行権限は別**。
- 「`jobUser` さえあればすべてのテーブルを読める」→ **データに対する閲覧権限がない**。
- 「`bigquery.user` = `bigquery.dataViewer`」→ **別物**。`user` はプロジェクト内でジョブ実行・データセット作成が中心。
- `jobUser` を特定テーブルだけに付与できると考えない。**プロジェクト、フォルダ、組織**レベルで付与するロール。
- IAMロールは付与する**階層**で影響範囲が変わる。必要なデータセット/テーブルに `dataViewer` や `dataEditor` を絞るのが基本。

### 公式模試の問題文と必要な実装権限の違い

ACEの公式模試には、監査者に閲覧・レポート実行を許可する設問で「グループに dataViewer を付与」が最善選択肢となる例がある。出題意図は**IAMをグループに付与し、基本ロールより粒度が適切な事前定義ロールを選ぶ**こと。ただし、SQLクエリジョブを実行する実装では別途実行プロジェクトに jobUser（jobs.create）等が必要。模試の正答を「dataViewer単独でSQL実行可」と覚えない。

## 関連ページ

- [Wikiホーム](../index.md)
- [公式模試の判断軸と実運用上の注意点](../exam/official-sample-decision-patterns.md)

## 出典・検証

Google Cloud公式資料を **2026-10-19、2026-10-23** に確認:

- [BigQuery IAM roles and permissions](https://docs.cloud.google.com/bigquery/docs/access-control)
- [Run a query — Required roles and permissions](https://docs.cloud.google.com/bigquery/docs/running-queries)
- [Write query results — Required permissions](https://docs.cloud.google.com/bigquery/docs/writing-results)
- [Create datasets — Required permissions](https://docs.cloud.google.com/bigquery/docs/datasets)

学習会話から抽出した論点: `SRC-CHAT-20261019-01`。会話原文は未収録。

追加確認（公式模試からの論点）: SRC-OFFICIAL-SAMPLE-20261023-01。設問原文は転載していない。
