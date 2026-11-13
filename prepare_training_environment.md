# 事前準備（確認リスト）

[READMEへ](./README.md)

この文書は構築手順ではありません。演習の前に、使う値を控えるための一覧です。

## 1. AAP

| 確認項目 | 控える値 |
|---|---|
| AAPのURL | `https://aap.example.local/` |
| ログインユーザー名 / パスワード | 配布されたもの |
| バージョン | 2.5 以降。画面手順は 2.7 の日本語UIで確認 |
| 組織名 | 多くの環境では `Default` |
| 名前の接頭辞 | ログイン名から候補を控える。整え方は [演習1](./basic-intro/ex1.md) |

ブラウザでログインし、左メニューに **Automation Execution**、**Automation Decisions**、**Access Management** があることを確認してください。Demo で始まるサンプルがあっても、演習では別名で新規作成します。

資料中の `<name_prefix>` は、演習1で決める接頭辞に読み替えます。事前準備では、ログイン名から候補を控えるところまでです。

## 2. 対象サーバ

演習で行う変更は、Linux と Windows の決まったディレクトリへファイルを書くことです。

| OS | 書くファイル | 接続 |
|---|---|---|
| Linux | `/tmp/<name_prefix>-aap-training/note.txt` | SSH |
| Windows | `C:\temp\<name_prefix>-aap-training\note.txt` | PSRP |

Linux と Windows の両方を行います。違うのは、そのサーバをすでにあるものを使うか、AWS で作るかです。

| 状況 | 演習の進み方 | 事前に控えるもの |
|---|---|---|
| Linux と Windows がすでにある | [演習4](./basic-intro/ex4.md) は飛ばす。インターネットへ出ない前提で進める | 各サーバのアドレス、ログインユーザー、パスワード。Windows がドメイン参加済みならレルム名 |
| 対象サーバが無い | [演習3](./basic-intro/ex3.md) の Section 3 のあと、[演習4](./basic-intro/ex4.md) で AWS に作る。インターネットへ出られる前提 | AWS のアクセスキー、シークレットキー、リージョン。サーバのパスワードは演習4で自分で決める |

すでにサーバがある場合、ログインユーザーが `/tmp` と `C:\temp` の下に自分のディレクトリを作れることを確認します。Playbook が `/tmp/<name_prefix>-aap-training` と `C:\temp\<name_prefix>-aap-training` を作り、その中にファイルを書きます。

Windows の接続の書き方は [演習3](./basic-intro/ex3.md) です。

サンプルのアドレス `203.0.113.10` と `203.0.113.11` は例です。実行前に実在のアドレスへ置き換えます。

## 3. GitLab

| 確認項目 | 控える値 |
|---|---|
| GitLabのURL | `https://gitlab.example.local/` |
| ログインユーザー名 / パスワード | AAP の Source Control 認証情報にも、同じユーザー名とパスワードを入れる |
| プロジェクトを作る Namespace | グループ名、またはユーザー名 |
| デフォルトブランチ | この資料では `main` |

ファイルの作成は Web IDE で行います。AAP がリポジトリを読む認証は、GitLab のログインユーザー名とパスワードです。

二要素認証が有効なユーザーは、Git のパスワード認証が拒否されることがあります。その場合は、二要素認証を使っていないユーザーで進めてください。

## 次

[演習1](./basic-intro/ex1.md) で AAP にログインし、接頭辞を決めます。
