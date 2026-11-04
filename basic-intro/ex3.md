# 演習 3 - Inventoryとホスト/グループの作成、疎通確認

[前に戻る](./ex2.md)

---

この演習では、AAPの管理対象ホストを定義する **Inventory** を作成し、[演習2](./ex2.md)で作成したCredentialを使って実際にホストへ疎通確認を行います。

今回はホスト情報を手動で入力する「静的Inventory」の方法を扱います。GitLab上のYAMLファイルから自動的にホスト情報を取り込む方法（Inventory Source）は[基礎演習5](../basic-controller/ex5.md)で扱います。

## Section 1: Inventoryの作成

### Step 1: Inventory作成画面を開く

**Automation Execution > Infrastructure > インベントリー** を開き、**インベントリーの作成** のドロップダウンから、もう一度 **インベントリーの作成** を選びます。スマートインベントリーや構築済みインベントリーは選びません。

![インベントリーの作成画面。名前と組織が必須](../images/aap/inventory-create.png)

### Step 2: 基本情報を入力する

| 項目 | 入力値 |
|---|---|
| 名前 | `training-inventory` |
| 組織 | [演習1](./ex1.md)で確認した組織。この項目は必須です |

**インベントリーの作成** をクリックして保存します。

## Section 2: Groupの作成

作成した `training-inventory` の詳細画面に移動すると、**グループ** タブがあります。操作対象は `managed`、EC2作成Job専用の実行元は `automation` です。サーバの用意は [操作対象サーバ（AWS）](../aws_targets.md) の経路Aまたは経路Bです。

### Step 1: managedグループの作成

**グループ** タブからグループの作成を開き、以下を入力します。

| 項目 | 入力値 |
|---|---|
| Name | `managed` |

### Step 2: automationグループの作成

同様に、以下を入力してもう1つGroupを作成します。

| 項目 | 入力値 |
|---|---|
| Name | `automation` |

**Note:** OSS版Ansibleのインベントリにおける `children:` のグループ分けと同じ考え方です。`site.yml` は `hosts: managed` だけを変更します。`automation` は `provision_ec2.yml` が `localhost` でAWS APIを呼ぶためのグループです。

## Section 3: Hostの追加

### Step 1: managedグループへホストを追加する

`managed` グループの詳細画面を開き、**ホスト** タブからホストの作成を開きます。

**経路B（既存サーバ）:** 実サーバのIPまたはDNS名を `ansible_host` に書きます。台数が1台なら `srv1` だけで構いません。

**経路A（これからEC2を作る）:** プライベートIPは [基礎演習6](../basic-controller/ex6.md) の構築Jobのあとで分かります。この時点ではホスト名だけ登録し、`ansible_host` は仮の値で構いません。疎通確認は構築Jobのあとに行います。

| Host名 | Variables (YAML) |
|---|---|
| `srv1` | `ansible_host: 10.0.1.11` |
| `srv2` | `ansible_host: 10.0.1.12` |

Variablesの入力欄には、以下のようなYAML形式で入力します。アドレスは環境の値に置き換えてください。

```yaml
ansible_host: 10.0.1.11
```

### Step 2: automationグループへホストを追加する

経路AでEC2を作る場合だけ、`automation` グループに次のホストを追加します。経路Bでは不要です。

| Host名 | Variables (YAML) |
|---|---|
| `localhost` | `ansible_connection: local` |

```yaml
ansible_connection: local
```

**Note:** `ansible_host` は、AAPがSSHするときの接続先です。後の演習では、この値を `hostvars` から参照して、管理対象の一覧ファイルを作ります（[sample-content/roles/baseline/templates/peers.txt.j2](../sample-content/roles/baseline/templates/peers.txt.j2)）。OSS版Ansibleのホスト変数と同じ考え方です。

## Section 4: Ad Hoc Commandによる疎通確認

Inventoryとホストの登録が終わったら、Playbookを作らずにモジュールを1回だけ実行する **Ad Hoc Command** 機能で疎通確認を行います。OSS版Ansibleの `ansible all -m ping` に相当する操作です。

### Step 1: Run Commandを起動する

`training-inventory` の **ホスト** タブで、対象ホストにチェックを入れ、**Run command** をクリックします。

### Step 2: 実行内容を入力する

| 項目 | 入力値 |
|---|---|
| Module | `ping` |
| Arguments | （空欄でよい） |
| Limit | 対象ホストが自動的に設定されていることを確認 |
| Credential | [演習2](./ex2.md)で作成した `managed-nodes-ssh` を選択 |

最後の実行ボタン（**Launch** または **Next** のあとに出る実行）をクリックします。

### Step 3: 実行結果を確認する

実行が完了すると、Job詳細画面に以下のような結果が表示されます。

```
srv1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
srv2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

`managed` のホストが `SUCCESS` と表示されれば、CredentialとInventoryの設定が正しく、AAPから管理対象ノードへ接続できることが確認できました。経路Aでインスタンスがまだ無い場合は、ここを飛ばして [基礎演習6](../basic-controller/ex6.md) の構築Jobのあとで戻ってきてください。`localhost` はSSHしないので、この確認の対象にしません。

### Step 4: グループ単位での疎通確認

同様の手順で、「Limit」を `managed` グループのみに絞って再実行してみましょう。`srv1` と `srv2` が対象になることを確認してください。

## Section 5: トラブルシューティング

Ad Hoc Commandが失敗する場合は、以下を確認してください。

* Host変数の `ansible_host`（IPアドレス）が正しいか
* Credentialのユーザー名・パスワード（またはSSH秘密鍵）が正しいか
* AAPの実行環境（Execution Node）から管理対象ノードへネットワーク的に到達可能か（ファイアウォール、セキュリティグループ等）
* 管理対象ノードのSSHサービスが起動しているか
* Jobの詳細ログ（Output）にエラーメッセージが出ていないか

---

ここまでで、AAPの基本操作（ログイン、Credential、Inventory、疎通確認）が完了しました。次は、GitLabと連携したProjectの作成に進みます。

[基礎演習（Project・Job Template）へ進む](../basic-controller/ex4.md)
