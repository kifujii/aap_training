# 応用演習 2 - Vault Credentialと秘密情報管理

この演習では、`ansible-vault` で暗号化した機密情報を、AAPの **Vault Credential** と組み合わせて安全に利用する方法を学びます。管理対象にローカルの管理ユーザーを追加し、そのパスワードを暗号化して渡すシナリオで進めます。

**前提条件:** 応用演習1を完了していること

[前に戻る](./ex1.md) | [次へ進む](./ex3.md)

---

## Section 1: ansible-vaultによる暗号化（ローカル作業）

この手順は、GitLabリポジトリをcloneしたローカル環境（自分のPCやEEビルド用サーバ等、`ansible-vault` コマンドが使える環境）で行います。

> **Note:** `ansible-vault encrypt` はコマンド実行が必要なため、**GitLab Web IDEでは代替できません**（Web IDEはブラウザ上のエディタであり、ターミナル・コマンド実行機能を持ちません）。Step 1〜2（ファイル作成・暗号化）は必ずCLIが使える環境で行ってください。Step 4のpushは、暗号化済みファイルをテキストとして貼り付けるだけなので、Web IDEでも代替できます。

### Step 1: 暗号化する変数ファイルを作成する

```bash
$ cd aap-training-content   # sample-contentをpushしたリポジトリ
$ cat << 'EOF' > secrets.yml
baseline_admin_password: ChangeMe123!
EOF
```

パスワードは演習用の例です。実際の値に置き換えてください。平文のまま push しません。

### Step 2: ansible-vaultで暗号化する

```bash
$ ansible-vault encrypt secrets.yml
New Vault password:
Confirm New Vault password:
Encryption successful
```

暗号化後、`secrets.yml` の内容はテキストエディタで開いても判読できない暗号文になります。設定したVaultパスワードは忘れずに控えておいてください。

### Step 3: baseline roleが暗号化ファイルを読む条件を確認する

[sample-content/roles/baseline/tasks/main.yml](../sample-content/roles/baseline/tasks/main.yml) は、`baseline_create_admin_user` が true のときだけ `secrets.yml` を読み、ユーザー `infra` を `wheel` グループに作ります。既定の `group_vars/all.yml` ではこのフラグは false なので、基礎演習の実行ではスキップされています。

Job Template `apply-baseline` の Extra variables に、次を追加します。フラグを `secrets.yml` の中だけに書くと、ファイルを読む前に条件が false のままになり、パスワードが読まれません。

```yaml
baseline_create_admin_user: true
```

### Step 4: GitLabへpushする

**方法A: git CLIで行う場合（基本）**

```bash
$ git add secrets.yml
$ git commit -m "Add vault-encrypted local admin password"
$ git push
```

**方法B: GitLab Web IDEで行う場合（代替）**

1. Step 2で暗号化された `secrets.yml` の内容（`$ANSIBLE_VAULT;1.1;AES256` から始まる暗号文）を、ローカルのテキストエディタ等で開いてコピーする
2. GitLab Project画面で「Web IDE」を開き、`secrets.yml` を新規作成してその暗号文を貼り付ける
3. 「Source Control」アイコンからコミットメッセージ（例: `Add vault-encrypted local admin password`）を入力し、「Commit to main」をクリックする

**Note:** 暗号化済みの `secrets.yml` はリポジトリにpushしても問題ありません（内容が暗号化されているため）。**平文のVaultパスワード自体をリポジトリにコミットしないよう注意してください。**

## Section 2: AAP側でVault Credentialを作成する

### Step 1: Credential作成画面を開く

**Automation Execution > Infrastructure > 認証情報** を開き、**認証情報の作成** をクリックします。認証情報タイプは `Vault` です。

### Step 2: 入力する

| 項目 | 入力値 |
|---|---|
| Name | `vault-credential` |
| Organization | 基礎演習1で確認したOrganization |
| Credential Type | `Vault` |
| Vault Password | Section 1 Step 2で設定したVaultパスワード |
| Vault Identifier | 空欄（複数のVault IDを使い分ける場合のみ指定） |

「Save」をクリックして保存します。

**Note:** 複数の暗号化ファイルで異なるVaultパスワードを使い分けたい場合は、`ansible-vault encrypt --vault-id <ID>@prompt secrets.yml` のようにVault IDを指定し、Credential側の「Vault Identifier」も同じ値に設定します。

## Section 3: Job TemplateにVault Credentialを割り当てる

### Step 1: Job Template編集画面を開く

`apply-baseline` の編集画面を開きます。

### Step 2: Credentialsを追加する

既存の `managed-nodes-ssh`（Machine）に加えて、`vault-credential`（Vault）を「Credentials」欄に追加します。

| Credential種別 | 選択するCredential |
|---|---|
| Machine | `managed-nodes-ssh` |
| Vault | `vault-credential` |

**Note:** AAPのJob Templateは、Credential Typeごとに1つずつCredentialを割り当てる仕組みです。Machine Credentialとは別枠でVault Credentialを追加できます。

### Step 3: Projectを再Syncする

Section 1で追加した `secrets.yml` を取得するため、`aap-training-content` ProjectのSyncを実行します（[基礎演習4](../basic-controller/ex4.md)参照）。

## Section 4: 実行して確認する

### Step 1: Job Templateを実行する

`apply-baseline` を「Launch」します。

### Step 2: 結果を確認する

`srv1` と `srv2` で `Load the encrypted admin password` が失敗せず、`Create the local admin user` が `changed` になることを確認してください。パスワード本体は `no_log` によりログへ出ません。同じJobをもう一度実行すると、ユーザー作成タスクは `ok` になります。

Vault Credentialを割り当てずに実行すると、`include_vars` が復号に失敗します。次のようなエラーになることを確認しておくと、Credentialの役割が分かります（任意）。

```
fatal: [srv1]: FAILED! => {"message": "Attempting to decrypt but no vault secrets found"}
```

---

ここまでで、ansible-vaultによる機密情報の暗号化と、AAPのVault Credentialによる復号の連携ができるようになりました。次の演習では、Job Templateの定期実行と通知設定を扱います。

[次へ進む](./ex3.md)
