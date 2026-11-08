# 演習 9 - Vault

[演習8](./ex8.md) | [演習10へ](./ex10.md)

`training_secret` を暗号化して GitLab に置き、ジョブの中で復号してファイルへ書きます。暗号化は AAP サーバで行います。Web IDE には、暗号化したあとの文字列を貼ります。Web IDE では `ansible-vault` を実行できません。

書かれるファイル:

| OS | パス | 権限 |
|---|---|---|
| Linux | `/var/tmp/aap-training/secret.txt` | `0600` |
| Windows | `C:\Users\Public\aap-training\secret.txt` | 接続ユーザーが書き込める場所 |

## Section 1: AAPサーバで暗号化する

AAP サーバへ SSH します。

```bash
ssh <ユーザー>@<AAPサーバ>
command -v ansible-vault
```

コマンドが表示された場合は、そのシェルで次を実行します。

```bash
mkdir -p ~/aap-vault && cd ~/aap-vault
printf '%s\n' 'training_secret: TrainingSecret1' > secrets.yml
ansible-vault encrypt secrets.yml
```

`TrainingSecret1` は例です。英数字で自分の値にしてください。`ansible-vault encrypt` が聞くパスワードは、この文字列とは別です。復号用なので、自分だけが知る値を決め、控えます。

`cat secrets.yml` の先頭が `$ANSIBLE_VAULT;1.1;AES256` になっていれば暗号化できています。表示された全文をコピーします。

`ansible-vault` がホストに無く、コンテナで動いている場合:

```bash
podman ps
```

名前に `controller` を含むコンテナを選び、次のようにファイルを渡します。`<コンテナ>` は `podman ps` に出た名前です。

```bash
mkdir -p ~/aap-vault && cd ~/aap-vault
printf '%s\n' 'training_secret: TrainingSecret1' > secrets.yml
podman cp secrets.yml <コンテナ>:/tmp/secrets.yml
podman exec -it <コンテナ> ansible-vault encrypt /tmp/secrets.yml
podman cp <コンテナ>:/tmp/secrets.yml ./secrets.yml
cat secrets.yml
```

## Section 2: Web IDEに貼る

GitLab の **Code** を開き、**Web IDE** で、プロジェクトの直下に `secrets.yml` を作り、コピーした全文を貼ります。メニューが **Edit** の環境では、その中の **Web IDE** を選びます。暗号化前の1行は貼りません。

**Commit to main** でコミットします。

<details>
<summary>git CLIで行う場合</summary>

AAP サーバ上の暗号化した `secrets.yml` を、clone した作業ディレクトリへコピーしてから push します。

```bash
git add secrets.yml
git commit -m "Add encrypted training secret"
git push
```

</details>

コミットしたあと、AAP のプロジェクト `aap-training-content` を同期します。

## Section 3: Vault認証情報

**認証情報の作成** を開きます。

![Vault 認証情報。Vault Password の欄](../images/aap/credential-vault.png)

| 項目 | 入力値 |
|---|---|
| 名前 | `vault-training` |
| 認証情報タイプ | `Vault` |
| Vault Password | Section 1 で `ansible-vault encrypt` に入力したパスワード |

`TrainingSecret1` の方ではありません。

## Section 4: ジョブで復号する

`write-note-linux` の編集で、認証情報に `vault-training` を追加します。`linux-ssh` も付けたままにします。変数に次を入れます。

```yaml
training_use_vault: true
```

起動します。成功すると `/var/tmp/aap-training/secret.txt` の内容は `TrainingSecret1` です。ログには値を出さないようにしてあります。

`write-note-windows` にも `vault-training` と `windows-psrp` を付け、同じ変数で起動します。ファイルは `C:\Users\Public\aap-training\secret.txt` です。

変数を付け忘れると `secrets.yml` は読まれず、`secret.txt` は作られません。暗号化パスワードが違うと、ジョブは `secrets.yml` の読み込みで失敗します。

## 次

[演習10](./ex10.md) で、確認用ジョブを毎日実行する予定を作ります。
