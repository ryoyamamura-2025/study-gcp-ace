---
id: cloud-sql-backups-and-pitr
title: Cloud SQLの自動・オンデマンドバックアップとPITR
type: comparison
topics: [Cloud SQL, backup, automated-backup, on-demand-backup, PITR, MySQL, binary-log, PostgreSQL, WAL]
status: verified
verified_at: 2026-10-19
source_ids: [SRC-CHAT-20261019-01]
---

# Cloud SQL: バックアップとポイントインタイムリカバリ（PITR）

## 要点

- **自動バックアップ**: 設定したスケジュールで取得。日常的・継続的なデータ保護に使う。
- **オンデマンドバックアップ**: 必要な時点で明示的に取得。スキーマ変更や大規模更新の直前などに使う。
- **PITR（Point-in-Time Recovery）**: 保存済みのバックアップとトランザクションログを使い、復元可能な期間内の**指定時点**に復元する機能。「任意の時刻に戻したい」ときの判断軸。
- **MySQLのPITRにはバイナリログ（Binary Log）を使用**する。バイナリログはバックアップファイルの保存形式ではなく、データ変更などのイベントを記録するログ。

## 仕組み・判断軸

### 取得方法の比較

| 比較軸 | 自動バックアップ | オンデマンドバックアップ |
| --- | --- | --- |
| 取得タイミング | スケジュールによる自動実行 | 管理者が必要時に開始 |
| 主な用途 | 日常の保護、定期復元ポイントの確保 | 作業直前の保護、特定時点のバックアップ |
| 標準バックアップの保持 | 設定した保持期間（Cloud SQL for MySQLでは1〜365日） | 原則、削除されるまで保持。ただしインスタンス削除時は保持設定等に注意 |
| 拡張バックアップの保持 | バックアッププランに従う（最大10年） | カスタム保持期間を設定可能（最大10年、バックアップボールト／プランの制約あり） |

**注**: 「自動かオンデマンドか」は**取得契機**、「標準（Standard）か拡張（Enhanced）か」は**バックアップ管理方式**。別の分類なので混同しない。

### PITRのイメージ

例: 03:00にバックアップ、15:30に誤削除。PITR用ログが利用できるなら、15:29の状態への復元を指定できる。単なるバックアップ復元はバックアップ時点を基準とするが、PITRはログでその後の変更を追跡する。

PITRは復元可能な最古・最新時刻、およびトランザクションログの保持期間に制限される。**PITRを実行すると新しいCloud SQLインスタンスが作成される**。稼働中の既存インスタンスを直接その場で巻き戻すわけではない。

### DBエンジンとログ

| データベース | PITRで使うログ |
| --- | --- |
| Cloud SQL for MySQL | **バイナリログ（Binary Log）** |
| Cloud SQL for PostgreSQL | **WAL（Write-Ahead Log）** |
| Cloud SQL for SQL Server | **トランザクションログ** |

### MySQLでPITRを利用するための設定

1. **自動バックアップが有効**であることを確認する。
2. **PITR（バイナリログ）を有効化**する。MySQLではこの機能が `binaryLogEnabled` と対応する。
3. 必要な復元範囲に合わせて**トランザクションログの保持日数**を設定・確認する。

既存のCloud SQLインスタンスでPITRを有効化すると**再起動**が発生する。

**新規作成時の既定値（MySQLの現行仕様）**:
- **Enterprise**: Google Cloudコンソール作成ではPITRが既定で有効。gcloud CLI / Terraform / Cloud SQL Admin APIからの作成では既定で無効なので、必要なら明示的に設定する。
- **Enterprise Plus**: 作成方法にかかわらずPITRが既定で有効。
- **自動バックアップ**: コンソール作成では既定で有効。CLI / Terraform / APIでは有効化が必要な場合がある。

ログ保持期間の設定可能範囲は、Enterpriseが**1〜7日**、Enterprise Plusが**1〜35日**（MySQL、確認日現在）。サービスの版、エンジン、作成方法、設定は設問で確認する。

## 間違えやすい点

- **自動バックアップを有効にするだけではPITRの要件を満たしたと断定しない**。対象エンジンに必要なログ記録とPITR設定も確認する。
- 「MySQLのバックアップをバイナリ形式で保存する」という解釈は誤り。**バイナリログは操作変更の履歴**である。
- **PITRの復元先は新しいインスタンス**。既存インスタンスへ上書きする通常のバックアップ復元とは混同しない。
- 「オンデマンドバックアップは永遠に消えない」は誤り。標準バックアップでは明示削除やインスタンス削除の影響を受ける。インスタンス削除後に残すには保持バックアップ／最終バックアップの設定を確認する。拡張バックアップは保持ポリシーに従う。
- **PITRが既定で有効か**は、少なくともEnterprise/Enterprise Plusと作成方法で異なる。古い講座の一律の説明を鵜呑みにしない。

### ACE試験の見分け方

- 「毎日自動で保護」→ **自動バックアップ**
- 「変更作業の直前に保護」→ **オンデマンドバックアップ**
- 「誤操作直前の時刻まで戻す」→ **PITR**
- 「Cloud SQL for MySQLでPITRを有効にしたい」→ **自動バックアップとバイナリログ／PITR設定を確認**
- 「復元したら既存インスタンスが巻き戻るか」→ **PITRでは新規インスタンス**

## 関連ページ

- [Wikiホーム](../index.md)

## 出典・検証

Google Cloud公式資料を**2026-10-19**に確認:

- [Cloud SQL for MySQL: Choose your backup option](https://docs.cloud.google.com/sql/docs/mysql/backup-recovery/backup-options)
- [Cloud SQL for MySQL: Configure point-in-time recovery (PITR)](https://docs.cloud.google.com/sql/docs/mysql/backup-recovery/configure-pitr)
- [Cloud SQL for MySQL: Perform point-in-time recovery (PITR)](https://docs.cloud.google.com/sql/docs/mysql/backup-recovery/pitr)
- [Cloud SQL for PostgreSQL: Configure PITR](https://docs.cloud.google.com/sql/docs/postgres/backup-recovery/configure-pitr)
- [Cloud SQL for SQL Server: Configure PITR](https://docs.cloud.google.com/sql/docs/sqlserver/backup-recovery/configure-pitr)
- [Cloud SQL for MySQL: Manage standard backups](https://docs.cloud.google.com/sql/docs/mysql/backup-recovery/manage-standard-backups)

会話から抽出した論点: `SRC-CHAT-20261019-01`。講座問題や会話の原文は未収録。
