# GCP Associate Cloud Engineer — Learning Wiki

Google Cloud Associate Cloud Engineer (ACE) 試験向けの、日本語・公式情報ベースのナレッジベースです。

> 最終整備: 2026-10-19。公開リポジトリのため、個人情報・非公開チャット全文・講座の設問や解説の転載は保存しません。

## Wiki を読む

- [Wiki ホーム](wiki/index.md)
- [GKE: ノードプールとPodの配置](wiki/gke/node-pools-and-scheduling.md)
- [GKE: VPCネイティブのIP範囲](wiki/gke/vpc-native-ip-ranges.md)

## 運用の基本

このリポジトリは [LLM Wiki](https://llmwiki.cc/docs) の考え方で運用します。**sources（入力・出典）→ wiki（統合された知識）→ SCHEMA.md（構成・品質ルール）** の3層を分け、**ingest / query / lint** を繰り返します。

- `sources/`: 取り込んだ情報の来歴。今回のチャットは公開用の要約のみで、原文転載はしません。
- `wiki/`: サービス・概念・操作・比較を説明する、相互リンク付きのページ。
- [SCHEMA.md](SCHEMA.md): 記事種別、必須メタデータ、出典・検証規則。
- [AGENTS.md](AGENTS.md): AI/人間が新規情報を反映するときの手順。
- [CHANGELOG.md](CHANGELOG.md): 取り込み・改訂の記録。

## 更新するとき

1. 新しいチャットで浮かび上がった疑問・誤解・条件を抽出する。
2. [Google Cloud公式ドキュメント](https://docs.cloud.google.com/) に照らして確認する。
3. 既存記事を先に検索して更新する。重複した新規記事を作らない。
4. 文末の参照元・確認日・関連ページと索引を更新する。
5. リンク切れ、矛盾、旧仕様、公開可能な内容かどうかを点検する。

**収録範囲**: 初期版は、この作業セッションで参照可能だったプロジェクト会話の断片に限定。全履歴を取得済みという意味ではありません。詳細は [初期ソース](sources/chat-context-2026-10-19.md)。

## 公式試験情報

- [Associate Cloud Engineer 認定](https://cloud.google.com/learn/certification/cloud-engineer)
