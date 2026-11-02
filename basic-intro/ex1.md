# 演習 1 - AAPへのログインとUI構成の理解

> **注意:** この演習はすべてWebブラウザ上のAAP画面で行います。[事前準備](../prepare_training_environment.md)で確認したAAPのURL・ログイン情報を用意してください。

この演習では、AAP（Ansible Automation Platform）にログインし、2.5以降で採用されている統合UIの構成を理解します。あわせて、所属するOrganizationとUserの情報を確認します。

## Section 1: AAPへのログイン

### Step 1: ログイン画面を開く

ブラウザで、事前準備で確認したAAPのURLにアクセスします。

```
https://<AAPのURL>/
```

ログイン画面が表示されたら、配布されたユーザー名とパスワードを入力してログインしてください。

### Step 2: ログイン後の画面を確認

ログインに成功すると、ダッシュボード画面が表示されます。画面左側にナビゲーションメニューが表示されていることを確認してください。

**Note:** もしログインできない場合は、以下を確認してください。

* URLが正しいか（末尾のスラッシュの有無やポート番号など）
* ユーザー名・パスワードに誤りがないか（大文字小文字も含む）
* VPN接続や社内ネットワークへの接続が必要な環境かどうか

## Section 2: 統合UIのナビゲーション構成

AAP 2.5以降では、従来個別に提供されていたAutomation Controller（Playbook実行管理）とEDA Controller（イベント駆動自動化）の機能が、1つの統合UIにまとめられています。左側のナビゲーションは、大きく3つのグループに分かれています。

| ナビゲーショングループ | 主なメニュー | 役割 |
|---|---|---|
| **Automation Execution** | Templates, Projects, Inventories, Infrastructure（Execution Environments, Instance Groups, Instances）, Credentials, Jobs | Playbookの実行管理。本資料の中心となる機能群（旧 Automation Controller 相当） |
| **Automation Decisions** | Rulebook Activations, Rulebooks, Decision Environments, Event Streams | イベント駆動自動化（EDA）。[応用演習9](../advanced/ex9.md)以降で扱う |
| **Access Management** | Organizations, Teams, Users, Administration | RBAC（アクセス制御）関連。[応用演習1](../advanced/ex1.md)で詳しく扱う |

**Note:** AAP 2.4以前では、Automation ControllerとEDA Controllerが別々のURL・別々のUIで提供されていました。2.5以降はこれらが統合され、同一画面内でシームレスに切り替えられるようになっています。バージョンによって画面構成が異なる場合がありますが、本資料ではオブジェクトの名称（Project、Inventory、Job Templateなど）を軸に説明するため、多少メニューの配置が異なっても読み替えて進めてください。

### Step 1: Automation Executionメニューを開く

左側ナビゲーションから **Automation Execution** を開き、配下に以下のメニューがあることを確認してください。

* Templates（Job Template / Workflow Job Templateの一覧）
* Projects（SCM連携の管理単位）
* Inventories（管理対象ホストの一覧）
* Infrastructure（Execution Environments / Instance Groups / Instances）
* Credentials（認証情報の管理）
* Jobs（実行履歴）

現時点ではまだ何も作成していないため、各一覧は空の状態になっているはずです。これらを基礎演習2〜で順番に作成していきます。

### Step 2: Access Managementメニューを開く

次に **Access Management** を開き、以下のメニューを確認してください。

* Organizations
* Teams
* Users
* Administration

## Section 3: Organizationの確認

AAPでは、Project・Inventory・Credential・Job Templateなどのほぼすべてのオブジェクトが、いずれか1つの **Organization** に属します。Organizationはマルチテナント運用（部署・チームごとの管理区分）の基本単位です。

### Step 1: Organization一覧を開く

**Access Management > Organizations** を開きます。少なくとも1つ（多くの場合 `Default`、または研修用に用意された専用のOrganization）が存在することを確認してください。

### Step 2: 使用するOrganizationを控える

以降の演習（Credential、Inventory、Projectなどの作成）では、このOrganizationを選択して作成していきます。[事前準備](../prepare_training_environment.md)で確認したOrganization名を使用してください。

**Note:** 以降の演習手順では単に「Organizationを選択」と記載しますが、実際には上記で確認したOrganization名を選んでください。

## Section 4: Userの確認

### Step 1: 自分のUser情報を確認する

**Access Management > Users** から、自分のユーザー名を検索し、詳細画面を開いてください。以下の項目を確認します。

* 所属Organization
* 付与されているロール（System Administrator、Organization Admin等）

**Note:** ロールの詳細（どのロールがどの操作を許可するか）は[応用演習1 - RBACとアクセス制御](../advanced/ex1.md)で扱います。現時点では「Userにはロールが紐付いている」という点だけ理解しておいてください。

## Section 5: Jobsメニューの確認

**Automation Execution > Jobs** を開いてください。まだJob Templateを作成していないため、実行履歴は空です。この画面には今後、演習で実行したJobの履歴とログが蓄積されていきます。

---

ここまでで、AAPへのログインと統合UIの構成、Organization/Userの確認が完了しました。次の演習では、管理対象ノードへの接続に使うCredentialを作成します。

[次へ進む](./ex2.md)
