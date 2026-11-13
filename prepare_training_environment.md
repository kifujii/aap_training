# 事前準備（確認リスト）

[READMEへ](./README.md)

この文書は構築手順ではありません。演習の前に、使う値を控えるための一覧です。

## 1. AAPの情報

| 確認項目 | 控える値 |
|---|---|
| AAPのURL | `https://aap.example.local/` |
| ログインユーザー名 / パスワード | 配布されたもの |
| バージョン | 2.5 以降。画面手順は 2.7 の日本語UIで確認 |
| 組織名 | 多くの環境では `Default` |
| 名前の接頭辞 | ログイン名から候補を控える。整え方は [演習1](./basic-intro/ex1.md) |

ブラウザでログインし、左メニューに **Automation Execution**、**Automation Decisions**、**Access Management** があることを確認してください。Demo で始まるサンプルがあっても、演習では別名で新規作成します。

AAP のインストールとユーザー作成は、この資料の範囲外です。

## 2. 対象サーバの情報

演習では対象の Linux と Windows の、決まったディレクトリへファイルを作成します。
既存サーバを使う場合は、以下のパスにファイルを作成しても問題のないものを使用してください。また、以下の接続方法が許可されているかも確認してください。

| OS | 書くファイル | 接続方法 |
|---|---|---|
| Linux | `/tmp/<name_prefix>-aap-training/note.txt` | SSH |
| Windows | `C:\temp\<name_prefix>-aap-training\note.txt` | PSRP |

Linux と Windows の両方を行います。既存サーバを使用するか、AWSで新規作成するかで演習内容が変わります。

| 状況 | 演習の進み方 | 事前に控えるもの |
|---|---|---|
| Linux と Windows がすでにある | [演習4](./basic-intro/ex4.md) は飛ばす。インターネットへ出ない前提で進める | 各サーバのアドレス、ログインユーザー、パスワード。Windows がドメイン参加済みならレルム名 |
| 対象サーバが無い | [演習3](./basic-intro/ex3.md) の Section 3 のあと、[演習4](./basic-intro/ex4.md) で AWS に作る。インターネットへ出られる前提 | AWS のアクセスキー、シークレットキー、リージョン。サーバのパスワードは演習4で自分で決める |

すでにサーバがある場合、ログインユーザーが `/tmp` と `C:\temp` の下に自分のディレクトリを作れることを確認します。Playbook が `/tmp/<name_prefix>-aap-training` と `C:\temp\<name_prefix>-aap-training` を作り、その中にファイルを書きます。`<name_prefix>` は、上で控える接頭辞です。

Windows の接続は次のとおりです。

| Windows の状態 | 認証 | インベントリに書く変数 |
|---|---|---|
| ドメインに参加していない（AWS で作るサーバを含む） | NTLM | `ansible_connection: psrp` と `ansible_psrp_auth: ntlm` |
| ドメインに参加している | Kerberos | `ansible_connection: psrp`、`ansible_psrp_auth: kerberos`、`ansible_psrp_protocol: https`、`ansible_psrp_port: 5986` |

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

## 4. インターネット接続の有無

| 状況 | 扱い |
|---|---|
| サーバがすでにある | 演習中にインターネットへ出ない。`requirements.yml` は GitLab に置かない。カスタム実行環境の作成は、この研修では AAP サーバがコンテナイメージを取得できるときだけ行う |
| AWS でサーバを作る | インターネットへ出られる。`requirements.yml` を置き、AAP 2.7 の既定実行環境に入っている `amazon.aws` 10.3.0 を使う |

## 5. サンプルファイル

[sample-content/](./sample-content/) を、[演習5](./basic-controller/ex5.md) の Web IDE で GitLab に置きます。AWS で作る場合は、[演習4](./basic-intro/ex4.md) で先に一部のファイルだけ置きます。

## 次

[演習1](./basic-intro/ex1.md) で AAP にログインし、接頭辞を決めます。
