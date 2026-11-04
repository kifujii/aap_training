# 演習 5 - Inventory Source（SCM経由のYAMLインベントリ）

[前に戻る](./ex4.md)

---

この演習では、[基礎演習3](../basic-intro/ex3.md)で手動登録したInventoryに代えて、GitLab上のYAML形式インベントリファイル（`sample-content/inventory.yml`）から自動的にホスト情報を取り込む **Inventory Source** を設定します。

## Section 1: Inventory Sourceの考え方

| 方式 | 概要 | 向いているケース |
|---|---|---|
| **手動登録**（基礎演習3） | AAPの画面からHost/Groupを1件ずつ入力する | 台数が少ない、頻繁に変わらない環境 |
| **Inventory Source: Sourced from a Project** | SCM上のYAML/INIインベントリファイルをAAPが読み込み、Host/Groupを自動的に構築する | OSS版Ansibleのインベントリをそのまま使い回したい、台数が多い、Gitで変更履歴を管理したい |

Inventory SourceはSCM以外にも、クラウドAPIからホストを取得する方式があります。EC2のタグで取り込む手順は [操作対象サーバ（AWS）](../aws_targets.md) の任意節です。この演習は、GitLab上のYAMLを読む **Sourced from a Project** を作ります。経路Bの既存サーバも、EC2作成後にIPを書き戻した経路Aも、ここは同じです。

## Section 2: Inventory Sourceの作成

### Step 1: Inventoryの詳細画面を開く

**Automation Execution > Infrastructure > インベントリー** から、[基礎演習3](../basic-intro/ex3.md)で作成した `training-inventory` を開きます。

### Step 2: ソースタブで新規Sourceを追加する

**ソース** タブからソースの追加を開きます。

### Step 3: 入力する

| 項目 | 入力値 |
|---|---|
| Name | `gitlab-inventory-source` |
| Source | `Sourced from a Project` |
| Project | [基礎演習4](./ex4.md)で作成した `aap-training-content` |
| Inventory File | `inventory.yml`（Projectの同期後にドロップダウンから選択可能） |
| Update Options | `Overwrite`、`Overwrite Variables`、`Update on Launch` を有効化 |

**Note:** `Overwrite` を有効にすると、Sync時にSource由来でないHost/Groupが削除されます。[基礎演習3](../basic-intro/ex3.md)で手動登録したホストは、`inventory.yml` に同名で書いてあれば同期後も残ります。YAMLに無いホストは消えます。経路AでまだプライベートIPを書き戻していない場合は、サンプルの `10.0.1.11` などの仮アドレスのまま同期されます。構築Jobのあとで `ansible_host` を直して、もう一度同期してください。

画面下部の作成ボタンをクリックして保存します。項目名が英語のままの環境では、Source は `Sourced from a Project`、更新オプションは `Overwrite` / `Overwrite variables` / `Update on launch` です。

### Step 4: Syncの実行

作成した `gitlab-inventory-source` の行にある同期アイコンをクリックし、Syncを実行します。

## Section 3: 結果の確認

### Step 1: Host一覧を確認する

`training-inventory` の「Hosts」タブを開き、以下のホストが存在することを確認してください。

* `srv1`（`ansible_host` は `inventory.yml` に書いたアドレス）
* `srv2`（同様）
* `localhost`（`ansible_connection: local`。`provision_ec2.yml` 用）

### Step 2: Group一覧を確認する

「Groups」タブで `managed` と `automation` が、`inventory.yml` の `children:` 定義通りに構成されていることを確認してください。

### Step 3: ホスト変数を確認する

`srv1` の詳細を開き、「Variables」の `ansible_host` が、`inventory.yml` に記載した値と一致していることを確認してください。

**Note:** これらはOSS版Ansibleのホスト変数とまったく同じ扱いです。[基礎演習6](./ex6.md)でPlaybook実行時にこれらの変数が使われます。

## Section 4: 以後の演習での扱い

以降の演習（[基礎演習6](./ex6.md)以降）では、特に断りがない限り、この `training-inventory`（Inventory Source経由で同期済み）を使用します。管理対象ノードの構成を変更したい場合は、AAPの画面から直接編集するのではなく、GitLab上の `inventory.yml` を編集してpushし、Inventory SourceのSyncを再実行する運用に統一してください（[宿題 課題2](../homework/hm1.md)で実際に体験します）。

---

ここまでで、GitLab上のインベントリファイルをAAPに取り込む設定が完了しました。次の演習では、いよいよPlaybookを実行するJob Templateを作成します。

[次へ進む](./ex6.md)
