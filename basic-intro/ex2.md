# 演習 2 - Credentialの作成

[前に戻る](./ex1.md)

---

この演習では、AAPが管理対象ノードへ接続するために使う **Credential**（認証情報）を作成します。Credentialは「誰が」「どの認証方式で」接続するかをAAP内に安全に保存する仕組みで、Playbookや接続先ホストのIPアドレスを直接見ても、パスワードやSSH秘密鍵の内容は画面上に表示されません。

## Section 1: CredentialとCredential Typeの考え方

AAPのCredentialは、**Credential Type** ごとに入力項目（スキーマ）が決まっています。代表的なCredential Typeは以下の通りです。

| Credential Type | 用途 | 本資料での使用箇所 |
|---|---|---|
| **Machine** | 管理対象ノードへのSSH接続（ユーザー名、SSH秘密鍵 or パスワード） | 本演習 |
| **Amazon Web Services** | EC2作成などAWS APIの呼び出し（アクセスキー） | 本演習の経路A。[操作対象サーバ（AWS）](../aws_targets.md) |
| **Source Control** | GitLab等のSCMリポジトリへの接続（ユーザー名、Personal Access Token） | [基礎演習4](../basic-controller/ex4.md) |
| **Vault** | ansible-vaultで暗号化されたデータの復号パスワード | [応用演習2](../advanced/ex2.md) |
| **Container Registry** | カスタムExecution Environmentイメージのpull/push | [応用演習6](../advanced/ex6.md) |

今回はまず、管理対象ノードへのSSH接続に使う **Machine Credential** を作成します。

## Section 2: Machine Credentialの作成

### Step 1: Credential作成画面を開く

**Automation Execution > Infrastructure > 認証情報** を開き、**認証情報の作成** をクリックします。

![認証情報の作成画面。タイプに Machine を選ぶとユーザ名、パスワード、SSH Private Key、Privilege Escalation Method が出る](../images/aap/credential-machine.png)

### Step 2: 基本情報を入力する

以下の項目を入力します。

| 項目 | 入力値 |
|---|---|
| 名前 | `managed-nodes-ssh` |
| 組織 | [演習1](./ex1.md)で確認した組織。**組織の選択** から選ぶ |
| 認証情報タイプ | `Machine`。一覧が長いので、選択欄の検索に `Machine` と入力する |

### Step 3: 認証方式を入力する

Credential Typeに `Machine` を選択すると、以下の入力項目が表示されます。[事前準備](../prepare_training_environment.md)で確認した管理対象ノードの認証方式に応じて、いずれかの方法で入力してください。

**方法A: SSH秘密鍵を使う場合**

| 項目 | 入力値 |
|---|---|
| ユーザ名 | 管理対象ノードへの接続ユーザー名（例: `ec2-user`） |
| SSH Private Key | 秘密鍵の内容を貼り付け（`-----BEGIN OPENSSH PRIVATE KEY-----` から `-----END OPENSSH PRIVATE KEY-----` まで全文） |
| Private Key Passphrase | 秘密鍵にパスフレーズが設定されている場合のみ入力 |

**方法B: パスワードを使う場合**

| 項目 | 入力値 |
|---|---|
| ユーザ名 | 管理対象ノードへの接続ユーザー名 |
| パスワード | 接続パスワード |

### Step 4: 権限昇格（Privilege Escalation）の設定

多くの場合、管理対象ノード上での作業にはroot権限（`sudo`）が必要です。Credential作成画面の「Privilege Escalation」セクションで以下を設定します。

| 項目 | 入力値 |
|---|---|
| Privilege Escalation Method | `sudo` |
| Privilege Escalation Username | 空欄（デフォルトの `root` を使う場合） |
| Privilege Escalation Password | `sudo` にパスワードが必要な場合のみ入力 |

**Note:** これはOSS版Ansibleの `become: true` / `become_method: sudo` に相当する設定です。Credential側で一度設定しておけば、このCredentialを使うすべてのJob Templateで自動的に適用されます。

### Step 5: 保存

**認証情報の作成** をクリックして保存します。

## Section 3: 作成したCredentialの確認

### Step 1: Credential一覧に表示されることを確認

**認証情報** の一覧に `managed-nodes-ssh` が表示されていることを確認してください。

### Step 2: 秘密情報が表示されないことを確認

作成したCredentialを再度開いてみてください。SSH秘密鍵やパスワードの欄は空欄（または `encrypted` 等の表示）になっており、入力した内容がそのまま表示されないことを確認してください。これはAAPがCredentialの機密情報を暗号化して保存しているためです。

**Note:** パスワードや秘密鍵を一度保存すると、画面上で内容を確認することはできません（再入力してWrite-onlyで更新するのみ）。これはセキュリティ上の仕様であり、不具合ではありません。

---

## Section 4: AWS用Credential（経路A）

EC2を新規作成する経路だけが、このCredentialを使います。既存サーバをインベントリに書く経路では、作らずに次の演習へ進んでください。

### Step 1: 作成画面を開く

**Automation Execution > Infrastructure > 認証情報** で **認証情報の作成** をクリックします。

### Step 2: 入力する

| 項目 | 入力値 |
|---|---|
| 名前 | `aws-training` |
| 組織 | [演習1](./ex1.md)で確認した組織 |
| 認証情報タイプ | `Amazon Web Services`。選択欄の検索に `Amazon` と入力する |
| Access Key | 演習用IAMユーザーのアクセスキー |
| Secret Key | 対になるシークレットアクセスキー |

STSを使う場合だけ、画面にあるトークン欄も入力します。アクセスキーをGitLabのPlaybookへは書きません。

**認証情報の作成** をクリックして保存します。保存後にSecret Keyが再表示されないことは、Machine Credentialと同じです。

このCredentialを使うJob Templateは [操作対象サーバ（AWS）](../aws_targets.md) と [基礎演習6](../basic-controller/ex6.md) で作ります。`amazon.aws` が既定のExecution Environmentに無い環境では、構築Jobは [応用演習6](../advanced/ex6.md) のあとで実行します。

---

ここまでで、管理対象ノードへ接続するためのCredentialが作成できました。次の演習では、このCredentialを使って実際にホストへ疎通確認を行うため、Inventoryを作成します。

[次へ進む](./ex3.md)
