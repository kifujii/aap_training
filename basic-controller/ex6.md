# 演習 6 - 初めてのJob Templateの作成と実行

[前に戻る](./ex5.md)

---

この演習では、これまでに作成した **Credential**（[基礎演習2](../basic-intro/ex2.md)）、**Inventory**（[基礎演習3](../basic-intro/ex3.md)・[5](./ex5.md)）、**Project**（[基礎演習4](./ex4.md)）を組み合わせて、初めての **Job Template** を作成・実行します。

経路A（EC2を新規作成）は Section 1 でサーバを作り、Section 2 以降で初期設定を適用します。経路B（既存サーバ）は Section 1 を飛ばして Section 2 から実行します。変数と到達性の条件は [操作対象サーバ（AWS）](../aws_targets.md) を参照してください。

## Section 1: EC2の作成（経路Aのみ）

### Step 1: 変数を実環境の値にする

`provision_ec2.yml` は、`ec2_vpc_id`、`ec2_subnet_id`、`ec2_ami_id` が `vpc-xxxxxxxx` のようなプレースホルダのままだと停止します。Job Templateの **Extra variables** に、次の形で実値を書きます。AMI IDはコンソールのAMIカタログで確認し、資料の例をそのまま使いません。

```yaml
ec2_region: ap-northeast-1
ec2_vpc_id: vpc-0123456789abcdef0
ec2_subnet_id: subnet-0123456789abcdef0
ec2_ami_id: ami-0123456789abcdef0
ec2_key_name: aap-training
ec2_ssh_ingress_cidr: 10.0.0.0/16
```

`ec2_key_name` は、Machine Credentialに登録した秘密鍵と対になるキーペア名です。`ec2_ssh_ingress_cidr` は、AAPのExecution NodeがSSHしてくる範囲です。

### Step 2: Job Templateを作る

**Automation Execution > テンプレート** を開き、**テンプレートの作成** > **ジョブテンプレートの作成** をクリックします。

| 項目 | 入力値 |
|---|---|
| 名前 | `provision-ec2` |
| ジョブタイプ | `実行` |
| インベントリー | `training-inventory` |
| プロジェクト | `aap-training-content` |
| Playbook | `provision_ec2.yml` |
| 認証情報 | `aws-training`（Amazon Web Services） |
| 実行環境 | 空欄（既定）のまま。AAP 2.7 の既定EEには `amazon.aws` 10.3.0 が入っています |
| Extra variables | Step 1のYAML |

Machine Credentialは割り当てません。Playの対象は `localhost` で、接続は `local` です。

### Step 3: 実行してIPをインベントリへ戻す

**テンプレートの起動** を実行します。成功すると、各インスタンスのインスタンスIDとプライベートIPがログに出ます。

1. そのIPをGitLab上の `inventory.yml` の `srv1` / `srv2` の `ansible_host` に書く
2. pushする
3. [基礎演習5](./ex5.md) のInventory Sourceを再同期する
4. [基礎演習3](../basic-intro/ex3.md) の **Run command**（モジュール `ping`）で `managed` に届くことを確認する

同じJobを再実行しても、同じ `name` のインスタンスは作り直されません。

## Section 2: 初期設定Job Templateの作成

### Step 1: Job Template作成画面を開く

**Automation Execution > テンプレート** を開き、**テンプレートの作成** > **ジョブテンプレートの作成** をクリックします。

![ジョブテンプレートの作成。ジョブタイプの既定は「実行」で、インベントリー、プロジェクト、Playbook が必須](../images/aap/job-template-create.png)

### Step 2: 入力する

| 項目 | 入力値 |
|---|---|
| 名前 | `apply-baseline` |
| ジョブタイプ | `実行`（英語UIでは Run） |
| インベントリー | `training-inventory` |
| プロジェクト | `aap-training-content` |
| Playbook | `site.yml`（Projectが同期済みであればドロップダウンに表示される） |
| 認証情報 | `managed-nodes-ssh`（Machine Credential） |

他の項目（Extra variables、制限、詳細）は、いったんデフォルトのままで構いません。これらは[演習7](./ex7.md)で扱います。

**ジョブテンプレートの作成** をクリックして保存します。

## Section 3: Job Templateの実行

### Step 1: Launchする

Job Template一覧、または詳細画面から、`apply-baseline` の **テンプレートの起動** をクリックします。詳細画面の見え方は [演習1のスクリーンショット](../basic-intro/ex1.md) と同じ配置です。

### Step 2: 実行結果を確認する

Jobの実行画面に遷移し、リアルタイムで実行ログが表示されます。[sample-content/site.yml](../sample-content/site.yml)の内容に沿って、以下のようなPLAY RECAPが表示されれば成功です。

```
PLAY [Apply Linux baseline] ****************************************************

TASK [Gathering Facts] *********************************************************
ok: [srv1]
ok: [srv2]

TASK [baseline : Set the timezone] *********************************************
changed: [srv1]
changed: [srv2]

TASK [baseline : Install baseline packages] ************************************
changed: [srv1]
changed: [srv2]

TASK [baseline : Enable chronyd] ***********************************************
changed: [srv1]
changed: [srv2]

TASK [baseline : Write the login banner] ***************************************
changed: [srv1]
changed: [srv2]

TASK [baseline : Write the list of managed peers] ******************************
changed: [srv1]
changed: [srv2]

TASK [access : Configure sshd limits] ******************************************
changed: [srv1]
changed: [srv2]

RUNNING HANDLER [access : Reload sshd] *****************************************
changed: [srv1]
changed: [srv2]

PLAY RECAP *********************************************************************
srv1   : ok=13  changed=7  unreachable=0  failed=0  skipped=2  rescued=0  ignored=0
srv2   : ok=13  changed=7  unreachable=0  failed=0  skipped=2  rescued=0  ignored=0
```

`skipped` は、ローカル管理ユーザーを作るタスクです。Vaultをまだ使っていないためスキップされます（[応用演習2](../advanced/ex2.md)）。`ok` と `changed` の数は、すでに入っているパッケージによって前後します。`failed=0` かつ `unreachable=0` なら成功です。

`localhost` は PLAY RECAPに出ません。Playbookの対象が `managed` だからです。

### Step 3: Detailsタブを確認する

Job実行画面の「Details」タブで、以下の情報を確認してください。

| 項目 | 内容 |
|---|---|
| Status | `Successful` |
| Template | `apply-baseline` |
| Inventory | `training-inventory` |
| Project / Revision | 実行時にAAPが取得したGitLabのコミットハッシュ |
| Credentials | `managed-nodes-ssh` |
| Started / Finished | 実行開始・終了時刻 |

**Note:** Revisionが表示されることで、「どのコミット時点のPlaybookでこのJobが実行されたか」を後から追跡できます。[基礎演習4](./ex4.md)で `Update Revision on Launch` を有効にしているため、GitLab側のリポジトリを更新すれば、次回Launch時に自動的に最新コミットが使われます。

### Step 4: 再実行して冪等性を確認する

もう一度 **テンプレートの起動** をクリックし、同じJob Templateを再実行してみましょう。PLAY RECAPの `changed` が `0` になることを確認してください。すでに目的のタイムゾーンになっているホストでは、タイムゾーンを設定するタスクは `skipped` のままです。OSS版Ansibleと同じ「冪等性」の原則がAAP上でも適用されます。

## Section 4: Check実行（ドライラン）

### Step 1: Job Templateを編集する

`apply-baseline` の編集画面を開き、「Job Type」を `Check` に変更して保存します。

### Step 2: 実行する

再度 **テンプレートの起動** をクリックします。実際にはホストへ変更を加えず、変更が発生するかどうかだけをシミュレーションします。OSS版Ansibleの `--check` オプションに相当します。

### Step 3: 設定を元に戻す

確認が終わったら、「Job Type」を `Run` に戻しておきましょう（以降の演習で実際に変更を反映させるため）。

**Note:** `command` モジュールは、Checkモードでは実際のコマンドを実行しません。変更有無の表示は、通常実行のときと違うことがあります。確認が終わったらジョブタイプを `実行` に戻します。

---

ここまでで、Job Templateの作成・実行・Check実行ができるようになりました。次の演習では、Extra VariablesやSurveyなど、Job Templateのより詳細な設定を扱います。

[次へ進む](./ex7.md)
