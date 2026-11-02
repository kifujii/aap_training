# 演習 6 - 初めてのJob Templateの作成と実行

[前に戻る](./ex5.md)

---

この演習では、これまでに作成した **Credential**（[基礎演習2](../basic-intro/ex2.md)）、**Inventory**（[基礎演習3](../basic-intro/ex3.md)・[5](./ex5.md)）、**Project**（[基礎演習4](./ex4.md)）を組み合わせて、初めての **Job Template** を作成・実行します。

## Section 1: Job Templateの作成

### Step 1: Job Template作成画面を開く

**Automation Execution > Templates** を開き、「Add」→「Add job template」をクリックします。

### Step 2: 入力する

| 項目 | 入力値 |
|---|---|
| Name | `deploy-web-stack` |
| Job Type | `Run` |
| Inventory | `training-inventory` |
| Project | `aap-training-content` |
| Playbook | `site.yml`（Projectが同期済みであればドロップダウンに表示される） |
| Credentials | `managed-nodes-ssh`（Machine Credential） |

他の項目（Extra Variables、Limit、Verbosityなど）は、いったんデフォルトのままで構いません。これらは[演習7](./ex7.md)で扱います。

「Save」をクリックして保存します。

## Section 2: Job Templateの実行

### Step 1: Launchする

Job Template一覧（または詳細画面）から、`deploy-web-stack` の「Launch」（ロケットアイコン）をクリックします。

### Step 2: 実行結果を確認する

Jobの実行画面に遷移し、リアルタイムで実行ログが表示されます。[sample-content/site.yml](../sample-content/site.yml)の内容に沿って、以下のようなPLAY RECAPが表示されれば成功です。

```
PLAY [Deploy nginx web servers] ***********************************************

TASK [Gathering Facts] *********************************************************
ok: [web1]
ok: [web2]
ok: [web3]

TASK [nginx : install nginx packages] ******************************************
changed: [web1]
changed: [web2]
changed: [web3]

TASK [nginx : copy nginx.conf] **************************************************
changed: [web1]
changed: [web2]
changed: [web3]

TASK [nginx : copy index.html] **************************************************
changed: [web1]
changed: [web2]
changed: [web3]

TASK [nginx : start and enable nginx] *******************************************
changed: [web1]
changed: [web2]
changed: [web3]

PLAY [Deploy haproxy load balancer] *********************************************

TASK [Gathering Facts] *********************************************************
ok: [lb]

TASK [haproxy : install haproxy package] ****************************************
changed: [lb]

TASK [haproxy : copy haproxy.cfg] ************************************************
changed: [lb]

TASK [haproxy : start and enable haproxy] ****************************************
changed: [lb]

PLAY RECAP ***********************************************************************
web1   : ok=5  changed=4  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0
web2   : ok=5  changed=4  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0
web3   : ok=5  changed=4  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0
lb     : ok=4  changed=3  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0

Job finished with status: successful
```

### Step 3: Detailsタブを確認する

Job実行画面の「Details」タブで、以下の情報を確認してください。

| 項目 | 内容 |
|---|---|
| Status | `Successful` |
| Template | `deploy-web-stack` |
| Inventory | `training-inventory` |
| Project / Revision | 実行時にAAPが取得したGitLabのコミットハッシュ |
| Credentials | `managed-nodes-ssh` |
| Started / Finished | 実行開始・終了時刻 |

**Note:** Revisionが表示されることで、「どのコミット時点のPlaybookでこのJobが実行されたか」を後から追跡できます。[基礎演習4](./ex4.md)で `Update Revision on Launch` を有効にしているため、GitLab側のリポジトリを更新すれば、次回Launch時に自動的に最新コミットが使われます。

### Step 4: 再実行して冪等性を確認する

もう一度「Launch」をクリックし、同じJob Templateを再実行してみましょう。すべてのタスクが `changed` ではなく `ok`（変更なし）になることを確認してください。OSS版Ansibleと同じ「冪等性」の原則がAAP上でも適用されます。

```
PLAY RECAP ***********************************************************************
web1   : ok=5  changed=0  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0
web2   : ok=5  changed=0  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0
web3   : ok=5  changed=0  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0
lb     : ok=4  changed=0  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0
```

## Section 3: Check実行（ドライラン）

### Step 1: Job Templateを編集する

`deploy-web-stack` の編集画面を開き、「Job Type」を `Check` に変更して保存します。

### Step 2: 実行する

再度「Launch」をクリックします。実際にはホストへ変更を加えず、変更が発生するかどうかだけをシミュレーションします。OSS版Ansibleの `--check` オプションに相当します。

### Step 3: 設定を元に戻す

確認が終わったら、「Job Type」を `Run` に戻しておきましょう（以降の演習で実際に変更を反映させるため）。

**Note:** `uri` モジュールなど一部のモジュールはCheckモードに非対応です。[宿題 課題3](../homework/hm1.md)でこの点に注意が必要になります。

---

ここまでで、Job Templateの作成・実行・Check実行ができるようになりました。次の演習では、Extra VariablesやSurveyなど、Job Templateのより詳細な設定を扱います。

[次へ進む](./ex7.md)
