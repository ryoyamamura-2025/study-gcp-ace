---
id: SRC-CHAT-20261019-01
title: 2026-10-19 プロジェクト会話からの公開可能な要点
type: source
captured_at: 2026-10-19
completeness: partial
---

# プロジェクト会話からの要点（抜粋・原文ではない）

このソースは、今回の作業時に参照可能だったプロジェクト会話の**一部**を、公開用に抽象化したものです。ユーザーの過去チャット全文ではありません。講座の設問や回答は転載しません。

## 論点A: GKE 既存稼働中のクラスタで異なるマシンタイプが必要

- 見えている学習論点: 既存ノードは `n1-standard-2`、新しいPodに `n2-highmem-16` が必要、稼働中のPodを維持したい。
- 混同しやすい操作: クラスタ/ノードのバージョンアップと、異なるマシンタイプのノードプール追加。
- 統合先: [ノードプールとPod配置](../wiki/gke/node-pools-and-scheduling.md)

## 論点B: VPCネイティブGKEのIPv4割り当て

- 見えている学習論点: サブネットのプライマリIPとセカンダリIPの違い、Node・Pod・Serviceへの割り当て。
- 追加確認が必要だった例外: Service IPにGKE管理範囲が利用されるクラスタもある。
- 統合先: [VPCネイティブIP範囲](../wiki/gke/vpc-native-ip-ranges.md)

## 論点C: Cloud SQLのバックアップとPITR

- 見えている学習論点: 自動バックアップとオンデマンドバックアップの違い、PITRの有効化要件、MySQLのバイナリログの役割。
- 混同しやすい点: バイナリログとバックアップの保存形式、バックアップ時点への復元と時刻指定のPITR、Enterprise/Enterprise Plusの既定設定、標準/拡張バックアップの保持ルール。
- 統合先: [Cloud SQLのバックアップとPITR](../wiki/cloud-sql/backups-and-pitr.md)

## 収録対象外

- タイトルのみ確認できる別トピックや、全文が提供されていない過去の解説は未収録。
- 本ファイルは元チャットへの機械可読な永続リンクを持たず、非公開の元会話を復元しません。

## Wiki化時に確認した公式資料

- https://docs.cloud.google.com/kubernetes-engine/docs/how-to/node-pools
- https://docs.cloud.google.com/kubernetes-engine/docs/how-to/upgrading-a-cluster
- https://docs.cloud.google.com/kubernetes-engine/docs/concepts/alias-ips
- https://docs.cloud.google.com/sql/docs/mysql/backup-recovery/backup-options
- https://docs.cloud.google.com/sql/docs/mysql/backup-recovery/configure-pitr
- https://docs.cloud.google.com/sql/docs/mysql/backup-recovery/pitr
- https://docs.cloud.google.com/sql/docs/postgres/backup-recovery/configure-pitr
- https://docs.cloud.google.com/sql/docs/sqlserver/backup-recovery/configure-pitr
