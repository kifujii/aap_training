# 宿題

今までの基礎演習・応用演習で学んだことを使って、実際にAAP上でWeb基盤の構築〜運用のフローを組み立ててもらいます。

みなさんに構築してもらったAAP構成（Organization/Credential/Inventory/Project/Job Template/Workflow等）とGitLabリポジトリの内容をレビューさせていただいて、研修が全て終了となります。

## シナリオ

あなたはインフラエンジニアとして、社内向けWebアプリケーションの基盤をAAPで構築・運用することになりました。

最初はシンプルな構成からスタートし、利用者の増加やセキュリティ監査、運用効率化の要求に応じて段階的にシステムを成長させていきます。

各課題は前の課題で作ったAAP構成・GitLabリポジトリの内容に機能を追加していく形式です。Project/Job Templateの構成は**壊さず・増やしすぎず**、必要な機能を既存のオブジェクトに組み込んでいってください。

## 成果物の構成

最終的に以下の構成で提出してください。課題を進めるごとに、GitLabリポジトリのファイルとAAP上のオブジェクトが増えていきます。

### GitLabリポジトリ（`aap-training-content`、[基礎演習4](../basic-controller/ex4.md)で作成したもの）

```
aap-training-content/
├── inventory.yml                # 課題1〜2で更新
├── site.yml                      # 課題1〜4で更新
├── verify.yml                     # 課題3で拡張
├── secrets.yml                    # vault暗号化済み（課題4で追加）
├── rulebooks/
│   └── auto_recovery.yml          # 課題5で追加
├── .gitlab-ci.yml                 # 課題5で追加（任意）
└── roles/
    ├── nginx/                     # 課題1で作成
    ├── haproxy/                   # 課題1で作成、課題4で拡張
    └── test/                      # 課題3で作成
```

### AAP上のオブジェクト

```
Organization: （研修用に指定されたもの）
Credentials: managed-nodes-ssh, gitlab-scm-credential, vault-credential
Project: aap-training-content
Inventory: training-inventory（Inventory Source経由）
Job Templates: deploy-web-stack, verify-deployment
Workflow Job Templates: web-stack-workflow
Teams: web-ops-team（課題4で作成）
Notification Templates: job-failure-webhook
Rulebook Activations: auto-recovery-activation（課題5で作成）
```

## 宿題のルール

* **生成AI（ChatGPT、GitHub Copilot、Claude等）の使用は禁止です。** 自分の手でAAPの設定を行い、自分の頭で考えてください。公式ドキュメントの参照はもちろんOKです。
* 本資料で扱っていない**OSS版Ansibleのモジュール・構文**（`uri`, `assert`, `block/rescue`, `serial` など）を使う課題があります。これらの文法自体は[konono/ansible_training_2026](https://github.com/konono/ansible_training_2026)（参考元のOSS版教材）の応用演習で解説されているので、必要に応じて参照してください。

## コーディング規約

本宿題では以下の規約に従ってください。

* `site.yml` の各Playには `become: true` を明示的に指定すること
* roleの適用は `roles:` ディレクティブを使うこと
* AAPのCredential名は、種類がひと目でわかるように命名すること（例: `<用途>-<Credential Type>`）
* AAPのJob Template名は、動詞＋対象の形式で命名すること（例: `deploy-web-stack`）

---

## 課題 1 — Web基盤の構築

新規プロジェクトが始まりました。[基礎演習4〜7](../basic-controller/ex7.md)で学んだ内容を使って、GitLab連携・Inventory Source・Job Templateの基本構成を作ってください。

### 課題1の前提条件

* [事前準備](../prepare_training_environment.md)に記載の管理対象ノードが利用可能であること
* 課題に取り組む前に、**設計ドキュメント**を作成すること（形式: ppt or Markdown）

#### 設計ドキュメントに含める内容

* 構成の概要（AAPのObjectと、実際のGitLab/管理対象ノードとの関係図）
* アーキテクチャ図（Client → HAProxy:80 → nginx×2:8080）
* AAP上で作成するOrganization/Credential/Project/Inventory/Job Templateの一覧
* GitLabリポジトリのディレクトリ構成（上記の「成果物の構成」を参考に）
* Inventory Sourceで読み込むYAMLインベントリの設計（どの変数をホスト変数にするか）

### 要件

* `nginx` role と `haproxy` role を作成し、`site.yml` から呼び出すこと（[sample-content](../sample-content/)をベースにしてよい）
* **nginx**: port 8080で待ち受け、`index.html` に**自ノードのIPアドレス**と**バージョン情報**（デフォルト `v1.0`）を表示すること
* **HAProxy**: port 80で待ち受け、backendのnginxに**Round-Robin**で分散すること
* GitLab ProjectとAAPのSource Control Project連携、Inventory Source、Job Template（`deploy-web-stack`）をすべて作成し、「Launch」1つでデプロイが完了する状態にすること

<details>
<summary>ヒント</summary>

**参考演習:**
* Projectの作成、Inventory Source → [基礎演習4](../basic-controller/ex4.md)、[基礎演習5](../basic-controller/ex5.md)
* Job Templateの作成 → [基礎演習6](../basic-controller/ex6.md)
* roleの作り方、テンプレートと変数 → [konono/ansible_training_2026](https://github.com/konono/ansible_training_2026) 基礎演習5〜6

</details>

---

## 課題 2 — サーバー増設

利用者が増えてきたため、nginxを1台増設して3台体制にすることになりました。

### 要件

* GitLab上の `inventory.yml` に新しいWebサーバー（例: `web3`）を追加し、計4台（nginx×3, haproxy×1）の構成にする
* **AAPの画面からInventoryを直接編集するのではなく**、GitLabへのpush→Inventory SourceのSync→Job Templateの実行、という手順だけで増設が完了する状態にすること

### 取り組み方

まず `inventory.yml` に新しいホストを追加してGitLabにpushし、Inventory SourceをSyncしてから `deploy-web-stack` を実行してみてください。

* うまくいった場合 → 新しいホストにnginxがデプロイされ、HAProxyのbackendにも自動的に追加されていれば完了です
* うまくいかなかった場合 → 課題1で作ったroleのコードを見直しましょう。例えば以下のような点が原因になることが多いです
  * HAProxyテンプレートのbackendホスト一覧が**ハードコード**されていないか？ → Inventoryの `web` グループから**動的に列挙**する形に修正する
  * nginxのポート番号などがHAProxyテンプレート内で**固定値**になっていないか？ → `hostvars` 経由で変数を参照する形に修正する

### 確認ポイント

* `inventory.yml` の変更・push・Inventory SourceのSync・Job Template実行だけでnginxのデプロイとHAProxyのbackend追加が完了しているか？
* テンプレート内で `{% for %}` ループを使い、Inventoryから動的にホスト一覧を取得しているか？

<details>
<summary>ヒント</summary>

**参考演習:**
* Inventory Sourceと `hostvars` の考え方 → [基礎演習5](../basic-controller/ex5.md)
* テンプレート内の `{% for %}` ループ、`hostvars` の参照方法 → [konono/ansible_training_2026](https://github.com/konono/ansible_training_2026) 応用演習6・9

</details>

---

## 課題 3 — Workflow化と自動テスト

「デプロイの都度、承認とデプロイ後の確認を手作業でやるのが面倒です」という運用チームからの声を受けて、Workflow化と自動テストを追加してください。

### 要件

* [応用演習4](../advanced/ex4.md)を参考に、`deploy-web-stack` → 承認（Approval） → 検証、という流れの **Workflow Job Template** を構築すること
* 検証ステップでは、単にホストの到達性を確認する `verify.yml` を拡張し、`test` roleを新規作成して以下を自動検証すること（`assert` モジュールで検証し、失敗したらPlaybookをエラーで停止させること）
  * コントローラーからHAProxy経由でHTTPリクエストを送り、**ステータスコードが200**であること
  * レスポンスボディに**webグループのいずれかのIPアドレス**が含まれていること
  * webノードの台数分リクエストを投げたとき、**レスポンスボディが全て異なる**こと（Round-Robin確認）
* HAProxyのポートが応答可能になるまで**待機してから**テストを開始すること
* [基礎演習7](../basic-controller/ex7.md)を参考に、`deploy-web-stack` にSurveyを設定し、`nginx_site_version` をデプロイ実行者が入力できるようにすること

<details>
<summary>ヒント</summary>

* ポートの待機・HTTPリクエスト・リトライ制御・`assert` の文法は、本資料では扱っていないOSS版Ansibleの範囲です。[konono/ansible_training_2026](https://github.com/konono/ansible_training_2026) の応用演習1・3を参照してください。
* `uri` モジュールはCheck modeに非対応です。Workflow内のノードをCheckモードで動かす予定がある場合は注意してください。

**参考演習:**
* Workflow Job Template、Approvalノード → [応用演習4](../advanced/ex4.md)
* Survey → [基礎演習7](../basic-controller/ex7.md)
* `uri`/`wait_for`/`assert`の文法 → [konono/ansible_training_2026](https://github.com/konono/ansible_training_2026) 応用演習1・3

</details>

---

## 課題 4 — セキュリティ対応

セキュリティ監査チームから以下の要件が来ました。既存の構成に組み込んで対応してください。

### 要件

**a) 権限の最小化** — RBAC設計

[応用演習1](../advanced/ex1.md)を参考に、`web-ops-team` を作成し、以下の方針でロールを割り当てること。

* `web-ops-team` には `deploy-web-stack` と `web-stack-workflow` の `Execute` ロールのみ付与する（編集権限は付与しない）
* `aap-training-content` Projectには `Read` ロールのみ付与する

**b) 秘密情報の保護** — Vault Credential

[応用演習2](../advanced/ex2.md)を参考に、HAProxyのstats画面に認証（`stats auth user:pass`）を追加すること。ユーザー名とパスワードは `secrets.yml` に記述し、`ansible-vault encrypt` で暗号化して管理し、`vault-credential` 経由でJob Templateから復号できるようにすること。

**c) 失敗時の通知** — Notification

[応用演習3](../advanced/ex3.md)を参考に、`deploy-web-stack` と `web-stack-workflow` の両方に、失敗時（Error）の通知を設定すること。

<details>
<summary>ヒント</summary>

**参考演習:**
* a) RBACとTeam/ロール設計 → [応用演習1](../advanced/ex1.md)
* b) ansible-vaultとVault Credential → [応用演習2](../advanced/ex2.md)
* c) Notification Template → [応用演習3](../advanced/ex3.md)

</details>

---

## 課題 5 — 本番運用に向けた仕上げ

いよいよ本番環境への適用が近づいてきました。以下の要件を追加して、本番運用に耐える構成に仕上げてください。

### 要件

**a) 定期的な健全性チェック** — Schedule

[応用演習3](../advanced/ex3.md)を参考に、`verify-deployment`（または課題3で拡張した検証Job Template）を定期実行するScheduleを設定すること。

**b) イベント駆動による自動対応** — EDA

[応用演習9〜10](../advanced/ex9.md)を参考に、以下を満たすRulebook（`rulebooks/auto_recovery.yml`）とRulebook Activationを作成すること。

* Webhookで `{"status": "down", "host": "<ホスト名>"}` 形式のイベントを受信する
* イベントを受信したら、該当ホストを `Limit` に指定して `deploy-web-stack` を再実行する（設定の再適用による自動復旧を想定）
* Rulebook ActivationのOutputログに、受信したイベント内容が記録されることを確認する

<details>
<summary>ヒント</summary>

**参考演習:**
* Schedule → [応用演習3](../advanced/ex3.md)
* Rulebookの基本構造、`run_job_template`アクション → [応用演習9](../advanced/ex9.md)、[応用演習10](../advanced/ex10.md)

</details>

---

## 付録: 環境情報

管理対象ノードの実際のホスト名・IPアドレス・認証情報は[事前準備](../prepare_training_environment.md)で確認した値を使用してください。本資料内の例示（`web1`〜`web3`, `lb`, `192.0.2.x`）は、実際の環境の値に置き換えて進めてください。

| 項目 | 内容 |
|---|---|
| AAPのURL / ログイン情報 | [事前準備](../prepare_training_environment.md)参照 |
| GitLabのURL / ログイン情報 | [事前準備](../prepare_training_environment.md)参照 |
| 管理対象ノードの構成 | Webサーバー × 3、ロードバランサー × 1（課題2で3台体制に増設） |
