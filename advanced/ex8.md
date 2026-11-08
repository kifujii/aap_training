# 応用演習 8 - EDAからJobを起動する

[応用演習7](./ex7.md) | [応用演習9へ](./ex9.md)

`status` が `success` のイベントを受けたら、`verify-note-linux` を起動します。

## Section 1: Rulebookを書き換える

Web IDE で `rulebooks/notify_on_webhook.yml` を、次の内容に置き換えます。組織名は演習1で控えた名前にします。

```yaml
---
- name: Run verify when the event says success
  hosts: all

  sources:
    - ansible.eda.webhook:
        host: 0.0.0.0
        port: 5000

  rules:
    - name: Launch verify-note-linux
      condition: event.payload.status == "success"
      action:
        run_job_template:
          name: verify-note-linux
          organization: Default
          job_args:
            limit: "{{ event.payload.host | default(omit) }}"
```

コミットして、プロジェクトを同期します。

<details>
<summary>git CLIで行う場合</summary>

```bash
git add rulebooks/notify_on_webhook.yml
git commit -m "Launch verify-note-linux from the rulebook"
git push
```

</details>

## Section 2: 決定環境

**Automation Decisions > 決定環境** を開き、一覧にある既定の Decision Environment の名前を控えます。この演習では新しくは作りません。

## Section 3: Controllerへ接続する認証情報

Automation Decisions の認証情報、または共通の認証情報で、タイプ `Red Hat Ansible Automation Platform` を作ります。

| 項目 | 入力値 |
|---|---|
| 名前 | `eda-controller` |
| Host | AAP の URL |
| OAuth トークン | [応用演習5](./ex5.md) で発行した API トークン |

ユーザー名とパスワードの欄がある場合は、ジョブを起動できるユーザーでも構いません。トークンを入れる欄がある場合は、トークンを使います。

## Section 4: Event Stream

**Automation Decisions > Event Streams** で追加します。

| 項目 | 入力値 |
|---|---|
| 名前 | `check-webhook-stream` |
| タイプ | Webhook |

保存後に表示される URL と、呼び出し用の認証情報を控えます。再表示できない値は、その場でメモします。

## Section 5: Activation

**Automation Decisions > ルールブックのアクティベーション** で追加します。

| 項目 | 入力値 |
|---|---|
| 名前 | `note-webhook-activation` |
| 決定環境 | Section 2 で控えたもの |
| プロジェクト | `aap-training-content` と同じ Git のプロジェクト。Decisions 側に無い場合は、同じ URL と `gitlab-scm` で作る |
| Rulebook | `rulebooks/notify_on_webhook.yml` |
| 認証情報 | `eda-controller` |
| Event Streams | `check-webhook-stream` |

状態が Running になることを確認します。

## Section 6: イベントを送る

```bash
curl -X POST "<Event StreamのURL>" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer <Event Streamの認証>" \
    -d '{"status":"success","host":"lin1"}'
```

**ジョブ** の一覧に `verify-note-linux` が増え、制限が `lin1` になっていれば完了です。Activation のログに、受信した JSON が出ていることも確認します。

終わったら `note-webhook-activation` を無効にします。無効にしないと、同じ URL へ送るたびにジョブが起動します。

## 次

[応用演習9](./ex9.md)
