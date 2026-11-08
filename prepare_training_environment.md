# 事前準備（確認リスト）

本資料は、以下がすべて**構築済み・準備済み**であることを前提としています。研修開始前に、各受講者・各環境担当者で確認しておいてください。

> **Note:** このドキュメントは環境の構築手順ではありません。構築済みの環境情報を確認し、演習で使う値を控えるためのチェックリストです。

## 1. AAP（Ansible Automation Platform）

| 確認項目 | 控える値（例） |
|---|---|
| AAPのURL | `https://aap.example.local/` |
| ログインユーザー名 / パスワード | （各受講者に配布されたもの） |
| 対象バージョン | 2.5 以降（統合UI） |
| ログイン後に使用する組織名 | 多くの環境では `Default`。研修用の専用組織がある場合はその名前 |

### 動作確認

1. ブラウザでAAPのURLにアクセスし、ログインできることを確認してください。
2. ログイン後、画面左側のナビゲーションに **Automation Execution** / **Automation Decisions** / **Access Management** のメニューが表示されていることを確認してください（[基礎演習1](./basic-intro/ex1.md)で詳しく扱います）。日本語UIの項目名は [画面ラベル対応表](./ui-labels.md) を見てください。一覧に Demo で始まるサンプルがあっても、演習では別名で新規作成します。

> AAP自体の構築（インストーラ実行、ライセンス登録等）やユーザー作成は本資料の対象外です。未構築の場合は、環境管理者に事前構築を依頼してください。

## 2. 管理対象ノード

演習で変更するのは、Linuxサーバの時刻同期、基本パッケージ、ログインバナー、SSHの接続制限、状態点検です。Webサーバの公開設定は扱いません。

サーバの用意は次のどちらかです。手順の詳細は [操作対象サーバ（AWS）](./aws_targets.md) にまとめています。

| 経路 | 使うとき | 演習開始前に控えるもの |
|---|---|---|
| **A. EC2を新規作成** | 操作対象がまだない | リージョン、VPC ID、Subnet ID、AMI ID、キーペア名、AAPからSSHを許可するCIDR。アクセスキーは [基礎演習2](./basic-intro/ex2.md) のCredentialに入れる |
| **B. 既存サーバ** | EC2やその他のLinuxがすでにある | インベントリ名 `srv1` / `srv2` に対応するIPまたはDNS名。台数は1台でもよい |

経路Aのインスタンス作成は [sample-content/provision_ec2.yml](./sample-content/provision_ec2.yml) で行います。AAP 2.7 の既定Execution Environmentには `amazon.aws` 10.3.0 が入っています。構築Jobの実行環境は既定のままで進めます。別のEEを指定する場合は、[操作対象サーバ（AWS）](./aws_targets.md) の版の説明を見てください。

サンプルのアドレスは `10.0.1.11` と `10.0.1.12` です。実際の値に置き換えてください。経路Aでは、作成Jobのログに出たプライベートIPが置き換え先です。

### 確認事項

* OSはRHEL系で、`dnf` と `chronyd` が使えること
* SSH接続できるユーザー（Amazon Linux / RHELのAMIでは多くの場合 `ec2-user`）がいること
* そのユーザーで `sudo`（権限昇格）ができること
* AAPのExecution Nodeから各ノードへSSH（22番ポート）が到達できること
* 認証方式（SSH鍵またはパスワード）を確認しておくこと（[基礎演習2](./basic-intro/ex2.md)でCredentialとして登録します）

## 3. GitLab

| 確認項目 | 控える値（例） |
|---|---|
| GitLabのURL | `https://gitlab.example.local/` |
| ログインユーザー名 / パスワード | （各受講者に配布されたもの） |
| 演習用Namespace（グループ） | 例: `aap-training` |

### 確認事項

* GitLabにログインできること
* 新規Project（リポジトリ）を作成できる権限があること
* Personal Access Token（`api` / `read_repository` スコープ）を発行できること（[基礎演習4](./basic-controller/ex4.md)で使用）

> GitLabアカウントが未作成の場合は、環境管理者にアカウント発行を依頼してください。

## 4. Execution Environment（EE）ビルド用サーバー

[応用演習6](./advanced/ex6.md)（カスタムEEの作成）でのみ必要です。それ以外の演習では不要です。

| 確認項目 | 内容 |
|---|---|
| OS | Linux（RHEL系推奨） |
| インターネット接続 | 必要（ベースイメージ・Collectionの取得のため） |
| 必須コマンド | `podman` または `docker`、`ansible-builder`、`ansible-navigator` |
| コンテナレジストリ | ビルドしたEEイメージをpushする先（社内レジストリ等） |

### 確認コマンド例

```bash
$ ansible-builder --version
$ ansible-navigator --version
$ podman --version
```

いずれかが未インストールの場合は、[応用演習6](./advanced/ex6.md)の冒頭で導入手順を案内します。

## 5. サンプルコンテンツの取得

本資料の演習（[基礎演習4](./basic-controller/ex4.md)以降）では、[sample-content/](./sample-content/) ディレクトリ内のPlaybook/Roleを、各自のGitLab Projectにpushして使用します。この研修資料リポジトリ自体をcloneしておいてください。

```bash
$ git clone <この研修資料リポジトリのURL>
$ cd aap_training/sample-content
```

具体的なpush手順は[基礎演習4](./basic-controller/ex4.md)で案内します。

---

準備ができたら、[基礎演習1](./basic-intro/ex1.md)から始めてください。
