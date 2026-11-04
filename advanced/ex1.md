# 応用演習 1 - RBACとアクセス制御

この演習では、AAPのRBAC（Role-Based Access Control）の基本的な考え方と、Team・Userへのロール付与の実践方法を学びます。

**前提条件:** 基礎演習1〜7を完了していること

[基礎演習に戻る](../basic-controller/ex7.md) | [次へ進む](./ex2.md)

---

## Section 1: RBACの基本概念

AAPでは、UserまたはTeamに対して「どのオブジェクトに」「どのロールを」付与するかを設定することで、アクセス制御を行います。

### ビルトインロール一覧（代表例）

| ロール | 付与対象オブジェクト | 権限の概要 |
|---|---|---|
| **System Administrator** | システム全体 | すべての操作が可能（最上位） |
| **System Auditor** | システム全体 | すべてのオブジェクトを閲覧可能（変更不可） |
| **Organization Admin** | Organization | 当該Organization配下のすべてのオブジェクトを管理可能 |
| **Organization Member** | Organization | 当該Organizationに所属している状態（それ自体は操作権限を持たない） |
| **Admin** | Project / Inventory / Job Template等 | 当該オブジェクトの編集・削除・権限管理が可能 |
| **Execute** | Job Template / Workflow Job Template | 当該テンプレートの実行（Launch）のみ可能。編集不可 |
| **Use** | Project / Inventory / Credential | 当該オブジェクトをJob Template作成時に選択・利用できる（直接の編集は不可） |
| **Read** | 各種オブジェクト | 閲覧のみ可能 |
| **Approve** | Workflow Job Template | Workflow内のApprovalノードを承認/拒否できる |
| **Auditor** | Organization等 | 配下オブジェクトを閲覧可能（変更不可） |

**Note:** オブジェクトの種類によって付与可能なロールの種類は異なります。例えばCredentialには `Admin` / `Use` 程度しかありませんが、Job Templateには `Admin` / `Execute` / `Read` など、より細かい区分があります。

## Section 2: Teamの作成

研修用の例として、「Job Templateの実行だけを許可された運用担当チーム」を作成してみましょう。

### Step 1: Team作成画面を開く

**Access Management > チーム** を開き、チームの作成を開始します。

### Step 2: 入力する

| 項目 | 入力値 |
|---|---|
| Name | `web-ops-team` |
| Organization | 基礎演習1で確認したOrganization |

「Save」をクリックして保存します。

### Step 3: Teamにユーザーを追加する

作成した `web-ops-team` の詳細画面 →「Users」タブ →「Add」をクリックし、研修用に用意された別のユーザー（存在しない場合は自分自身でも構いません）を追加します。

## Section 3: Teamへのロール付与

### Step 1: Job Templateへの実行権限を付与する

`apply-baseline` Job Templateの詳細画面 →「Access」（または「Team Access」）タブ →「Add」をクリックします。

| 項目 | 入力値 |
|---|---|
| Team | `web-ops-team` |
| Role | `Execute` |

「Save」をクリックします。

### Step 2: Projectへの閲覧権限を付与する

同様に `aap-training-content` Projectの「Access」タブで、`web-ops-team` に `Read` ロールを付与します。

**Note:** Job Templateを実行するだけであれば、紐付くProject/Inventory/Credentialの `Admin` 権限までは必要ありません。必要最小限の権限（Execute / Read / Use）を組み合わせるのが基本原則です。

## Section 4: 動作確認

### Step 1: Team所属ユーザーでログインする

`web-ops-team` に追加したユーザーでAAPにログインします（別ブラウザ・シークレットウィンドウの利用を推奨）。

### Step 2: 権限の範囲を確認する

* **Automation Execution > テンプレート** に `apply-baseline` が表示され、**テンプレートの起動** が実行できることを確認します。
* Job Templateの編集画面（Playbook、Credentials等の変更）にアクセスしようとすると、編集操作が許可されないことを確認します。
* `aap-training-content` Projectは閲覧できますが、Syncや設定変更はできないことを確認します。

## Section 5: Organization Adminロールの確認

比較のため、自分のユーザー（Organization Adminロールを持つ場合）で改めてログインし、Job Templateの作成・編集・削除、Credentialの作成など、すべての操作が可能であることを確認してください。

## まとめ

| ロールの組み合わせ例 | 用途 |
|---|---|
| Job Template: `Execute` | 運用担当者がJobを実行するだけの権限 |
| Project: `Read` + Inventory: `Read` | 構成内容を確認できるが変更はできない権限 |
| Job Template: `Admin` | Job Template自体の設定変更権限（Playbook変更、Credential変更など） |
| Organization: `Admin` | Organization配下のすべてを管理する権限（通常は少数の管理者のみ） |

RBACを適切に設計することで、「実行はできるが設定は変更できない」「閲覧はできるが実行はできない」といった、現場の運用フローに合わせた権限管理が可能になります。[宿題 課題4](../homework/hm1.md)では、より実践的なRBAC設計を行います。

---

[次へ進む](./ex2.md)
