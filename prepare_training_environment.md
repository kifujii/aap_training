# 事前準備（確認リスト）

本資料は、以下がすべて**構築済み・準備済み**であることを前提としています。研修開始前に、各受講者・各環境担当者で確認しておいてください。

> **Note:** このドキュメントは環境の構築手順ではありません。構築済みの環境情報を確認し、演習で使う値を控えるためのチェックリストです。

## 1. AAP（Ansible Automation Platform）

| 確認項目 | 控える値（例） |
|---|---|
| AAPのURL | `https://aap.example.local/` |
| ログインユーザー名 / パスワード | （各受講者に配布されたもの） |
| 対象バージョン | 2.5 以降（統合UI） |
| ログイン後に使用するOrganization名 | 例: `Default` または研修用に用意された専用Organization名 |

### 動作確認

1. ブラウザでAAPのURLにアクセスし、ログインできることを確認してください。
2. ログイン後、画面左側のナビゲーションに **Automation Execution** / **Automation Decisions** / **Access Management** のメニューが表示されていることを確認してください（[基礎演習1](./basic-intro/ex1.md)で詳しく扱います）。

> AAP自体の構築（インストーラ実行、ライセンス登録等）やユーザー作成は本資料の対象外です。未構築の場合は、環境管理者に事前構築を依頼してください。

## 2. 管理対象ノード

本資料の演習では、Web基盤（nginx）とロードバランサー（haproxy）を想定したサンプル構成を扱います。実際のホスト名・IPアドレスはお使いの環境に合わせて置き換えてください。

| ロール | 本資料での表記例 | 置き換え先（実際の値） |
|---|---|---|
| Webサーバー | `web1` (`192.0.2.11`) | |
| Webサーバー | `web2` (`192.0.2.12`) | |
| Webサーバー | `web3` (`192.0.2.13`) ※演習2で追加 | |
| ロードバランサー | `lb` (`192.0.2.14`) | |

### 確認事項

* 各ノードにSSH接続できるユーザー（例: `ansible`）が作成済みであること
* そのユーザーで `sudo`（権限昇格）が可能であること
* AAP（Execution Node）から各ノードへSSH（22番ポート）が到達可能であること
* 認証方式（SSH鍵 or パスワード）を確認しておくこと（[基礎演習2](./basic-intro/ex2.md)でCredentialとして登録します）

> 管理対象ノードが未構築の場合は、環境管理者に事前構築を依頼してください。本資料ではコンテナ・VM等の構築手順は扱いません。

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
