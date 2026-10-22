---
id: SRC-OFFICIAL-SAMPLE-20261023-01
title: 公式ACE模擬試験の論点（会話提供分の要約）
type: source
topics: [ACE, official-sample, IAM, Cloud Run, GKE, BigQuery, network, billing]
status: verified
verified_at: 2026-10-23
source_ids: []
---

# 公式ACE模擬試験の学習論点（2026-10-23）

## 取得範囲
- 2026-10-23の学習会話でGoogle Cloud ACE公式模擬試験の20問とフィードバックが共有された。
- 設問や選択肢の原文は転載せず、学習上の判断軸を要約する。
- 公式側は模擬試験が本試験の出題範囲・難易度を代表しないと注意している。

## 抽出したテーマ
- 最小権限とグループ単位のIAM、基本/事前定義/カスタムロール。
- Cloud Storage/Firestoreのイベント、Cloud Storageライフサイクル。
- Pub/Sub、Bigtable、BigQuery、GKE、Cloud Run、GKE Autopilot。
- Cloud VPN/Interconnect、Transfer Appliance、DNS、静的IP、CIDR拡張。
- Terraform、Billingエクスポートとラベル、クォータと予算アラート。
- Cloud Monitoringとカスタムメトリクス、監査ログ、INFORMATION_SCHEMA。

## 注意
模試での最善選択肢と、現実の設定で必要な全条件は区別する。SQL実行にはdataViewerだけでなく通常jobUserが必要。構成変更の監査ログとオブジェクト・ドキュメント変更トリガーは異なる。

## 出典
元資料：2026-10-23の学習会話（本文非公開）。
公開の試験案内：https://cloud.google.com/learn/certification/cloud-engineer
解説：[公式模試に見る選定判断軸](../wiki/exam/official-sample-decision-patterns.md)
