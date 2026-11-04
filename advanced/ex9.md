# 応用演習 9 - EDA入門

この演習では、Event-Driven Ansible (EDA) の基本概念と、Rulebookの基本構造を学びます。実際にController連携まで行うのは[応用演習10](./ex10.md)です。

**前提条件:** 基礎演習1〜7を完了していること

[前に戻る](./ex8.md) | [次へ進む](./ex10.md)

---

## Section 1: EDAとは

これまでの演習で扱ったJob Template・Workflow Job Templateは、いずれも「人（またはSchedule）がJobを起動する」ことが前提でした。EDAは、外部からのイベント（Webhook通知、監視アラート、メッセージキューへの投入など）を検知し、条件に応じて**自動的にアクションを実行する**仕組みです。

| 概念 | OSS版Ansible/AAPの用語との対応 |
|---|---|
| **Rulebook** | Playbookに相当する、YAML形式の定義ファイル |
| **Decision Environment (DE)** | Execution Environmentに相当する、Rulebook実行用のコンテナイメージ |
| **Rulebook Activation** | Job Templateの「実行中インスタンス」に相当する、Rulebookを稼働状態にしたもの |

## Section 2: Rulebookの基本構造

Rulebookは大きく `sources`（イベントの入力元）と `rules`（条件とアクション）で構成されます。

```yaml
---
- name: Webhookイベントを受信して記録する
  hosts: all

  sources:
    - ansible.eda.webhook:
        host: 0.0.0.0
        port: 5000

  rules:
    - name: デプロイ完了イベントを記録する
      condition: event.payload.status == "success"
      action:
        debug:
          msg: "デプロイ完了イベントを受信しました: {{ event.payload }}"
```

| 要素 | 説明 |
|---|---|
| `sources` | イベントの受信方法を定義。今回の例では `ansible.eda.webhook`（HTTP Webhookの受信）を使用 |
| `rules` | 1つ以上のルールを定義。各ルールは `condition`（発火条件）と `action`（実行する処理）を持つ |
| `event.payload` | 受信したWebhookのJSONボディ。`condition` や `action` の中から参照できる |

## Section 3: サンプルRulebookの作成

### Step 1: eda.decision_environments collectionのインストール（ローカル確認用）

```bash
$ ansible-galaxy collection install ansible.eda
```

### Step 2: Rulebookファイルを作成する

[応用演習4](./ex4.md)までで使用した `aap-training-content` リポジトリに、Rulebook用のディレクトリを追加します。

```bash
$ cd aap-training-content
$ mkdir -p rulebooks
$ cat << 'EOF' > rulebooks/notify_on_webhook.yml
---
- name: Webhookイベントを受信して記録する
  hosts: all

  sources:
    - ansible.eda.webhook:
        host: 0.0.0.0
        port: 5000

  rules:
    - name: 任意のイベントをログに出力する
      condition: true
      action:
        debug:
          msg: "イベントを受信しました: {{ event.payload }}"
EOF
```

### Step 3: GitLabへpushする

**方法A: git CLIで行う場合（基本）**

```bash
$ git add rulebooks/
$ git commit -m "Add sample rulebook for EDA training"
$ git push
```

**方法B: GitLab Web IDEで行う場合（代替）**

1. GitLab Project画面で「Web IDE」を開く
2. `rulebooks/notify_on_webhook.yml` を新規作成し、Step 2で作成した内容を貼り付ける
3. 「Source Control」アイコンからコミットメッセージ（例: `Add sample rulebook for EDA training`）を入力し、「Commit to main」をクリックする

## Section 4: ローカルでの動作確認（任意）

AAPに登録する前に、`ansible-rulebook` コマンドでローカル動作を確認できます（EEビルド用サーバ等、`ansible-rulebook` が導入済みの環境で実施）。

> **Note:** このSectionはコマンド実行が前提のため、**GitLab Web IDEでは代替できません**。Web IDEのみで進めている場合は、このSectionは読み物として確認し、[応用演習10](./ex10.md)でAAP上のRulebook Activationとして動作確認してください。

### Step 1: 導入する

```bash
$ python3 -m pip install --user ansible-rulebook
```

### Step 2: Rulebookを起動する

```bash
$ ansible-rulebook --rulebook rulebooks/notify_on_webhook.yml -i inventory.yml --verbose
```

### Step 3: 別のターミナルからWebhookを送信する

```bash
$ curl -X POST http://localhost:5000/endpoint \
    -H "Content-Type: application/json" \
    -d '{"status": "success", "host": "srv1"}'
```

Rulebookを起動したターミナルに、`debug` アクションで出力したメッセージが表示されることを確認してください。

```
{"msg": "イベントを受信しました: {'status': 'success', 'host': 'srv1'}"}
```

確認が終わったら、`ansible-rulebook` プロセスを `Ctrl+C` で停止してください。

---

ここまでで、Rulebookの基本構造とローカルでの動作確認方法を理解しました。次の演習では、このRulebookをAAP上の **Rulebook Activation** として稼働させ、Job Templateの自動起動まで連携させます。

[次へ進む](./ex10.md)
