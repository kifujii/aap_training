# 応用演習 8 - EDAからJobを起動する

[応用演習7](./ex7.md) | [削除手順へ](../cleanup.md)

`status` が `success` のイベントを受けたら、`<name_prefix>-verify-note-linux` を起動します。受け口は、応用演習7で作った Event Stream のままです。

## Section 1: Rulebookを書き換える

Web IDE で `rulebooks/notify_on_webhook.yml` を、次の内容に置き換えます。`<name_prefix>` は演習1の接頭辞に置き換えます。`organization` は演習1で控えた組織名にします。`sources` の名前は、Event Stream `<name_prefix>-check-webhook-stream` と同じです。`ansible.eda.webhook: {}` の行は、応用演習7で書いたものを残します。

```yaml
---
- name: Run verify when the event says success
  hosts: all

  sources:
    - name: <name_prefix>-check-webhook-stream
      ansible.eda.webhook: {}

  rules:
    - name: Launch <name_prefix>-verify-note-linux
      condition: event.payload.status == "success"
      action:
        run_job_template:
          name: <name_prefix>-verify-note-linux
          organization: Default
          job_args:
            limit: "{{ event.payload.host | default(omit) }}"
```

コミットします。Controller のプロジェクトと、Decisions のプロジェクトの両方を同期します。

<details>
<summary>git CLIで行う場合</summary>

```bash
git add rulebooks/notify_on_webhook.yml
git commit -m "Launch <name_prefix>-verify-note-linux from the rulebook"
git push
```

</details>

## Section 2: Activationを更新する

`<name_prefix>-eda-controller` は応用演習7で作ってあります。URL が `https://<AAPのURL>/api/controller/` であることを確認します。

`<name_prefix>-note-webhook-activation` を一度無効にし、再び有効にします。ルールブックを書き換えたあとに再起動すると、同期した内容で待ち受けます。状態が Running になることを確認します。すぐに失敗して `/api/v2/config/` と出ている場合は、`<name_prefix>-eda-controller` の URL を見直してから再起動します。

## Section 3: イベントを送る

`<name_prefix>-verify-note-linux` の編集画面を開き、**起動プロンプト** の制限にチェックを入れて保存します。このチェックが無いと、Rulebook が渡した制限は無視され、ジョブの制限は空のままです。

```bash
curl -X POST "<Event StreamのURL>" \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer <応用演習7で控えたToken>" \
    -d '{"status":"success","host":"lin1"}'
```

**ジョブ** の一覧に `<name_prefix>-verify-note-linux` が増え、制限が `lin1` になっていれば完了です。Activation のログに、ジョブを起動した記録が出ていることも確認します。

終わったら `<name_prefix>-note-webhook-activation` を無効にします。無効にしないと、同じ URL へ送るたびにジョブが起動します。

## 次

[削除手順](../cleanup.md) で自分のオブジェクトを消します。
