# 演習 7 - Job Templateを実行する

[演習6](./ex6.md) | [演習8へ](./ex8.md)

Linux と Windows のそれぞれに `note.txt` を書きます。1つのジョブに付けられる Machine 認証情報は1つなので、テンプレートは2つ作ります。

## Section 1: Linux

**Automation Execution > テンプレート** を開き、**テンプレートの作成** > **ジョブテンプレートの作成** をクリックします。

![ジョブテンプレートの作成画面](../images/aap/job-template-create.png)

| 項目 | 入力値 |
|---|---|
| 名前 | `write-note-linux` |
| インベントリー | `training-inventory` |
| プロジェクト | `aap-training-content` |
| Playbook | `write_note_linux.yml` |
| 実行環境 | 空欄 |
| 認証情報 | `linux-ssh` |
| 制限 | 空欄のまま（Playbook の対象はグループ `linux`） |

表に無い欄は空欄のままにします。**起動プロンプト** には、この時点ではチェックを入れません。実行環境も空欄です。

保存すると詳細画面に戻ります。右上の **テンプレートの起動** をクリックします。

起動すると、ジョブの **出力** が開きます。上部に成功か失敗と、Changed の数が出ます。各タスクの行は、変更が無ければ `ok`、ファイルを書いたときは `changed` です。

![ジョブの出力。見るのは上部の Changed と、Write note.txt の行。ok は緑、changed は黄](../images/aap/job-output.png)

写真は、後の演習まで進んだあとのログです。`Load secrets.yml` の行が `skipped` でも、この演習では問題ありません。

先頭の `SSH password:` と `Vault password:` は、認証情報を渡したときの表示です。パスワードそのものは出ません。その下に Python の WARNING が出ても、この演習では無視して構いません。

ジョブが成功すると、`lin1` の `/tmp/aap-training/note.txt` は次の内容になります。`training_note` の既定値は `managed by AAP` です。

```
hostname: lin1
address: <ansible_host の値>
note: managed by AAP
```

同じテンプレートをもう一度、**テンプレートの起動** から起動します。2回目は、出力の `Write note.txt` の行が `ok` になり、上部の Changed は 0 です。内容が同じなら書き直しません。

ファイルを変えずに結果だけ見るときは、**テンプレートの編集** を開き、**ジョブタイプ** を **チェック** に変えて保存してから起動します。選択肢は **実行** と **チェック** です。ファイルは変わりません。内容が今のファイルと違うタスクは `changed`、同じ内容のタスクは `ok` になります。

![ジョブタイプを開いたところ。選択肢はチェックと実行。横の起動プロンプトは外したまま](../images/aap/job-type.png)

写真の認証情報に `vault-training` が見えるのは、演習9で足したあとです。この演習の認証情報は `linux-ssh` だけです。

起動画面には「チェックモード」という項目はありません。ジョブタイプの横の **起動プロンプト** にチェックが無いと、起動画面でジョブタイプを変えても **実行** のままです。この演習では起動画面ではなく、編集画面のジョブタイプを変えます。確認が終わったら、ジョブタイプを **実行** に戻して保存します。

## Section 2: Windows

`write-note-linux` をコピーできる場合はコピーし、コピーのメニューが無い場合は同じ項目で新規作成します。違う箇所だけ次のとおりです。

| 項目 | 入力値 |
|---|---|
| 名前 | `write-note-windows` |
| Playbook | `write_note_windows.yml` |
| 認証情報 | `windows-psrp` |

起動して成功することを確認します。ファイルは `C:\temp\aap-training\note.txt` です。中身の形は Linux と同じで、1行目は `hostname: win1` です。

Kerberos の場合は、演習5で `krb5.conf` を置いてから実行します。失敗したら、レルム、KDC、ユーザー名が `ユーザー@レルム` になっているかを確認します。

## Section 3: 読み取り用テンプレート

同じ手順で、変更しない確認用を2つ作ります。

| 名前 | Playbook | 認証情報 |
|---|---|---|
| `verify-note-linux` | `verify_note_linux.yml` | `linux-ssh` |
| `verify-note-windows` | `verify_note_windows.yml` | `windows-psrp` |

インベントリーとプロジェクトは、書くときと同じです。両方を起動し、成功することを確認します。`note.txt` にホスト名または `managed by AAP` が無いと、ここで失敗します。

## 次

[演習8](./ex8.md) で、ファイルに書くメモを起動時に入力できるようにします。
