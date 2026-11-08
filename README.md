# Ansible Automation Platform ハンズオントレーニング

## 概要

* **目的:** Ansible Automation Platform (AAP) の画面操作を、Linux と Windows へのファイル配置を題材にして学ぶ
* **対象:** AAPを使ったことがない方〜運用担当者
* **前提:** Playbook の `hosts`、`tasks`、モジュール、変数の読み方が分かること。AAP の操作はこの資料だけで進めます
* **前提環境:** AAP 2.5 以降（画面確認は 2.7 の日本語UI）、GitLab（画面確認は 19 の英語UI）。いずれも構築済みで、ユーザーも作成済み

## 事前準備

→ [事前準備（確認リスト）](./prepare_training_environment.md)

本資料は AAP と GitLab が構築済みであることを前提にしています。AAP 自体のインストール手順は含みません。

## どの演習をやるか

対象の Linux と Windows がすでにある場合と、AWS で新しく作る場合で、やる演習が一部違います。表の「実施」だけを、番号順に進めてください。

| 演習 | サーバがすでにある | AWSで新しく作る |
|---|---|---|
| [1 ログイン](./basic-intro/ex1.md) | 実施 | 実施 |
| [2 認証情報](./basic-intro/ex2.md) | 実施 | 実施。サーバのログイン情報は演習3のあと |
| [3 AWSでサーバを作る](./basic-intro/ex3.md) | **飛ばして演習4へ** | 実施 |
| [4 インベントリと疎通](./basic-intro/ex4.md) | 実施 | 実施 |
| [5 GitLab](./basic-controller/ex5.md) | 実施 | 実施。演習3で作ったプロジェクトに残りのファイルを足す |
| [6 Inventory Source](./basic-controller/ex6.md) | 実施 | 実施 |
| [7 Job Template](./basic-controller/ex7.md) | 実施 | 実施 |
| [8 Survey](./basic-controller/ex8.md) | 実施 | 実施 |
| [9 Vault](./basic-controller/ex9.md) | 実施 | 実施 |
| [10 スケジュール](./basic-controller/ex10.md) | 実施 | 実施 |
| [11 Workflow](./basic-controller/ex11.md) | 実施 | 実施 |
| [応用1 RBAC](./advanced/ex1.md) | 実施 | 実施 |
| [応用2 通知](./advanced/ex2.md) | 環境内のメールサーバ | S3に書いてログで確認 |
| [応用3 EEの確認](./advanced/ex3.md) | 実施 | 実施 |
| [応用4 カスタムEE](./advanced/ex4.md) | イメージを取得できるときだけ | AAPサーバで実施 |
| [応用5 API](./advanced/ex5.md) | 実施 | 実施 |
| [応用6 CLI](./advanced/ex6.md) | インターネットへ出られるとき | AAPサーバで実施 |
| [応用7 EDA](./advanced/ex7.md) | 実施 | 実施 |
| [応用8 EDAからJob起動](./advanced/ex8.md) | 実施 | 実施 |
| [応用9 GitLab応用](./advanced/ex9.md) | ランナーがイメージを取得できるとき、CIの節を実施 | 実施 |

サーバがすでにある環境では、演習中にインターネットへ出ない前提で進めます。コレクションの追加取得や、外のコンテナイメージの取得はしません。AWS でサーバを作る場合は、インターネットへ出られる前提です。

作るファイルは次の2か所です。OSの時刻、パッケージ、SSHの設定は変えません。

| OS | ファイル |
|---|---|
| Linux | `/var/tmp/aap-training/note.txt` |
| Windows | `C:\Users\Public\aap-training\note.txt` |

Windows への接続方式は PSRP です。ドメインに参加していない Windows（AWS で作るサーバを含む）は NTLM、ドメインに参加している Windows は Kerberos を使います。

## 対象バージョン

* AAP 2.5 以降（統合UI: **Automation Execution** / **Automation Decisions** / **Access Management**）
* SCM: GitLab。ファイルの編集は Web IDE を使います
* カスタム Execution Environment を作る場合は AAP サーバ上で行います。そのための別サーバは作りません
* Private Automation Hub は対象外です

## 画面の見方

→ [画面ラベル対応表](./ui-labels.md)

## 用語集

→ [AAP用語集](./aap_glossary.md)

## サンプル

→ [sample-content/](./sample-content/)

GitLab のプロジェクトへ、Web IDE で置いて使います。

## 宿題

→ [宿題](./homework/hm1.md)

課題1から順に、同じ Linux と Windows へ機能を足していきます。各課題に完了条件を書いてあります。
