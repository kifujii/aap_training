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

作成した `training-inventory` の詳細画面に移動すると、**グループ** タブがあります。[事前準備](../prepare_training_environment.md)で確認した管理対象ノードの構成に合わせて、2つのグループを作成します。管理対象ノードがまだない場合は、到達確認できるホストが1台でもあれば、そのホストだけで画面操作の確認を先に進めてください。`web1` などの名前は例です。

### Step 1: webグループの作成

**グループ** タブからグループの作成を開き、以下を入力します。

| 項目 | 入力値 |
|---|---|
| Name | `web` |

### Step 2: loadbalancerグループの作成

同様に、以下を入力してもう1つGroupを作成します。

| 項目 | 入力値 |
|---|---|
| Name | `loadbalancer` |

**Note:** OSS版Ansibleのインベントリにおける `children:` のグループ分けと同じ考え方です。グループ単位でJob Templateの実行対象を指定したり、Playbook内の `hosts: web` で参照したりできます。

## Section 3: Hostの追加

### Step 1: webグループへホストを追加する

`web` グループの詳細画面を開き、**ホスト** タブからホストの作成を開きます。[事前準備](../prepare_training_environment.md)で確認した実際のホスト名・IPアドレスに置き換えて、以下のように追加してください。

| Host名 | Variables (YAML) |
|---|---|
| `web1` | `ansible_host: 192.0.2.11`<br>`nginx_listen_port: 8080` |
| `web2` | `ansible_host: 192.0.2.12`<br>`nginx_listen_port: 8080` |

Variablesの入力欄には、以下のようなYAML形式で入力します。

```yaml
ansible_host: 192.0.2.11
nginx_listen_port: 8080
```

### Step 2: loadbalancerグループへホストを追加する

同様に、`loadbalancer` グループに以下のホストを追加します。

| Host名 | Variables (YAML) |
|---|---|
| `lb` | `ansible_host: 192.0.2.14` |

**Note:** `nginx_listen_port` のようなホスト変数は、後の演習（[基礎演習6](../basic-controller/ex6.md)、[応用演習4](../advanced/ex4.md)）でPlaybook/テンプレートから `hostvars` 経由で参照します。OSS版Ansibleのホスト変数と同じ考え方です。

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
web1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
web2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
lb | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

すべてのホストが `SUCCESS` と表示されれば、CredentialとInventoryの設定が正しく、AAPから管理対象ノードへ接続できることが確認できました。

### Step 4: グループ単位での疎通確認

同様の手順で、「Limit」を `web` グループのみに絞って再実行してみましょう。`web1`・`web2` のみが対象になることを確認してください。

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
