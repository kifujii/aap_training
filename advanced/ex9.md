# 応用演習 9 - GitLabのWebhookとCI

[応用演習8](./ex8.md) | [宿題へ](../homework/hm1.md)

GitLab へのコミットを契機に、AAP のプロジェクト同期を走らせます。CI の節は、GitLab Runner がコンテナイメージを取得できる場合だけ実施します。取得できない場合は Section 1 だけで終わりです。

## Section 1: pushで同期する

`aap-training-content` プロジェクトの編集で、Webhook を有効にして保存します。表示された **Webhook URL** と **Webhook Key** を控えます。

GitLab の **Settings > Webhooks** を開きます。

| 項目 | 入力値 |
|---|---|
| URL | 控えた Webhook URL |
| Secret token | 控えた Webhook Key |
| Trigger | Push events |

**Add webhook** をクリックします。

動作確認は Web IDE で行います。`group_vars/all.yml` の `training_note` を `managed by AAP` のままにしておき、行末に空白を1つ足すなど、意味が変わらない変更でコミットします。数秒後、AAP のプロジェクトのジョブ履歴に、ボタンを押していない同期が増えていれば完了です。

GitLab の webhook の Recent Deliveries が 2xx であることも確認します。失敗している場合は、URL と Secret token を見比べます。

終わったら `training_note` が `managed by AAP` であることを確認します。

<details>
<summary>git CLIで行う場合</summary>

```bash
git add group_vars/all.yml
git commit -m "Touch training_note to test the webhook"
git push
```

</details>

## Section 2: ansible-lint（Runnerがイメージを取得できる場合）

Web IDE で `.gitlab-ci.yml` を作ります。

```yaml
stages:
  - lint

ansible-lint:
  stage: lint
  image: python:3.12-slim
  before_script:
    - pip install ansible-lint
  script:
    - ansible-lint write_note_linux.yml write_note_windows.yml
  rules:
    - if: '$CI_PIPELINE_SOURCE == "push"'
```

コミットし、**CI/CD > Pipelines** で `ansible-lint` が成功することを確認します。

<details>
<summary>git CLIで行う場合</summary>

```bash
git add .gitlab-ci.yml
git commit -m "Add ansible-lint"
git push
```

</details>

## Section 3: CIからJobを起動する（同じ条件のときだけ）

GitLab の **Settings > CI/CD > Variables** に、次を追加します。値はマスクします。

| 変数 | 値 |
|---|---|
| `AAP_TOKEN` | 応用演習5の API トークン |
| `AAP_API_ROOT` | `https://<AAPのURL>/api/controller/v2` |

`.gitlab-ci.yml` を次に置き換えます。`7` は `write-note-linux` の ID です。

```yaml
stages:
  - lint
  - run

ansible-lint:
  stage: lint
  image: python:3.12-slim
  before_script:
    - pip install ansible-lint
  script:
    - ansible-lint write_note_linux.yml write_note_windows.yml

trigger-write-note:
  stage: run
  image: curlimages/curl:latest
  script:
    - |
      curl -sk -X POST \
        -H "Authorization: Bearer ${AAP_TOKEN}" \
        -H "Content-Type: application/json" \
        -d '{"extra_vars": {"training_note": "from gitlab ci"}, "limit": "lin1"}' \
        "${AAP_API_ROOT}/job_templates/7/launch/"
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
```

コミットすると、lint のあと `write-note-linux` が起動します。`lin1` の `note:` が `from gitlab ci` になれば完了です。確認後、Survey で `managed by AAP` に戻します。

## 次

[宿題](../homework/hm1.md) で、ここまでの操作を1つの手順にまとめます。
