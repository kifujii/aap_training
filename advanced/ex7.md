# 応用演習 7 - EDA入門

[応用演習6](./ex6.md) | [応用演習8へ](./ex8.md)

これまでのジョブは、人が起動するか、スケジュールで起動するものでした。EDA は、受け取ったイベントの内容に応じて処理を始めます。

| 用語 | 役割 |
|---|---|
| Rulebook | イベントの受け口と、条件に対する処理を書いた YAML |
| Decision Environment | Rulebook を動かすコンテナイメージ |
| Rulebook Activation | Rulebook を動かし続ける設定 |

## Section 1: RulebookをWeb IDEで置く

`aap-training-content` の Web IDE で `rulebooks/notify_on_webhook.yml` を作り、次を貼ります。

```yaml
---
- name: Record a webhook
  hosts: all

  sources:
    - ansible.eda.webhook:
        host: 0.0.0.0
        port: 5000

  rules:
    - name: Log every event
      condition: true
      action:
        debug:
          msg: "イベントを受信しました: {{ event.payload }}"
```

**Commit to main** でコミットし、AAP のプロジェクトを同期します。

<details>
<summary>git CLIで行う場合</summary>

```bash
mkdir -p rulebooks
# 上の内容を rulebooks/notify_on_webhook.yml に保存してから
git add rulebooks/notify_on_webhook.yml
git commit -m "Add sample rulebook"
git push
```

</details>

## Section 2: ローカルでの起動は任意

AAP サーバに `ansible-rulebook` がある場合だけ、次で受信を試せます。無い場合は [応用演習8](./ex8.md) の画面上の Activation で確認します。

```bash
ansible-rulebook --rulebook rulebooks/notify_on_webhook.yml -i inventory.yml --verbose
```

別の端末から:

```bash
curl -X POST http://localhost:5000/endpoint \
    -H "Content-Type: application/json" \
    -d '{"status":"success","host":"lin1"}'
```

ログに `lin1` が出れば、Rulebook はイベントを受け取っています。確認が終わったら、起動したプロセスは止めます。

## 次

[応用演習8](./ex8.md) で、同じイベントから `verify-note-linux` を起動します。
