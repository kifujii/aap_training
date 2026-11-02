# 演習 5 - Inventory Source（SCM経由のYAMLインベントリ）

[前に戻る](./ex4.md)

---

この演習では、[基礎演習3](../basic-intro/ex3.md)で手動登録したInventoryに代えて、GitLab上のYAML形式インベントリファイル（`sample-content/inventory.yml`）から自動的にホスト情報を取り込む **Inventory Source** を設定します。

## Section 1: Inventory Sourceの考え方

| 方式 | 概要 | 向いているケース |
|---|---|---|
| **手動登録**（基礎演習3） | AAPの画面からHost/Groupを1件ずつ入力する | 台数が少ない、頻繁に変わらない環境 |
| **Inventory Source: Sourced from a Project** | SCM上のYAML/INIインベントリファイルをAAPが読み込み、Host/Groupを自動的に構築する | OSS版Ansibleのインベントリをそのまま使い回したい、台数が多い、Gitで変更履歴を管理したい |

Inventory SourceはSCM以外にも、クラウドAPI（AWS/Azure/GCP等）から動的にホストを取得する方式もありますが、本資料ではGitLab連携を前提に **Sourced from a Project** のみを扱います。

## Section 2: Inventory Sourceの作成

### Step 1: Inventoryの詳細画面を開く

**Automation Execution > Inventories** から、[基礎演習3](../basic-intro/ex3.md)で作成した `training-inventory` を開きます。

### Step 2: Sourcesタブで新規Sourceを追加する

「Sources」タブ → 「Add」をクリックします。

### Step 3: 入力する

| 項目 | 入力値 |
|---|---|
| Name | `gitlab-inventory-source` |
| Source | `Sourced from a Project` |
| Project | [基礎演習4](./ex4.md)で作成した `aap-training-content` |
| Inventory File | `inventory.yml`（Projectの同期後にドロップダウンから選択可能） |
| Update Options | `Overwrite`、`Overwrite Variables`、`Update on Launch` を有効化 |

**Note:** `Overwrite` を有効にすると、Sync時にSource由来でないHost/Groupが削除されます。[基礎演習3](../basic-intro/ex3.md)で手動登録したHost（`web1`, `web2`, `lb`）は `inventory.yml` にも同名で定義されているため、Sync後は自動的にSource由来の情報（`web3` を含む）に揃います。

「Save」をクリックして保存します。

### Step 4: Syncの実行

作成した `gitlab-inventory-source` の行にある同期アイコンをクリックし、Syncを実行します。

## Section 3: 結果の確認

### Step 1: Host一覧を確認する

`training-inventory` の「Hosts」タブを開き、以下のホストが存在することを確認してください。

* `web1`（`ansible_host: 192.0.2.11`, `nginx_listen_port: 8080`）
* `web2`（`ansible_host: 192.0.2.12`, `nginx_listen_port: 8080`）
* `web3`（`ansible_host: 192.0.2.13`, `nginx_listen_port: 8080`）※ `sample-content/inventory.yml` に新たに定義されているホスト
* `lb`（`ansible_host: 192.0.2.14`）

### Step 2: Group一覧を確認する

「Groups」タブで `web`・`loadbalancer` の2グループが、`inventory.yml` の `children:` 定義通りに構成されていることを確認してください。

### Step 3: ホスト変数を確認する

`web1` の詳細を開き、「Variables」に `ansible_host` や `nginx_listen_port` が、`inventory.yml` に記載した値と一致していることを確認してください。

**Note:** これらはOSS版Ansibleのホスト変数とまったく同じ扱いです。[基礎演習6](./ex6.md)でPlaybook実行時にこれらの変数が使われます。

## Section 4: 以後の演習での扱い

以降の演習（[基礎演習6](./ex6.md)以降）では、特に断りがない限り、この `training-inventory`（Inventory Source経由で同期済み）を使用します。管理対象ノードの構成を変更したい場合は、AAPの画面から直接編集するのではなく、GitLab上の `inventory.yml` を編集してpushし、Inventory SourceのSyncを再実行する運用に統一してください（[宿題 課題2](../homework/hm1.md)で実際に体験します）。

---

ここまでで、GitLab上のインベントリファイルをAAPに取り込む設定が完了しました。次の演習では、いよいよPlaybookを実行するJob Templateを作成します。

[次へ進む](./ex6.md)
