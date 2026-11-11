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

対象の Linux と Windows がすでにある場合と、AWS で新しく作る場合で、やる演習が一部違います。次のどちらかの順に進めてください。

サーバがすでにある場合: 演習1 → 2 → 3（Section 3 は飛ばす）→ 5 → 6 → 7 → 8 → 9 → 10 → 11 → 応用。演習4はしません。

サーバが無い場合: 演習1 → 2 → 3（Section 1、2、3）→ 4 → 3（Section 4、5）→ 5 → 6 → 7 → 8 → 9 → 10 → 11 → 応用。

| 演習 | サーバがすでにある | AWSで新しく作る |
|---|---|---|
| [1 ログイン](./basic-intro/ex1.md) | 実施 | 実施 |
| [2 認証情報](./basic-intro/ex2.md) | 実施 | 実施。サーバのログイン情報は演習4のあと |
| [3 インベントリと疎通](./basic-intro/ex3.md) | 実施。Section 3 は飛ばす | Section 1から3を実施。ホスト登録は演習4のあと |
| [4 AWSでサーバを作る](./basic-intro/ex4.md) | **飛ばして演習5へ** | 実施。終わったら演習3の Section 4 へ戻る |
| [5 GitLab](./basic-controller/ex5.md) | 実施 | 実施。演習4で作ったプロジェクトに残りのファイルを足す |
| [6 Inventory Source](./basic-controller/ex6.md) | 実施 | 実施 |
| [7 Job Template](./basic-controller/ex7.md) | 実施 | 実施 |
| [8 Survey](./basic-controller/ex8.md) | 実施 | 実施 |
| [9 Vault](./basic-controller/ex9.md) | 実施 | 実施 |
| [10 スケジュール](./basic-controller/ex10.md) | 実施 | 実施 |
| [11 Workflow](./basic-controller/ex11.md) | 実施 | 実施 |
| [応用1 RBAC](./advanced/ex1.md) | 実施 | 実施 |
| [応用2 通知](./advanced/ex2.md) | 環境内のメールサーバ | S3に書いてログで確認 |
| [応用3 カスタムEE](./advanced/ex3.md) | イメージを取得できるときだけ | AAPサーバで実施 |
| [応用4 API](./advanced/ex4.md) | 実施 | 実施 |
| [応用5 Playbookから起動](./advanced/ex5.md) | 実施 | 実施 |
| [応用6 Webhook](./advanced/ex6.md) | 実施 | 実施 |
| [応用7 EDA](./advanced/ex7.md) | 実施 | 実施 |
| [応用8 EDAからJob起動](./advanced/ex8.md) | 実施 | 実施 |

サーバがすでにある環境では、演習中にインターネットへ出ません。コレクションとコンテナイメージは、AAP に入っているものを使います。AWS でサーバを作る場合は、インターネットへ出られます。

作るファイルは次の2か所です。ディレクトリは Playbook が作ります。

| OS | ファイル |
|---|---|
| Linux | `/tmp/<name_prefix>-aap-training/note.txt` |
| Windows | `C:\temp\<name_prefix>-aap-training\note.txt` |

`<name_prefix>` は演習1で決める接頭辞です。人ごとにディレクトリが分かれるので、同じサーバを複数人で使えます。

Windows への接続方式は PSRP です。ドメインに参加していない Windows（AWS で作るサーバを含む）は NTLM、ドメインに参加している Windows は Kerberos を使います。

## 対象バージョン

* AAP 2.5 以降（統合UI: **Automation Execution** / **Automation Decisions** / **Access Management**）
* SCM: GitLab。ファイルの編集は Web IDE を使います
* カスタム Execution Environment を作る場合は、AAP サーバの `/tmp/<name_prefix>-custom-ee` で行います
* 同じ AAP を複数人で使うときは、演習1で決めた接頭辞をオブジェクト名の先頭に付けます。学習後の削除は [削除手順](./cleanup.md) です

## 画面の見方

→ [画面ラベル対応表](./ui-labels.md)

## 用語集

→ [AAP用語集](./aap_glossary.md)

## サンプル

→ [sample-content/](./sample-content/)

GitLab のプロジェクトへ、Web IDE で置いて使います。
