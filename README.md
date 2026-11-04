# Ansible Automation Platform ハンズオントレーニング

## 概要

* **目的:** Ansible Automation Platform (AAP) の基本操作から応用機能まで実践的に学ぶ
* **対象:** AAPを使ったことがない方〜運用担当者
* **前提環境:** AAP 2.5 以降（画面確認は 2.7 の日本語UI）、GitLab（画面確認は 19 の英語UI）。いずれも構築済み・ユーザ作成済み

本資料は [konono/ansible_training_2026](https://github.com/konono/ansible_training_2026)（OSS Ansibleのハンズオン教材）の構成を参考に、AAPの機能を使う前提で再構成したものです。Ansible Playbook自体の書き方（loop, when, role化など）を学びたい場合は、まず参考元のOSS版教材で基礎を身につけてから本資料に進むことを推奨します。

## 事前準備

→ [事前準備（確認リスト）](./prepare_training_environment.md)

本資料はAAPが構築済み・ユーザも作成済みであることを前提としています。構築手順は対象外です。

## 対象バージョン・環境前提

* AAP 2.5 以降（統合UI: **Automation Execution** / **Automation Decisions** / **Access Management**）
* SCM連携: GitLab（既存ユーザ利用可）
* Execution Environment作成: `ansible-builder` が使える、インターネット接続可能な別Linuxサーバを想定
* Private Automation Hubは本資料の対象外

## 画面の見方

→ [画面ラベル対応表](./ui-labels.md)

ボタン名やメニュー名は、ここを見ながら読み替えてください。主要画面のスクリーンショットも各演習に入れています。

## 用語集

→ [AAP用語集](./aap_glossary.md)

## サンプルコンテンツ

→ [sample-content/](./sample-content/) — 演習で使うサンプルPlaybook/Role。GitLabのProjectにpushして使用します。

操作対象のLinuxサーバは、AWSで新規作成するか、既存サーバをインベントリに書きます。→ [操作対象サーバ（AWS）](./aws_targets.md)

## カリキュラム

### 基礎演習 — AAPの基本操作 (basic-intro/)

全員必須。順番に実施してください。

| # | タイトル | 学ぶこと |
|---|---------|---------|
| 1 | [AAPへのログインとUI構成の理解](./basic-intro/ex1.md) | 統合UI構成、Organizationの確認 |
| 2 | [Credentialの作成](./basic-intro/ex2.md) | Machine Credential、Source Control Credential |
| 3 | [Inventoryとホスト/グループの作成、疎通確認](./basic-intro/ex3.md) | 静的Inventory、Ad Hoc Command |

### 基礎演習 — Project・Job Template (basic-controller/)

basic-intro修了後に実施してください。

| # | タイトル | 学ぶこと |
|---|---------|---------|
| 4 | [Projectの作成（GitLab連携）](./basic-controller/ex4.md) | GitLab Project作成、SCM Project、Sync |
| 5 | [Inventory Source（SCM経由のYAMLインベントリ）](./basic-controller/ex5.md) | Inventory Source、SCMベースの動的読み込み |
| 6 | [初めてのJob Templateの作成と実行](./basic-controller/ex6.md) | Job Template、実行、出力確認 |
| 7 | [Job Template詳細設定とSurvey](./basic-controller/ex7.md) | Extra Vars、Limit、Prompt on Launch、Survey |

### 応用演習 (advanced/)

basic修了後、目的に応じて選択してください。

| # | タイトル | 学ぶこと |
|---|---------|---------|
| 1 | [RBACとアクセス制御](./advanced/ex1.md) | Organization/Team/User、ビルトインロール |
| 2 | [Vault Credentialと秘密情報管理](./advanced/ex2.md) | ansible-vault、Vault Credential |
| 3 | [ScheduleとNotification](./advanced/ex3.md) | Cronスケジュール、Notification Template |
| 4 | [Workflow Job Template](./advanced/ex4.md) | 複数Job連携、成功/失敗分岐、Approval Node |
| 5 | [Execution Environments入門](./advanced/ex5.md) | EEの概念、Job Templateへの割当 |
| 6 | [カスタムEEの作成](./advanced/ex6.md) | ansible-builder、レジストリpush、Controller登録 |
| 7 | [REST APIの活用](./advanced/ex7.md) | 認証、Job起動・状態取得 |
| 8 | [CLI/Automation連携](./advanced/ex8.md) | ansible.controller collection、ansible-navigator |
| 9 | [EDA入門](./advanced/ex9.md) | Decision Environment、Rulebook基本構造 |
| 10 | [EDA Rulebook ActivationとController連携](./advanced/ex10.md) | Event Stream、Job Template起動連携 |
| 11 | [GitLab連携の応用](./advanced/ex11.md) | Webhook自動Sync、CI/CDパイプライン統合 |

### 宿題

→ [宿題](./homework/hm1.md)

全課題がひとつのシナリオ（Linuxサーバの初期設定→運用）として繋がっています。前の課題から継続して機能を追加していく形式です。

| 課題 | シナリオ | 使うテクニック |
|------|------|-----------|
| 1 | 初期設定の適用 | Credential, Inventory Source, Project, Job Template |
| 2 | サーバ増設 | Inventoryの変更のみで対応 |
| 3 | Workflow化と点検 | Workflow Job Template, Survey, 適用後の点検 |
| 4 | セキュリティ対応 | RBAC設計, Vault Credential, Notification |
| 5 | 本番運用に向けた仕上げ | Schedule, EDAによる自動対応 |
