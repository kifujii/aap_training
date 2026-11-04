# 応用演習 5 - Execution Environments入門

この演習では、Job実行の裏側で使われている **Execution Environment (EE)** の仕組みを理解します。カスタムEEの作成は次の[応用演習6](./ex6.md)で扱うため、ここではまず既存のEEを確認します。

**前提条件:** 基礎演習1〜7を完了していること（応用演習1〜4とは独立して実施できます）

[前に戻る](./ex4.md) | [次へ進む](./ex6.md)

---

## Section 1: Execution Environmentとは

従来のAnsible Tower/AWXでは、Ansible本体やCollection、Pythonライブラリをコントローラー自体に直接インストールしていました。この方式には以下のような課題がありました。

* プロジェクトAとプロジェクトBで必要なAnsibleのバージョンやCollectionが競合する
* コントローラーの環境を変更すると、既存のすべてのPlaybook実行に影響する

AAPでは、Job実行に必要なAnsible本体・Python依存パッケージ・Collectionなどをひとまとめにした**コンテナイメージ**（Execution Environment）を使ってJobを実行します。Job Templateごとに異なるEEを指定できるため、Playbook/プロジェクトごとに必要なバージョンを独立して管理できます。

## Section 2: Default Execution Environmentの確認

### Step 1: Execution Environments一覧を開く

**Automation Execution > Infrastructure > 実行環境** を開きます。

### Step 2: デフォルトEEの詳細を確認する

一覧に表示されているEE（例: `Default execution environment` のような名称）を開き、以下を確認します。

| 項目 | 内容 |
|---|---|
| Image | コンテナイメージの参照先（例: `registry.example.local/ee-supported-rhel9:latest`） |
| Pull | イメージの取得ポリシー（`Always` / `Missing` / `Never`） |
| Organization | 空欄の場合は全Organizationで使用可能 |

## Section 3: Job TemplateへのEE割当を確認する

### Step 1: Job Template編集画面を開く

`apply-baseline` の編集画面を開き、「Execution Environment」欄を確認します。

### Step 2: 割当を確認する

空欄（Projectのデフォルトを使用）になっているか、特定のEEが明示的に指定されているかを確認します。明示的に指定しない場合、Projectに設定されたデフォルトEE、またはシステム全体のデフォルトEEが使われます。

**Note:** EEの決定順序はおおむね「Job Template個別の指定」＞「Projectのデフォルト」＞「システムのデフォルト」の優先順位です。意図せず別のEEが使われている場合は、この優先順位を確認してください。

## Section 4: EEに含まれる内容を確認する（参考）

AAPの画面だけでは、EEコンテナ内にインストールされているCollectionの一覧までは詳細に確認できません。[応用演習6](./ex6.md)で導入する `ansible-navigator` を使うと、ローカル環境からEEイメージの内容を確認できます。

```bash
$ ansible-navigator images
```

実行すると、利用可能なEEイメージの一覧と、選択したイメージに含まれるAnsibleバージョン・Collection・Pythonパッケージの情報を対話的に確認できます（`:details` コマンドで詳細表示）。

## Section 5: なぜEEを意識する必要があるのか

| シナリオ | EEが関係する理由 |
|---|---|
| Playbookで `community.general` の新しいモジュールを使いたい | そのCollectionがEEに含まれているか確認が必要 |
| 複数チームが異なるAnsibleバージョンのPlaybookを運用している | チームごとに異なるEEを割り当てることで共存できる |
| セキュリティ部門から使用ソフトウェアの一覧提出を求められた | EEイメージ単位でバージョン・内容を管理・報告できる |

---

ここまでで、EEの基本的な役割と確認方法を理解しました。次の演習では、実際にカスタムEEを作成し、AAPに登録します。

[次へ進む](./ex6.md)
