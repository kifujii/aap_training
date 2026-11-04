# 応用演習 2 - Vault Credentialと秘密情報管理

この演習では、`ansible-vault` で暗号化した機密情報を、AAPの **Vault Credential** と組み合わせて安全に利用する方法を学びます。HAProxyのstats画面に認証を追加するシナリオを例に進めます。

**前提条件:** 応用演習1を完了していること

[前に戻る](./ex1.md) | [次へ進む](./ex3.md)

---

## Section 1: ansible-vaultによる暗号化（ローカル作業）

この手順は、GitLabリポジトリをcloneしたローカル環境（自分のPCやEEビルド用サーバ等、`ansible-vault` コマンドが使える環境）で行います。

> **Note:** `ansible-vault encrypt` はコマンド実行が必要なため、**GitLab Web IDEでは代替できません**（Web IDEはブラウザ上のエディタであり、ターミナル・コマンド実行機能を持ちません）。Step 1〜2（ファイル作成・暗号化）は必ずCLIが使える環境で行ってください。Step 3（role修正）やStep 4（push）は、暗号化済みファイルをテキストとして貼り付けるだけなので、Web IDEでも代替可能です。

### Step 1: 暗号化する変数ファイルを作成する

```bash
$ cd aap-training-content   # sample-contentをpushしたリポジトリ
$ cat << 'EOF' > secrets.yml
haproxy_stats_user: admin
haproxy_stats_password: ChangeMe123!
EOF
```

### Step 2: ansible-vaultで暗号化する

```bash
$ ansible-vault encrypt secrets.yml
New Vault password:
Confirm New Vault password:
Encryption successful
```

暗号化後、`secrets.yml` の内容はテキストエディタで開いても判読できない暗号文になります。設定したVaultパスワードは忘れずに控えておいてください。

### Step 3: haproxy roleでVault変数を読み込むように変更する

`roles/haproxy/tasks/main.yml` の先頭に、暗号化ファイルを読み込むタスクを追加します。

```yaml
---
- name: include encrypted secrets
  include_vars: "{{ playbook_dir }}/../secrets.yml"

- name: install haproxy package
  package:
    name: haproxy
    state: present
# ...以下省略（既存のタスクはそのまま）
```

また、`defaults/main.yml` に `haproxy_stats_enabled: true` を追加し、`haproxy.cfg.j2` のstats認証部分（`haproxy_stats_user` / `haproxy_stats_password`）が機能する状態にします。

### Step 4: GitLabへpushする

**方法A: git CLIで行う場合（基本）**

```bash
$ git add secrets.yml roles/haproxy/
$ git commit -m "Add vault-encrypted haproxy stats credentials"
$ git push
```

**方法B: GitLab Web IDEで行う場合（代替）**

1. Step 2で暗号化された `secrets.yml` の内容（`$ANSIBLE_VAULT;1.1;AES256` から始まる暗号文）を、ローカルのテキストエディタ等で開いてコピーする
2. GitLab Project画面で「Web IDE」を開き、`secrets.yml` を新規作成してその暗号文を貼り付ける
3. Step 3で修正した `roles/haproxy/tasks/main.yml`・`defaults/main.yml` の内容も同様にWeb IDE上で書き換える
4. 「Source Control」アイコンからコミットメッセージ（例: `Add vault-encrypted haproxy stats credentials`）を入力し、「Commit to main」をクリックする

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

`deploy-web-stack` の編集画面を開きます。

### Step 2: Credentialsを追加する

既存の `managed-nodes-ssh`（Machine）に加えて、`vault-credential`（Vault）を「Credentials」欄に追加します。

| Credential種別 | 選択するCredential |
|---|---|
| Machine | `managed-nodes-ssh` |
| Vault | `vault-credential` |

**Note:** AAPのJob Templateは、Credential Typeごとに1つずつCredentialを割り当てる仕組みです。Machine Credentialとは別枠でVault Credentialを追加できます。

### Step 3: Projectを再Syncする

Section 1で変更した `secrets.yml` とrole修正を取得するため、`aap-training-content` ProjectのSyncを実行します（[基礎演習4](../basic-controller/ex4.md)参照）。

## Section 4: 実行して確認する

### Step 1: Job Templateを実行する

`deploy-web-stack` を「Launch」します。

### Step 2: 結果を確認する

PLAY RECAPで `lb` ホストの `include encrypted secrets` タスクと `copy haproxy.cfg` タスクが実行され、エラーなく完了することを確認してください。

```
TASK [haproxy : include encrypted secrets] **************************************
ok: [lb]

TASK [haproxy : install haproxy package] *****************************************
ok: [lb]

TASK [haproxy : copy haproxy.cfg] *************************************************
changed: [lb]
```

Vault Credentialを割り当てずに実行すると、`include_vars` タスクが復号に失敗し、以下のようなエラーになることも確認しておくとよいでしょう（任意）。

```
fatal: [lb]: FAILED! => {"message": "Attempting to decrypt but no vault secrets found"}
```

---

ここまでで、ansible-vaultによる機密情報の暗号化と、AAPのVault Credentialによる復号の連携ができるようになりました。次の演習では、Job Templateの定期実行と通知設定を扱います。

[次へ進む](./ex3.md)
