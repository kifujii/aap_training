# 応用演習 4 - Workflow Job Template

この演習では、複数のJob Templateを連携させ、成功/失敗に応じて分岐させる **Workflow Job Template** を作成します。承認（Approval）ノードの使い方もあわせて扱います。

**前提条件:** 応用演習1〜3を完了していること

[前に戻る](./ex3.md) | [次へ進む](./ex5.md)

---

## Section 1: 確認用Job Templateの作成

Workflowの中で「デプロイ後の確認」を行うため、[sample-content/verify.yml](../sample-content/verify.yml)を実行する新しいJob Templateを作成します。

### Step 1: Job Templateを作成する

**Automation Execution > Templates** →「Add」→「Add job template」で、以下を入力します。

| 項目 | 入力値 |
|---|---|
| Name | `verify-deployment` |
| Job Type | `Run` |
| Inventory | `training-inventory` |
| Project | `aap-training-content` |
| Playbook | `verify.yml` |
| Credentials | `managed-nodes-ssh` |

「Save」をクリックして保存します。単体で一度「Launch」し、正常終了することを確認しておきましょう。

## Section 2: Workflow Job Templateの作成

### Step 1: Workflow Job Template作成画面を開く

**Automation Execution > Templates** →「Add」→「Add workflow template」をクリックします。

### Step 2: 入力する

| 項目 | 入力値 |
|---|---|
| Name | `web-stack-workflow` |
| Organization | 基礎演習1で確認したOrganization |

「Save」をクリックすると、Workflow Visualizer（ノードを配置するグラフィカルな編集画面）が開きます。

## Section 3: ノードの配置

### Step 1: 開始ノードを追加する

「Start」ノードの「+」をクリックし、以下を設定します。

| 項目 | 入力値 |
|---|---|
| Node Type | `Job Template` |
| Job Template | `deploy-web-stack` |

### Step 2: 成功時のApprovalノードを追加する

`deploy-web-stack` ノードの右側（または下側）の「+」から、「On Success」の分岐でノードを追加します。

| 項目 | 入力値 |
|---|---|
| Node Type | `Approval` |
| Name | `本番反映前の承認` |
| Description | デプロイ内容を確認し、問題なければ承認してください |
| Timeout | `1` hour（任意） |

### Step 3: 承認後のノードを追加する

Approvalノードの「On Success」（承認された場合）の分岐で、以下のノードを追加します。

| 項目 | 入力値 |
|---|---|
| Node Type | `Job Template` |
| Job Template | `verify-deployment` |

### Step 4: 保存する

画面右上の「Save」をクリックしてWorkflowを保存します。以下のような構造になっていることを確認してください。

```
deploy-web-stack --(成功)--> [承認: 本番反映前の承認] --(承認)--> verify-deployment
                 \
                  --(失敗)--> （何も繋がない。応用演習3のError通知で検知する）
```

## Section 4: Workflowの実行

### Step 1: Launchする

`web-stack-workflow` を「Launch」します。

### Step 2: 承認待ち状態を確認する

`deploy-web-stack` ノードが成功すると、Approvalノードが「Pending」状態になり、Workflowの実行が一時停止します。

### Step 3: 承認する

Workflow実行画面（または **Automation Execution > Jobs** の該当ジョブ）から、Approvalノードの詳細を開き、「Approve」をクリックします。

### Step 4: 残りのノードが実行されることを確認する

承認後、`verify-deployment` ノードが自動的に実行され、Workflow全体が `Successful` のステータスで完了することを確認してください。

### Step 5: 拒否（Deny）した場合の動作を確認する（任意）

もう一度Workflowを実行し、今回はApprovalノードで「Deny」を選択してみましょう。Workflow全体が `Failed` として終了し、`verify-deployment` ノードは実行されないことを確認してください。

**Note:** Approvalノードを拒否・タイムアウトさせた場合の動きは、Workflow全体を失敗として扱うか、別の分岐（例: ロールバック用のJob Template）に繋ぐかを設計できます。実運用では、「失敗時のロールバック」や「承認者への再通知」を別ノードとして追加することも検討してください。

## Section 5: まとめ

| 要素 | 役割 |
|---|---|
| Job Templateノード | 既存のJob Templateをそのままノードとして呼び出す |
| Approvalノード | 人による承認を挟む。`Approve` ロールを持つUser/Teamのみ承認可能 |
| On Success / On Failure / Always分岐 | 前段ノードの結果に応じて、後続の実行経路を切り替える |

Workflow Job Templateを使うことで、「デプロイ→承認→検証」のような、複数ステップにわたる運用フローをAAP上で一元管理できます。[宿題 課題3](../homework/hm1.md)でも、この仕組みを活用します。

---

ここまでで、Workflow Job Templateの基本的な使い方を習得しました。次の演習では、Job実行の裏側で使われているExecution Environmentについて学びます。

[次へ進む](./ex5.md)
