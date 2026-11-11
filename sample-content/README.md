# sample-content

演習で GitLab に置くファイルです。編集は GitLab の Web IDE を使います。

## ファイル

| ファイル | 用途 |
|---|---|
| `inventory.yml` | Linux `lin1` と Windows `win1`。アドレスは例示用の `203.0.113.10` / `203.0.113.11` |
| `group_vars/all.yml` | `training_note` と、AWS で使うリージョンなどの既定値 |
| `write_note_linux.yml` | Linux の `/tmp/aap-training/note.txt` を書く |
| `write_note_windows.yml` | Windows の `C:\temp\aap-training\note.txt` を書く |
| `verify_note_linux.yml` | Linux の note.txt を読む |
| `verify_note_windows.yml` | Windows の note.txt を読む |
| `create_aws_servers.yml` | サーバが無いときだけ実行する。VPC、サブネット、Linux、Windows を作る |
| `remove_aws_servers.yml` | 上記のリソースを削除する |
| `notify_aws.yml` | AWS で作った場合の通知確認。S3 に書いて読み戻す |
| `requirements.yml` | **AWS で作る場合だけ** GitLab に置く。`amazon.aws` 10.3.0 |
| `krb5.conf.example` | ドメイン参加済み Windows を Kerberos で接続するときの見本 |

`requirements.yml` を置いたプロジェクトを、インターネットへ出られない環境で同期すると失敗します。サーバがすでにある場合は、このファイルを GitLab に置かないでください。

パスワードと `resource_prefix` はファイルに書きません。Job 起動時の Extra variables で渡します。

## Web IDEで置く

1. GitLab のプロジェクトを開く
2. 右上の **Code** を開き、**Web IDE** をクリックする。**Edit** の中にある環境では、そちらから開く
3. 左のファイル一覧からファイルを作り、このディレクトリの内容を貼り付ける
4. **Source Control** を開き、メッセージを書いて **Commit to main** をクリックする

サーバがすでにある場合は `requirements.yml` を除きます。AWS で作る場合は含めます。`inventory.yml` の `ansible_host` は、コミット前に自分のサーバのアドレスへ書き換えます。

<details>
<summary>git CLIで行う場合</summary>

```bash
git clone https://gitlab.example.local/<namespace>/aap-training-content.git
cd aap-training-content
# このディレクトリのファイルをコピーする。サーバがある環境では requirements.yml をコピーしない
git add .
git commit -m "Add training playbooks"
git push
```

ユーザー名とパスワードを聞かれたら、GitLab のログインに使うものを入力します。

</details>
