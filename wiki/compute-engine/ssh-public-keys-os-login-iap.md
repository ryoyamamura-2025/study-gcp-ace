---
id: compute-engine-ssh-public-keys-oslogin-iap
title: Compute EngineのSSH公開鍵・プロジェクト全体鍵・OS Login・IAP
type: comparison
topics: [Compute Engine, SSH, public key, private key, project metadata, instance metadata, OS Login, IAP, IAM, external IP]
status: verified
verified_at: 2026-10-20
source_ids: [SRC-CHAT-20261020-SSH-01]
---

# Compute Engine: SSH公開鍵の共有とVMへの接続

## 要点

- **公開鍵は複数のVMに同じものを登録できる**。公開鍵は検証側に配置する情報であり、SSH認証に必要な秘密鍵とは役割が異なる。
- **プロジェクト全体のSSH鍵（project-wide SSH keys）**とは、同じ鍵ペアを複数人で共有する意味ではない。プロジェクトメタデータに複数ユーザーそれぞれの公開鍵を登録し、許可対象の複数VMに適用できる仕組み。
- Aさん・Bさんが**同じ公開鍵を使う**ことは技術的にはあり得るが、同じSSH鍵ペアを共有している場合、対応する**秘密鍵まで共有すると利用者別の識別・取り消しが難しい**。原則として利用者ごとに別の鍵ペアを使う。
- **公開鍵だけをコピーしてもSSH認証はできない**。対応する秘密鍵で認証を完了できる必要がある。
- **SSH鍵は認証手段であってネットワーク経路ではない**。外部IPのないVMへインターネット側から接続する典型的な手段はIAP TCP転送（権限とファイアウォール等の前提が必要）。

## 仕組み・判断軸

### 1. 公開鍵と秘密鍵の役割

| 項目 | 置く場所 | 役割 |
| --- | --- | --- |
| 公開鍵 | VMが参照するメタデータ、またはOS Loginプロフィール等 | 対応する秘密鍵を利用した認証を検証するために使う |
| 秘密鍵 | 通常は各ユーザーの端末など | 秘密に保持し、SSH接続時に鍵ペアの所有を証明するために使う |

例: Aさんが鍵ペアA、Bさんが鍵ペアBを作成する。プロジェクトメタデータにA・B両方の公開鍵を登録すれば、プロジェクト全体鍵を受け付けるVMに**それぞれ自分の秘密鍵**で認証できる。ここで「複数のVMが各公開鍵を受け入れる」ことと「A・Bが秘密鍵を共有する」ことは全く別。

逆に、BさんがAさんの**公開鍵のみ**をコピーしても、Aさんの秘密鍵を持たないためAさんの鍵ペアによる認証はできない。A・Bが同じ鍵ペア（秘密鍵も含む）を使用する構成自体は可能だが、監査・鍵失効・漏洩時の影響から避ける。

### 2. プロジェクト全体鍵とインスタンス固有鍵

| 登録先 | SSH公開鍵の適用先 | 例外・注意 |
| --- | --- | --- |
| プロジェクトメタデータ | 原則としてプロジェクト内のVM | 対象VMがプロジェクト全体鍵を拒否している場合は使えない |
| インスタンスメタデータ | 対象の1台のVM | OS Loginが有効なVMではメタデータ管理鍵を使用しない |

- インスタンス側で `block-project-ssh-keys=TRUE` を設定すると、プロジェクトメタデータに登録されたSSH公開鍵を拒否できる。ただしインスタンス側で許可された鍵まで拒否する意味ではない。
- **OS Loginが有効なVMでは、プロジェクト・インスタンスメタデータのSSH鍵をゲストエージェントが無視する**。OS LoginではGoogleアカウントに紐づく認証とIAM権限（`roles/compute.osLogin`、`roles/compute.osAdminLogin` など）を利用する。

### 3. OS Loginとメタデータ管理の違い

- **メタデータ管理のSSH鍵**: メタデータへの公開鍵登録で利用可能な鍵を管理する。鍵が残ればユーザーのクラウドIAM側のアクセス権変更だけでは鍵によるOSアクセスが自動的に失効しない場合がある。定期的な鍵の削除・管理が必要。
- **OS Login**: Linux VMへのSSHアクセスをGoogleアカウント・IAMによって管理する。ユーザーの認証試行時にIAMアクセスを評価し、アクセス取り消しを一元管理しやすい。
- 試験で「IAMを使ってOSへのログイン可否や管理者権限を制御する」ならOS Loginを優先して判断する。

### 4. 外部IPのないVMへSSH接続するならIAP TCP転送

- IAP TCP転送は、外部IPを持たないVMへのSSH接続経路を提供できる。SSH認証そのものをIAPが公開鍵の代わりにするわけではない。
- 利用にはIAPへの適切なIAM権限（通常 `roles/iap.tunnelResourceAccessor`）と、IAP送信元IPv4範囲 `35.235.240.0/20` からTCP/22を許可するファイアウォールルール等が必要。
- **「外部IPなしでVMへSSHする」≠「VMからインターネットへ発信する」**。後者のインターネット送信は別のネットワーク設計（例: Cloud NAT）の論点で、SSH鍵の配置では満たせない。

## 間違えやすい点

- **誤**: 「プロジェクト全体のSSH鍵 = 全員が1組の公開鍵・秘密鍵を共有する」→ **正**: 複数ユーザーの公開鍵をプロジェクトメタデータに登録でき、対象VM群に適用される。
- **誤**: 「公開鍵が同じなら、誰でもその公開鍵を使ってログインできる」→ **正**: 対応する秘密鍵を使った認証が必要。
- **誤**: 「プロジェクトメタデータの鍵は絶対に全VMで使える」→ **正**: `block-project-ssh-keys` や OS Login 有効化などの例外がある。
- **誤**: 「SSH鍵を登録すれば、外部IPのないVMへネットワーク接続できる」→ **正**: SSH認証と接続経路は別問題。IAP TCP転送などを検討。
- **誤**: 「IAPの権限だけあればVMへログインできる」→ **正**: IAP経路の権限に加えてOSへのログイン権限・認証が必要。

## ACE試験の選定キーワード

| 設問条件 | 判断する機能 |
| --- | --- |
| 複数のVMへ同じ公開鍵を適用する | プロジェクトメタデータのSSH公開鍵 |
| 特定VMだけプロジェクト全体の鍵を拒否 | `block-project-ssh-keys` |
| IAMでLinux VMのログインやsudo権限を制御 | OS Login |
| 外部IPなしでVMにSSH接続 | IAP TCP転送（IAM・ファイアウォールも必要） |
| VMからインターネットへのアウトバウンド通信 | Cloud NATなどの送信経路（SSH鍵ではない） |

## 関連ページ

- [Wikiホーム](../index.md)
- [GKEのVPCネイティブIP範囲](../gke/vpc-native-ip-ranges.md)

## 出典・検証

Google Cloud公式文書を **2026-10-20** に確認:

- [Add SSH keys to VMs](https://docs.cloud.google.com/compute/docs/connect/add-ssh-keys)
- [Restrict SSH keys from VMs](https://docs.cloud.google.com/compute/docs/connect/restrict-ssh-keys)
- [About OS Login](https://docs.cloud.google.com/compute/docs/oslogin)
- [Best practices for controlling SSH login access](https://docs.cloud.google.com/compute/docs/connect/ssh-best-practices/login-access)
- [Use IAP for TCP forwarding](https://docs.cloud.google.com/iap/docs/using-tcp-forwarding)

会話由来の学習論点: `SRC-CHAT-20261020-SSH-01`（「公開鍵を複数VMで共有」「A・Bで同じ公開鍵のケース」「プロジェクト全体鍵の意味」を抽象化。設問や講座の原文は転載しない）。
