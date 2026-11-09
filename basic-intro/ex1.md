# 演習 1 - AAPへのログイン

> **注意:** この演習はすべてWebブラウザ上のAAP画面で行います。[事前準備](../prepare_training_environment.md)で確認したAAPのURL・ログイン情報を用意してください。

この演習では、AAP（Ansible Automation Platform）にログインし、次の演習で使うメニューと組織名を確認します。

## Section 1: ログイン

### Step 1: ログイン画面を開く

ブラウザで、事前準備で確認したAAPのURLにアクセスします。

```
https://<AAPのURL>/
```

ログイン画面が表示されたら、配布されたユーザー名とパスワードを入力してログインしてください。

### Step 2: ログイン後の画面を確認

ログインに成功すると、**Overview** が表示されます。見出しは「Welcome to the Ansible Automation Platform」です。左側にメニューがあることを確認してください。

![ログイン直後の Overview。左にメニュー、中央にホストとプロジェクトの件数](../images/aap/overview.png)

黄色い帯はサブスクリプション期限の通知です。手順とは関係ありません。

メニュー名とボタン名は [画面ラベル対応表](../ui-labels.md) にまとめてあります。以降の演習も、この対応表の表記で操作します。

**Note:** ログインできない場合は、URL、ユーザー名とパスワードの大文字小文字、VPN や社内ネットワークが必要かを確認してください。

## Section 2: 左メニュー

左メニューは3つのグループに分かれています。この演習で開くのは **Automation Execution** と **Access Management** です。

| グループ | この資料で使うもの |
|---|---|
| **Automation Execution** | ジョブ、テンプレート、スケジュール、プロジェクト。Infrastructure の中にインベントリー、実行環境、認証情報 |
| **Automation Decisions** | [応用演習7](../advanced/ex7.md) 以降 |
| **Access Management** | 組織。チームと API Tokens は応用演習 |

### Step 1: Automation Executionを開く

**Automation Execution** を開き、次があることを確認します。

* **ジョブ**
* **テンプレート**
* **スケジュール**
* **プロジェクト**
* **Infrastructure** の中の **インベントリー**、**実行環境**、**認証情報**

一覧が空とは限りません。Demo Project や Demo Job Template のようなサンプルが最初から入っている環境があります。演習ではそれらを編集せず、別名で新規作成します。

### Step 2: 組織名を控える

**Access Management > 組織** を開きます。以降の作成画面で選ぶ組織名を控えてください。1つだけの環境では `Default` であることが多いです。

オブジェクトは、この組織のどれか1つに属します。以降の手順で「組織を選択」と書いたときは、ここで控えた名前を選びます。

誰にどの操作を許すかは [応用演習1](../advanced/ex1.md) で扱います。この演習では確認しません。

## 次

[演習2](./ex2.md) で、接続に使う認証情報を作ります。
