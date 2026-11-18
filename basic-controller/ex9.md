# 演習 9 - Vault

[演習8](./ex8.md) | [演習10へ](./ex10.md)

> **前提:** [演習8](./ex8.md) の制限の起動プロンプトと Survey。暗号化に使うサーバ（この研修では AAP サーバ）へ SSH できること

この演習では文字列の暗号化を取り扱います。
`training_secret` の値だけを暗号化して GitLab に置き、ジョブの中で復号してファイルへ書きます。暗号化は、`ansible-vault` コマンドが使えるサーバで行います。この研修では AAP サーバを使います。作業ディレクトリは、そのサーバの `/tmp` です。Web IDE には、`ansible-vault encrypt_string` が表示した文字列を貼ります。Web IDE では `ansible-vault` を実行できません。

書かれるファイル:

| OS | パス | 権限 |
|---|---|---|
| Linux | `/tmp/<name_prefix>-aap-training/secret.txt` | `0600` |
| Windows | `C:\temp\<name_prefix>-aap-training\secret.txt` | 接続ユーザーが書き込める場所 |

## Section 1: ansible-vault が使えるサーバで値を暗号化する

この研修では AAP サーバへ SSH します。

```bash
ssh <ユーザー>@<AAPサーバ>
cd /tmp
command -v ansible-vault
```

コマンドが表示された場合は、その場で次を実行します。暗号化する文字列は、コマンドの引数には書きません。引数に書くと、シェルの履歴に残ります。

```bash
cd /tmp
ansible-vault encrypt_string --prompt --output '/tmp/<name_prefix>-secrets.yml'
```

入力要求は、上から次の順です。

1. `New Vault password:` と `Confirm New Vault password:` には、同じパスワードを入れます。これは復号用です。`TrainingSecret1` とは別の、自分だけが知る値を決め、控えます。
2. `Variable name (enter for no name):` には `training_secret` と入れます。
3. `String to encrypt (hidden):` には `TrainingSecret1` と入れます。別の英数字にしてもよいです。入力した文字は画面に出ません。

`Encryption successful` と出て、`/tmp/<name_prefix>-secrets.yml` の先頭が `training_secret: !vault |`、次の行が `$ANSIBLE_VAULT;1.1;AES256` なら、値だけが暗号化されています。このファイルの全文をコピーします。

`ansible-vault` がこのサーバに無く、コンテナで動いている場合は、名前に `controller` を含むコンテナを選びます。`<コンテナ>` は `podman ps` に出た名前です。入力要求は、上の1から3と同じです。画面の `training_secret: !vault |` から、暗号の最後の行までをコピーします。`Encryption successful` は含めません。

```bash
cd /tmp
podman ps
podman exec -it <コンテナ> ansible-vault encrypt_string --prompt
```

SSH でこのサーバに入れない、または `ansible-vault` も `podman` も使えない場合は、この演習を全部飛ばして [演習10](./ex10.md) へ進みます。`secret.txt` は作られません。サーバへ入れるようになったら、Section 1 から実施します。

## Section 2: Web IDEに貼る

GitLab の **Code** を開き、**Web IDE** で、プロジェクトの直下に `secrets.yml` を作り、コピーした全文を貼ります。メニューが **Edit** の環境では、その中の **Web IDE** を選びます。平文の `TrainingSecret1` は貼りません。

**Commit to main** でコミットします。

<details>
<summary>git CLIで行う場合</summary>

`/tmp/<name_prefix>-secrets.yml` を、clone した作業ディレクトリへコピーしてから push します。

```bash
git add secrets.yml
git commit -m "Add encrypted training secret"
git push
```

</details>

コミットしたあと、AAP のプロジェクト `<name_prefix>-aap-training-content` を同期します。

## Section 3: Vault認証情報

**認証情報の作成** を開きます。

![Vault 認証情報。Vault Password の欄](../images/aap/credential-vault.png)

| 項目 | 入力値 |
|---|---|
| 名前 | `<name_prefix>-vault-training` |
| 認証情報タイプ | `Vault` |
| Vault Password | Section 1 の `New Vault password:` に入れたパスワード |

`String to encrypt (hidden):` に入れた値ではありません。

## Section 4: ジョブで復号する

`<name_prefix>-write-note-linux` の編集で、認証情報に `<name_prefix>-vault-training` を追加します。`<name_prefix>-linux-ssh` も付けたままにします。**Extra variables** は、演習8で残した `name_prefix` の行を残したまま、次の行を足します。起動画面にはこの欄は出ないので、編集画面で保存します。

```yaml
name_prefix: tanaka01
training_use_vault: true
```

`tanaka01` は、演習7で保存してある自分の接頭辞のままにします。

![詳細画面。認証情報は linux-ssh と vault-training、Extra variables は training_use_vault: true。実行環境に既定の名前が見えても、編集画面では空欄のままにする](../images/aap/template-launch.png)

![編集画面の Extra variables](../images/aap/extra-variables.png)

写真の Extra variables は `training_use_vault` だけです。自分の画面では、その上に `name_prefix` の行もあります。制限の起動プロンプトは、演習8で入れたままにします。

起動すると、演習8と同じ **Prompt on Launch** が開きます。**制限** は空欄のまま **Next** を押します。**Survey** は `managed by AAP` のまま **Next** を押し、**Review** で **Finish** を押します。メモの文面は変えません。`Show the note written to note.txt` は `note.txt の note は managed by AAP です` のままです。成功すると、`Show the secret written to secret.txt` に `secret.txt を書きました。権限は 0600 です` と出ます。ログに出るのは、ファイルを書いたことと権限です。復号した値は出ません。実際の運用でも、復号した値はジョブのログに出しません。ファイルは `/tmp/<name_prefix>-aap-training/secret.txt` です。

`<name_prefix>-write-note-windows` にも `<name_prefix>-vault-training` と `<name_prefix>-windows-psrp` を付けます。Extra variables は、`name_prefix` の行を残したまま `training_use_vault: true` を足します。起動画面に **Prompts** は出ません。**Survey** は `managed by AAP` のまま **Finish** を押します。成功すると、`Show the secret written to secret.txt` に `secret.txt を書きました` と出ます。ファイルは `C:\temp\<name_prefix>-aap-training\secret.txt` です。

`training_use_vault: true` を足し忘れると、`Load secrets.yml when vault is enabled`、`Write secret.txt`、`Read secret.txt` は `skipped` になり、`Show the secret written to secret.txt` は出ません。暗号化パスワードが違うと、ジョブは `secrets.yml` の読み込みで失敗します。

## 次

[演習10](./ex10.md) で、確認用ジョブを毎日実行する予定を作ります。
