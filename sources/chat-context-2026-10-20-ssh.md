---
id: SRC-CHAT-20261020-SSH-01
title: 2026-10-20 SSH鍵・公開鍵の共有に関する会話の公開可能な要点
type: source
captured_at: 2026-10-20
completeness: partial
---

# SSH公開鍵・プロジェクト全体鍵の学習論点

この文書は当該会話の学習上の疑問だけを抽象化した要点であり、非公開チャットやUdemyの設問原文・解説を転載しない。

- 「プロジェクト全体の公開鍵」は1組の鍵ペアを複数人で共有することなのか、という疑問。
- 同じ公開鍵をAさん・Bさんが保持してよいのか、公開鍵だけの共有と秘密鍵の共有に何の違いがあるか、という疑問。
- SSH鍵は**認証**の話であり、外部IPなしのVMへの**経路**とは分ける必要がある。OS LoginやIAP TCP転送との使い分けがACEに必要。
- 統合先: [Compute EngineのSSH公開鍵・OS Login・IAP](../wiki/compute-engine/ssh-public-keys-os-login-iap.md)

## 参照した公式文書（2026-10-20）

- https://docs.cloud.google.com/compute/docs/connect/add-ssh-keys
- https://docs.cloud.google.com/compute/docs/connect/restrict-ssh-keys
- https://docs.cloud.google.com/compute/docs/oslogin
- https://docs.cloud.google.com/compute/docs/connect/ssh-best-practices/login-access
- https://docs.cloud.google.com/iap/docs/using-tcp-forwarding
