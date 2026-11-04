# 宿題

今までの基礎演習・応用演習で学んだことを使って、Linuxサーバの初期設定から定常運用までの流れをAAP上に組み立ててもらいます。

みなさんに構築してもらったAAP構成（Organization/Credential/Inventory/Project/Job Template/Workflow等）とGitLabリポジトリの内容をレビューさせていただいて、研修が全て終了となります。

## シナリオ

あなたはインフラエンジニアとして、社内のLinuxサーバをAAPで初期設定し、増設・点検・権限分離・定期確認までを同じ仕組みに載せることになりました。

最初は時刻同期、基本パッケージ、ログインバナー、SSHの接続制限から始めます。その後、台数増加、適用前の承認、管理ユーザーのパスワード保護、失敗通知、定期点検、ディスク逼迫を想定した再適用へ広げます。Webサイトの公開やロードバランサーは対象にしません。

各課題は前の課題で作ったAAP構成・GitLabリポジトリの内容に機能を追加していく形式です。Project/Job Templateの構成は**壊さず・増やしすぎず**、必要な機能を既存のオブジェクトに組み込んでいってください。

サーバがまだ無い場合は [操作対象サーバ（AWS）](../aws_targets.md) の経路AでEC2を作成します。すでにサーバがある場合は経路Bで、`inventory.yml` の `ansible_host` にそのIPまたはDNS名を書きます。

## 成果物の構成

最終的に以下の構成で提出してください。課題を進めるごとに、GitLabリポジトリのファイルとAAP上のオブジェクトが増えていきます。

### GitLabリポジトリ（`aap-training-content`、[基礎演習4](../basic-controller/ex4.md)で作成したもの）

```
aap-training-content/
├── inventory.yml                  # 課題1〜2で更新
├── group_vars/all.yml
├── site.yml                       # 課題1の初期設定
├── verify.yml                     # 課題3で点検項目を追加
├── patch.yml                      # 課題5の更新確認に使ってよい
├── provision_ec2.yml              # 経路Aを選んだ場合
├── secrets.yml                    # vault暗号化済み（課題4で追加）
├── rulebooks/
│   └── auto_recovery.yml          # 課題5で追加
└── roles/
    ├── baseline/
    └── access/
```

### AAP上のオブジェクト

```
Organization: （研修用に指定されたもの）
Credentials: managed-nodes-ssh, gitlab-scm-credential, vault-credential
             （経路Aを実施した場合は aws-training も）
Project: aap-training-content
Inventory: training-inventory（Inventory Source経由）
Job Templates: apply-baseline, verify-hosts
               （経路Aを実施した場合は provision-ec2 も）
Workflow Job Templates: baseline-workflow
Teams: infra-ops-team（課題4で作成）
Notification Templates: job-failure-webhook
Rulebook Activations: auto-recovery-activation（課題5で作成）
```

## 宿題のルール

* **生成AI（ChatGPT、GitHub Copilot、Claude等）の使用は禁止です。** 自分の手でAAPの設定を行い、自分の頭で考えてください。公式ドキュメントの参照はもちろんOKです。
* 本資料で扱っていない**OSS版Ansibleのモジュール・構文**（`assert`, `command`, `wait_for` など）を使う課題があります。これらの文法自体は[konono/ansible_training_2026](https://github.com/konono/ansible_training_2026)（参考元のOSS版教材）の応用演習で解説されているので、必要に応じて参照してください。

## コーディング規約

本宿題では以下の規約に従ってください。

* `site.yml` の各Playには `become: true` を明示的に指定すること
* roleの適用は `roles:` ディレクティブを使うこと
* AAPのCredential名は、種類がひと目でわかるように命名すること（例: `<用途>-<Credential Type>`）
* AAPのJob Template名は、動詞＋対象の形式で命名すること（例: `apply-baseline`）

---

## 課題 1 — 初期設定の適用

[基礎演習4〜7](../basic-controller/ex7.md)で学んだ内容を使って、GitLab連携・Inventory Source・Job Templateの基本構成を作ってください。

### 課題1の前提条件

* 経路AならEC2を作成できるAWSアカウント、経路BならSSHできる既存サーバがあること（[事前準備](../prepare_training_environment.md)）
* 課題に取り組む前に、**設計ドキュメント**を作成すること（形式: ppt or Markdown）

#### 設計ドキュメントに含める内容

* 構成の概要（AAPのObjectと、GitLab・管理対象ノードの関係。経路Aを選ぶ場合はAWS APIでEC2を作る位置も書く）
* 適用する設定の一覧（タイムゾーン、パッケージ、chronyd、MOTD、sshdの接続制限）
* AAP上で作成するOrganization/Credential/Project/Inventory/Job Templateの一覧
* GitLabリポジトリのディレクトリ構成（上記の「成果物の構成」を参考に）
* Inventory Sourceで読み込むYAMLインベントリの設計（どの値をホスト変数にするか）

### 要件

* 経路Aまたは経路Bのどちらかで、`managed` グループのホストがAAPからSSHできる状態にすること
* [sample-content](../sample-content/) の `baseline` role と `access` role を使い、`site.yml` から呼び出すこと。設定項目を1つ以上、自分で追加してよい
* Job Template `apply-baseline` の **テンプレートの起動** 1回で、対象ホストのタイムゾーン、chronyd、`/etc/motd`、`/etc/ssh/sshd_config.d/99-aap-training.conf` がそろうこと
* 同じJobを再実行したとき、変更が無いタスクは `ok` または `skipped` になり、`changed` が残らないこと

<details>
<summary>ヒント</summary>

**参考演習:**
* 対象サーバの用意 → [操作対象サーバ（AWS）](../aws_targets.md)
* Projectの作成、Inventory Source → [基礎演習4](../basic-controller/ex4.md)、[基礎演習5](../basic-controller/ex5.md)
* Job Templateの作成 → [基礎演習6](../basic-controller/ex6.md)

</details>

---

## 課題 2 — サーバ増設

管理対象を1台増やします。Playbookにホスト名を直接書き足すのではなく、インベントリの変更だけで増設が終わる状態にしてください。

### 要件

* GitLab上の `inventory.yml` に新しいホスト（例: `srv3`）を追加する。経路Aで台数を増やす場合は、`ec2_instance_names` に同じ名前を足して `provision-ec2` を再実行し、払い出されたIPを `ansible_host` に書く
* **AAPの画面からInventoryを直接編集するのではなく**、GitLabへのpush→Inventory SourceのSync→`apply-baseline` の実行、という手順だけで増設が完了する状態にすること
* 増設後、既存ホストの `/etc/aap-training-peers` にも新しいホストが含まれること

### 取り組み方

まず `inventory.yml` にホストを追加してGitLabにpushし、Inventory SourceをSyncしてから `apply-baseline` を実行してください。

* うまくいった場合 → 新しいホストに初期設定が入り、既存ホストの peers ファイルにもそのホストが追加されていれば完了です
* peers ファイルが更新されない場合 → [sample-content/roles/baseline/templates/peers.txt.j2](../sample-content/roles/baseline/templates/peers.txt.j2) が `groups['managed']` と `hostvars` から一覧を作っているかを確認してください。ホスト名をテンプレートへ直接書いていないかを見ます

### 確認ポイント

* `inventory.yml` の変更、push、Inventory SourceのSync、Job Templateの実行だけで増設が完了しているか
* peers ファイルが `{% for %}` で `managed` グループを列挙しているか

<details>
<summary>ヒント</summary>

**参考演習:**
* Inventory Sourceとホスト変数 → [基礎演習5](../basic-controller/ex5.md)
* テンプレート内の `{% for %}` と `hostvars` → [konono/ansible_training_2026](https://github.com/konono/ansible_training_2026) 応用演習6・9

</details>

---

## 課題 3 — Workflow化と点検

初期設定のたびに、人が内容を確認してから点検できるようにしてください。

### 要件

* [応用演習4](../advanced/ex4.md)を参考に、`apply-baseline` → 承認（Approval） → `verify-hosts` という **Workflow Job Template** `baseline-workflow` を作ること
* `verify.yml` に点検を1つ以上追加し、失敗したらPlaybookが停止すること。サンプルにあるタイムゾーン、chronyd、ディスク空きに加えて、次のいずれかを `assert` で確認する
  * `/etc/ssh/sshd_config.d/99-aap-training.conf` に `MaxAuthTries` の設定値があること
  * `chronyc tracking` の結果に `Leap status` が含まれること
* [基礎演習7](../basic-controller/ex7.md)を参考に、`apply-baseline` のSurveyで `host_timezone` を実行者が入力できるようにすること。確認が終わったら `Asia/Tokyo` に戻すこと

<details>
<summary>ヒント</summary>

* `assert` と `command` の書き方は、サンプルの [verify.yml](../sample-content/verify.yml) と [konono/ansible_training_2026](https://github.com/konono/ansible_training_2026) の応用演習1・3が参考になります。

**参考演習:**
* Workflow Job Template、Approvalノード → [応用演習4](../advanced/ex4.md)
* Survey → [基礎演習7](../basic-controller/ex7.md)

</details>

---

## 課題 4 — セキュリティ対応

既存の構成に、権限分離、パスワードの暗号化、失敗通知を追加してください。

### 要件

**a) 権限の最小化** — RBAC設計

[応用演習1](../advanced/ex1.md)を参考に、`infra-ops-team` を作成し、以下の方針でロールを割り当てること。

* `infra-ops-team` には `apply-baseline` と `baseline-workflow` の `Execute` ロールのみ付与する（編集権限は付与しない）
* `aap-training-content` Projectには `Read` ロールのみ付与する

**b) 秘密情報の保護** — Vault Credential

[応用演習2](../advanced/ex2.md)を参考に、ローカル管理ユーザーのパスワードを `secrets.yml` に書き、`ansible-vault encrypt` で暗号化すること。平文のパスワードとVaultパスワードはリポジトリに置かないこと。`apply-baseline` から `vault-credential` で復号し、ユーザーが `wheel` に所属する状態にすること。暗号化前の形は [example_config/secrets.yml.example](./example_config/secrets.yml.example) を参照してください。

**c) 失敗時の通知** — Notification

[応用演習3](../advanced/ex3.md)を参考に、`apply-baseline` と `baseline-workflow` の両方に、失敗時（Error）の通知を設定すること。

<details>
<summary>ヒント</summary>

**参考演習:**
* a) RBACとTeam/ロール設計 → [応用演習1](../advanced/ex1.md)
* b) ansible-vaultとVault Credential → [応用演習2](../advanced/ex2.md)
* c) Notification Template → [応用演習3](../advanced/ex3.md)

</details>

---

## 課題 5 — 本番運用に向けた仕上げ

定期点検と、イベントを受けた再適用を追加してください。

### 要件

**a) 定期的な健全性チェック** — Schedule

[応用演習3](../advanced/ex3.md)を参考に、`verify-hosts` を定期実行するScheduleを設定すること。設定変更を伴う `apply-baseline` ではなく、点検用のJob Templateに付けること。

**b) 更新の有無を確認するJob** — 任意

[sample-content/patch.yml](../sample-content/patch.yml) を実行するJob Templateを作り、更新一覧だけを表示できることを確認すること。パッケージの適用（`patch_apply=true`）は、承認を挟むWorkflowにしない限り実行しないこと。

**c) イベント駆動による再適用** — EDA

[応用演習9〜10](../advanced/ex9.md)を参考に、次を満たすRulebook（`rulebooks/auto_recovery.yml`）とRulebook Activationを作ること。

* Webhookで `{"status": "disk_low", "host": "srv1"}` 形式のイベントを受信する
* 受信したら、`host` の値を `Limit` にして `apply-baseline` を再実行する
* Rulebook ActivationのOutputログに、受信したイベント内容が記録されること

<details>
<summary>ヒント</summary>

**参考演習:**
* Schedule → [応用演習3](../advanced/ex3.md)
* Rulebookの基本構造、`run_job_template`アクション → [応用演習9](../advanced/ex9.md)、[応用演習10](../advanced/ex10.md)

</details>

---

## 付録: 環境情報

管理対象のホスト名、IPアドレス、認証情報は[事前準備](../prepare_training_environment.md)と[操作対象サーバ（AWS）](../aws_targets.md)で確認した値を使ってください。資料中の `srv1`、`srv2`、`10.0.1.11` は例です。

| 項目 | 内容 |
|---|---|
| AAPのURL / ログイン情報 | [事前準備](../prepare_training_environment.md)参照 |
| GitLabのURL / ログイン情報 | [事前準備](../prepare_training_environment.md)参照 |
| 管理対象 | `managed` グループのLinuxサーバ。課題2で1台追加する |
