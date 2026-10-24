---
id: iam-service-account-impersonation-and-adc
title: サービスアカウント権限借用とgcloud CLI・ADC認証
type: comparison
topics: [ACE, IAM, service-account, impersonation, Service Account User, Service Account Token Creator, gcloud, ADC, keyless-authentication, named-configuration]
status: verified
verified_at: 2026-10-24
source_ids: []
---

# サービスアカウント権限借用、IAMロール、gcloud CLIとADC

## 要点

- **サービスアカウント権限借用（impersonation）**は、認証済みのユーザーなどが対象サービスアカウントの**短期認証情報**を取得し、そのサービスアカウントとしてGoogle Cloud APIを実行する方式。サービスアカウントの永続的JSON鍵を取得・保存せずに済む。
- **Service Account User（`roles/iam.serviceAccountUser`）**は `iam.serviceAccounts.actAs` を含み、VMやCloud Run等のリソースへSAを関連付けるときの代表的なロール。
- **Service Account Token Creator（`roles/iam.serviceAccountTokenCreator`）**は `iam.serviceAccounts.getAccessToken`、`getOpenIdToken`等を含み、gcloud CLIでのSA権限借用に使える。**Service Account Userだけでは権限借用できない**。
- **gcloud CLI用の認証**と**Application Default Credentials（ADC）**は異なる。通常の `gcloud auth application-default login` をしただけでは、gcloud CLIの使用主体をSAに変更できない。

## 仕組み・判断軸

### 付与する権限

| 要件 | ロール・権限 |
| --- | --- |
| VMやCloud RunへSAを関連付ける | `roles/iam.serviceAccountUser`：`iam.serviceAccounts.actAs` |
| ローカルgcloudコマンドを対象SAとして実行 | `roles/iam.serviceAccountTokenCreator`：`iam.serviceAccounts.getAccessToken` |
| OIDC IDトークンを生成 | `iam.serviceAccounts.getOpenIdToken`。IDトークンだけなら `roles/iam.serviceAccountOpenIdTokenCreator` も検討 |
| リソースへアクセス | **実際に利用するSA自身**に対象リソースへの必要最小限のロール |

これらのロールは異なる目的であり、Token Creatorが自動的に`actAs`を付与するわけではない。またSAへの関連付け権限だけで開発者がSAのアクセストークンを直接発行できるわけでもない。

### gcloud CLIの権限借用

前提：ログイン済みの本人に**対象SAへのToken Creator**が付与され、**Service Account Credentials API**が有効であること。対象SAは操作先の権限を持つこと。

- 一時的に1コマンドのみ：`--impersonate-service-account=SERVICE_ACCT_EMAIL` を指定。
- アクティブなgcloud構成で以降のgcloudコマンドに適用：`gcloud config set auth/impersonate_service_account SERVICE_ACCT_EMAIL`。
- 解除：`gcloud config unset auth/impersonate_service_account`。
- 環境変数 `CLOUDSDK_AUTH_IMPERSONATE_SERVICE_ACCOUNT` も使用可能。

認証済みユーザー → SAの短期アクセストークン発行 → SAとして対象Google Cloud APIを呼び出す → SAに付与されたIAM権限が評価される、という流れ。

短期アクセストークンの生成自体は`Token Creator`で許可されるが、操作先のアクセス権は**対象SAが持つIAMロール**による。本人に対象リソースのロールを直接付与する必要はない。

### gcloudのNamed Configuration：用途別に複数の設定を持つ

gcloud CLIは`default`だけでなく、**複数のNamed Configuration**を作成できる。構成ごとに`core/project`、`core/account`、`compute/region`、`auth/impersonate_service_account`等を別々に保存できる。ログイン済みのユーザー認証情報自体は構成ごとに分離された別ストアではない。

| 操作 | 使用する方法 |
| --- | --- |
| 作成 | `gcloud config configurations create dev-config`（既定では作成した構成を有効化） |
| 構成一覧の確認 | `gcloud config configurations list` |
| アクティブな構成の切替 | `gcloud config configurations activate prod-config` |
| 1コマンドにだけ構成を指定 | `gcloud ... --configuration=prod-config` |
| 現在のターミナル等の一連の操作で指定 | 環境変数 `CLOUDSDK_ACTIVE_CONFIG_NAME=prod-config` を指定 |
| 適用中のプロパティの確認 | `gcloud config list` |

**実務上の選定の考え方**：
- 手作業ではプロジェクト/環境/役割別に作成して必要に応じて切り替えればよい。
- スクリプトでは意図しないアクティブ構成に依存しないよう、実行時の`--configuration`指定またはそのプロセス/ターミナルだけの環境変数で構成を明示する。コマンドごとにグローバルなアクティブ構成を切り替える必要はない。
- `activate`で共有される既定のアクティブ構成を頻繁に切り替えると、並行作業や誤った本番操作の原因になり得る。
- **設定の分離≠IAMの分離**。使用するSA自体に最小権限を付与し、権限借用を許可する対象も絞る。

### gcloud CLI認証 vs ADC

| 設定 | 主な利用者 | 注意 |
| --- | --- | --- |
| `gcloud auth login` | gcloud CLI | 通常はログインしたユーザー本人として実行 |
| `gcloud config set auth/impersonate_service_account ...` | gcloud CLI | アクティブ構成でSAを権限借用。ADCには自動適用されない |
| `gcloud auth application-default login` | Google Cloudクライアントライブラリ等 | ローカルADCを設定。通常はユーザー認証情報を保存し、gcloud CLI自体の認証を切り替えない |
| `gcloud auth application-default login --impersonate-service-account=...` | 対応するクライアントライブラリ等 | 権限借用を使うADC。ライブラリに対応条件がある |

Cloud Run/Compute Engine等に関連付けたSAは、一般にメタデータサーバーを通してADCで認証情報を利用できる。開発PCにJSON鍵を置く方法より推奨される。

## 間違えやすい点

- `roles/iam.serviceAccountUser`（actAs）と`roles/iam.serviceAccountTokenCreator`（token発行）は同義ではない。
- `gcloud auth application-default login` は`gcloud auth login`とは別。gcloud CLI自身はADCを通常使わない。
- `gcloud config set auth/impersonate_service_account` は**アクティブなgcloud構成**に適用される。別の構成やADCに自動反映されるわけではない。
- 複数configは用途別の設定保存に便利だが、切り替えるだけで実際のIAM権限が追加・制限されるわけではない。
- `gcloud auth activate-service-account` + JSON鍵で操作できるが、鍵の生成・保管・漏洩リスクがある。鍵レス要件では不適切。
- Token Creatorは非常に強力。SAに権限過剰なロールがあれば権限借用したユーザーも実質そのアクセスが可能になる。**対象SA単位**など必要最小限の範囲に付与する。

### ACE試験での判断

- 「VMにSAを付ける」→ Service Account User（+ VM操作権限）。
- 「ローカルPCからSAとしてgcloudを実行、キー禁止」→ Token Creator + Impersonation。
- 「スクリプト中すべてのgcloudコマンドでSAを使う」→ `auth/impersonate_service_account` プロパティ。
- 「特定のコマンドだけSAとして使う」→ `--impersonate-service-account`。
- 「ローカルでSDKクライアントライブラリを動かす」→ ADC（必要ならSA権限借用ADC）。
- 「SAの権限テスト」→ SA本人がどのリソース権限を持つかも確認。
- 「開発・本番で別プロジェクトやSAを使い分ける」→ Named Configurationを分け、スクリプトでは構成の明示指定を検討。

## 関連ページ

- [Wikiホーム](../index.md)
- [BigQueryのクエリ実行IAM権限](../bigquery/iam-query-roles.md)
- [SSH公開鍵・OS Login・IAP](../compute-engine/ssh-public-keys-os-login-iap.md)

## 出典・検証

Google Cloud公式資料を2026-10-24に確認。

- [Manage gcloud CLI configurations](https://docs.cloud.google.com/sdk/docs/configurations)
- [gcloud configurations create](https://docs.cloud.google.com/sdk/gcloud/reference/config/configurations/create)
- [Use service account impersonation](https://docs.cloud.google.com/docs/authentication/use-service-account-impersonation)
- [Roles for service account authentication](https://docs.cloud.google.com/iam/docs/service-account-permissions)
- [Authenticate for the gcloud CLI](https://docs.cloud.google.com/sdk/docs/authenticate)
- [Set up Application Default Credentials](https://docs.cloud.google.com/docs/authentication/provide-credentials-adc)
- [How Application Default Credentials works](https://docs.cloud.google.com/docs/authentication/application-default-credentials)
