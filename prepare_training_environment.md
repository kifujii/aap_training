# 事前準備（確認リスト）

[READMEへ](./README.md)

この文書は構築手順ではありません。演習の前に、使う値を控えるための一覧です。

## 1. AAPの情報

| 確認項目 | 控える値 |
|---|---|
| AAPのURL | `aap.example.local`（`https://` は付けずに控える。資料では `https://<AAPのURL>/` の形で使う） |
| ログインユーザー名 / パスワード | 配布されたもの。応用演習1のユーザーとチームの作成、応用演習3の実行環境の作成のため、システム管理者のユーザーであること |
| バージョン | 2.5 以降。画面手順は 2.7 の日本語UIで確認 |
| 組織名 | 多くの環境では `Default` |
| 名前の接頭辞 | ログイン名から候補を控える。整え方は [演習1](./basic-intro/ex1.md) |
| 作業用サーバと SSH ユーザー | [演習9](./basic-controller/ex9.md) の `ansible-vault`、応用演習4・7・8 の `curl` と `python3` を実行するサーバ。講師が案内する。AAP の URL へ HTTPS で接続できること |
| AAP サーバへ SSH するユーザー（応用演習3） | AAP のコンテナを動かしているユーザー。カスタム実行環境を AAP サーバでビルドするときに使う |
| `registry.redhat.io` のアカウント（応用演習3） | ベースイメージを取るときのユーザー名とパスワード |
| SMTP サーバ（既存サーバを使用する場合の応用演習2） | ホスト名、ポート、差出人アドレス。認証が必要ならユーザー名とパスワード。環境内に無い場合は、応用演習2を飛ばす。AWS でサーバを作る場合は、応用演習2で `lin1` に検証用のメールサーバを入れるので不要 |

ブラウザでログインし、左メニューに **Automation Execution**、**Automation Decisions**、**Access Management** があることを確認してください。Demo で始まるサンプルがあっても、演習では別名で新規作成します。

AAP のコンテナを動かしているユーザーは、AAP 全体を操作できます。応用演習3でこのユーザーを使うときは、応用演習3と削除手順に書いてあるコマンドだけを実行します。`podman stop` や `podman rm` で AAP のコンテナを止めたり消したりすると、AAP 全体が止まります。

AAP のインストールとユーザー作成は、この資料の範囲外です。

## 2. 対象サーバの情報

演習では対象の Linux と Windows の、決まったディレクトリへファイルを作成します。
既存サーバを使用する場合は、以下のパスにファイルを作成しても問題のないものを使用してください。また、以下の接続方法が許可されているかも確認してください。

| OS | 書くファイル | 接続方法 |
|---|---|---|
| Linux | `/tmp/<name_prefix>-aap-training/note.txt` | SSH |
| Windows | `C:\temp\<name_prefix>-aap-training\note.txt` | PSRP |

Linux と Windows の両方を行います。既存サーバを使用するか、AWSで新規作成するかで演習内容が変わります。

| 状況 | 演習の進み方 | 事前に控えるもの |
|---|---|---|
| Linux と Windows がすでにある | [演習4](./basic-intro/ex4.md) は飛ばす。インターネットへ出ない前提で進める | 各サーバのアドレス、ログインユーザー、パスワード。Windows がドメイン参加済みなら、レルム名と Windows の FQDN（AAP から名前解決できること） |
| 対象サーバが無い | [演習3](./basic-intro/ex3.md) の Section 3 のあと、[演習4](./basic-intro/ex4.md) で AWS に作る。インターネットへ出られる前提 | AWS のアクセスキー、シークレットキー、リージョン。サーバのパスワードは演習4で自分で決める。1人で VPC を1つ使うので、受講者の人数分の VPC をリージョンの上限（初期値は5つ）の中で作れること |

すでにサーバがある場合、ログインユーザーが `/tmp` と `C:\temp` の下に自分のディレクトリを作れることを確認します。Playbook が `/tmp/<name_prefix>-aap-training` と `C:\temp\<name_prefix>-aap-training` を作り、その中にファイルを書きます。`<name_prefix>` は、上で控える接頭辞です。

Windows の接続は次のとおりです。

| Windows の状態 | 認証 | インベントリーに書く変数 |
|---|---|---|
| ドメインに参加していない（AWS で作るサーバを含む） | NTLM | `ansible_connection: psrp` と `ansible_psrp_auth: ntlm` |
| ドメインに参加している | Kerberos | `ansible_host` に FQDN、`ansible_connection: psrp`、`ansible_psrp_auth: kerberos`、`ansible_psrp_protocol: https`、`ansible_psrp_port: 5986` |

サンプルのアドレス `203.0.113.10` と `203.0.113.11` は例です。実行前に実在のアドレスへ置き換えます。

## 3. GitLab の接続情報

| 確認項目 | 控える値 |
|---|---|
| GitLabのURL | `https://gitlab.example.local/` |
| ログインユーザー名 / パスワード | AAP の Source Control 認証情報にも、同じユーザー名とパスワードを入れる |
| プロジェクトを作る Namespace | グループ名、またはユーザー名 |
| デフォルトブランチ | この資料では `main` |

ファイルの作成は Web IDE で行います。AAP がリポジトリを読む認証は、GitLab のログインユーザー名とパスワードです。

二要素認証が有効なユーザーは、Git のパスワード認証が拒否されることがあります。その場合は、二要素認証を使っていないユーザーで進めてください。

[応用演習6](./advanced/ex6.md) では、GitLab から AAP へ Webhook を送ります。AAP が社内のアドレスにある場合、GitLab の管理者設定 **Admin area > Settings > Network > Outbound requests** で、**Allow requests to the local network from webhooks and integrations** にチェックが入っていることを確認します。AAP のアドレスを **Local IP addresses and domain names that hooks and integrations can access** に書く方法でも構いません。

## 4. インターネット接続の有無（補足）

| サーバを用意する方法 | インターネット接続の必要性 |
|---|---|
| 既存サーバを使用 | インターネットへ接続せず演習可能。[応用演習3](./advanced/ex3.md) だけは、ベースイメージ（`registry.redhat.io`）とコレクション（Ansible Galaxy）の取得に接続が必要 |
| AWS でサーバを作る | 演習環境・AAPからインターネットへの接続が必要 |

## 5. サンプルファイル

[sample-content/](./sample-content/) を、[演習5](./basic-controller/ex5.md) の Web IDE で GitLab に置きます。AWS で作る場合は、[演習4](./basic-intro/ex4.md) で先に一部のファイルだけ置きます。

## 次

[演習1](./basic-intro/ex1.md) で AAP にログインし、接頭辞を決めます。
