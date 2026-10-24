---
id: organization-policy-vs-iam-firewall-nat
title: Organization Policy・IAM・VPC Firewall・Cloud NATと外部IP制約
type: comparison
topics: [ACE, Organization Policy, IAM, VPC Firewall, Cloud NAT, external-IP, vmExternalIpAccess, policy-inheritance, hierarchy]
status: verified
verified_at: 2026-10-24
source_ids: []
---

# Organization PolicyとIAM・VPC Firewall・Cloud NATの違い

## 要点

- **Organization Policy**: 組織・フォルダ・プロジェクトのリソースに許される構成を制限する。
- **IAM**: だれがどのリソースにどの操作をできるかを制御する。
- **VPC Firewall**: ネットワーク通信の許可・拒否。VMの外部IPの付与自体は防げない。
- **Cloud NAT**: 主に外向き通信を可能にするアドレス変換。VMの外部IP割り当て禁止や不特定のインバウンド公開とは異なる。
- 外部IPv4アドレスの使用を組織で制限する場合は、**`constraints/compute.vmExternalIpAccess`**が代表的な選択肢。

## 仕組み・判断軸

### VMの外部IP制限

`compute.vmExternalIpAccess` はCompute Engine VMに外部IPを使用させる範囲を制限する旧世代マネージド（legacy managed）のリスト型制約。

- **空の`allowedValues`**で外部IPを使用できるVMをゼロにできる。
- `allowedValues`にVM URIを列挙すると、そのVMだけ許可される。
- `deniedValues`は列挙したVMのみ拒否するため、「将来作るVMは原則すべて外部IPなし」にする条件とは合致しない。
- ポリシー適用前から外部IPを持つVMは、**さかのぼって外部IPを取り除かれない**。
- 既存MIGやGKEクラスタが外部IP付きVMを自動追加・再作成する場合、スケール/自動修復が失敗する可能性がある。

### 操作権限と構成制約の独立性

VM作成権限をIAMから得ていても、外部IP割り当て禁止の実効Org Policyがあれば、外部IP付きVMの新規作成は拒否される。一方、外部IPを付けないVMは他の要件と権限が満たされれば作成できる。

| 何を制御？ | サービス | 例 |
| --- | --- | --- |
| リソースに許す構成 | Organization Policy | 外部IPなし、使用可能なマシンタイプ制限 |
| 人間やSAの操作権限 | IAM | VMの作成権限 |
| パケット・通信の許可拒否 | VPC Firewall | SSH/TCP 22番へのIngress拒否 |
| 外向き通信時のIPアドレス変換 | Cloud NAT | 外部IPなしのVMからインターネットへ接続 |

### 階層と継承の例外

通常は**組織 → フォルダ → プロジェクト**へOrg Policyを継承する。しかし、下位リソースで同じ制約に別のポリシーを設定すると、タイプと階層評価規則により継承ポリシーが上書きされたり結合されたりする。**組織ポリシーが絶対に下位で上書き不能だとは言えない**。

- `roles/orgpolicy.policyAdmin`等、Org Policy変更権限を持つ利用者を制限する。
- `compute.vmExternalIpAccess`の許可リストで運用する場合、子リソースの独自ポリシーで上書きされないか点検する。
- 組織レベルのポリシーを設定しても、既存外部IP付きVMが即時修正されるわけではない。

## 間違えやすい点

- VPC Firewallで外部からの接続を拒否しても、外部IPアドレスそのものの割り当ては禁止できない。
- Cloud NATを設定しても、VMへの外部IP割り当てを強制的に無効化できない。NATは着信公開の手段ではない。
- 管理者からCompute Instance Adminを削除するとVM作成もできなくなるため「作成可・外部IPのみ禁止」を満たさない。
- `compute.vmExternalIpAccess` は新規変更に制約をかけるが、既存VMをさかのぼって変更しない。
- Org Policyにおける「組織レベルに設定＝全ての下位階層で例外なく変更不能」は不正確。権限管理と実効ポリシーの確認が必要。

### ACE試験での判断基準

- 「誰が作成できる？」→ IAM。
- 「組織全体でVM設定を禁止、VM作成自体は許す」→ Organization Policy。
- 「あるポートへの入出力通信を制限」→ Firewall。
- 「外部IPを割り当てないVMがインターネットへ出る」→ Cloud NAT（Google APIだけならPrivate Google Accessなど別方式も検討）。
- 「新規VMの外部IPを禁止」→ `constraints/compute.vmExternalIpAccess`（空の許可リスト）。
- 「既存MIGを維持」→ インスタンス再作成/自動スケールの外部IP利用を事前確認。

## 関連ページ

- [Wikiホーム](../index.md)
- [ロードバランサ選定](../network/load-balancer-selection.md)
- [サービスアカウント権限借用とIAM](service-account-impersonation-and-adc.md)

## 出典・検証

Google Cloud公式ドキュメントを2026-10-24に確認。

- [Organization policies for Compute Engine](https://docs.cloud.google.com/compute/docs/access/organization-policies)
- [Restrict external IP addresses to specific instances](https://docs.cloud.google.com/compute/docs/ip-addresses/configure-static-external-ip-address)
- [Organization Policy overview](https://docs.cloud.google.com/organization-policy/overview)
- [Hierarchy evaluation](https://docs.cloud.google.com/organization-policy/hierarchy-evaluation)
- [Cloud NAT overview](https://docs.cloud.google.com/nat/docs/overview)
