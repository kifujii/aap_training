# sample-content

[READMEへ](../README.md)

演習で GitLab に置くファイルです。編集は GitLab の Web IDE を使います。

## ファイル

| ファイル | 用途 |
|---|---|
| `inventory.yml` | Linux `lin1` と Windows `win1`。アドレスは例示用の `203.0.113.10` / `203.0.113.11` |
| `group_vars/all.yml` | `training_note` と、AWS で使うリージョンなどの既定値 |
| `write_note_linux.yml` | Linux の `/tmp/<name_prefix>-aap-training/note.txt` を書く |
| `write_note_windows.yml` | Windows の `C:\temp\<name_prefix>-aap-training\note.txt` を書く |
| `verify_note_linux.yml` | Linux の note.txt を読む |
| `verify_note_windows.yml` | Windows の note.txt を読む |
| `create_aws_servers.yml` | サーバが無いときだけ実行する。VPC、サブネット、Linux、Windows を作る |
| `setup_mailpit.yml` | 応用演習2（AWS で作った場合）。`lin1` に検証用のメールサーバ Mailpit を入れる |
| `show_mailpit_messages.yml` | 応用演習2（AWS で作った場合）。Mailpit が受け取ったメールの件名と宛先を表示する |
| `remove_aws_servers.yml` | `create_aws_servers.yml` で作った AWS のリソースを削除する |
| `launch_note.yml` | 応用演習5。既定の実行環境から、Extra variables の `name_prefix` を付けた `write-note-linux` を起動する |
| `use_ini_file.yml` | 応用演習3。`community.general.ini_file` で値を書き、カスタム実行環境の前後を比べる |
| `check_become.yml` | 演習7。sudo で `whoami` を実行し、`root` になることを確かめる |
| `show_handed_note.yml` | 演習11。前のジョブが `set_stats` で渡したメモを表示する |
| `remove_my_notes.yml` | 学習後に、対象サーバの `/tmp/<name_prefix>-aap-training` と `C:\temp\<name_prefix>-aap-training` を消す |
| `remove_my_lab.yml` | 学習後に、自分の接頭辞の AAP オブジェクトを削除する |
| `requirements.yml` | **AWS で作る場合だけ** GitLab に置く。`amazon.aws` 10.3.0 |

`requirements.yml` を置いたプロジェクトを、インターネットへ出られない環境で同期すると失敗します。既存サーバを使用する場合は、このファイルを GitLab に置かないでください。

パスワードと `resource_prefix` はファイルに書きません。Job 起動時の Extra variables で渡します。

## Web IDEで置く

1. GitLab のプロジェクトを開く
2. 右上の **Code** を開き、**Web IDE** をクリックする。**Edit** の中にある環境では、そちらから開く
3. 左のファイル一覧からファイルを作り、このディレクトリの内容を貼り付ける
4. **Source Control** を開き、メッセージを書いて **Commit to main** をクリックする

どのファイルをいつ置くかは、各演習に書いてあります。既存サーバを使用する場合は、`requirements.yml` を置きません。`inventory.yml` の `ansible_host` は、コミット前に自分のサーバのアドレスへ書き換えます。

<details>
<summary>git CLIで行う場合</summary>

```bash
git clone 'https://gitlab.example.local/<namespace>/<name_prefix>-aap-training-content.git'
cd '<name_prefix>-aap-training-content'
# このディレクトリのファイルをコピーする。サーバがある環境では requirements.yml をコピーしない
git add .
git commit -m "Add training playbooks"
git push
```

ユーザー名とパスワードを聞かれたら、GitLab のログインに使うものを入力します。

</details>
