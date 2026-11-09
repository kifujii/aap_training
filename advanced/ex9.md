# 応用演習 9 - GitLabのWebhookとCI

[応用演習8](./ex8.md) | [宿題へ](../homework/hm1.md)

GitLab の push を Event Stream で受けます。プロジェクト編集画面のオプションは、Clean、削除、Track submodules、Update revision on launch、ブランチの上書き許可です。push の記録は Event Stream の Events received に残ります。プロジェクトの同期は、ジョブの起動時更新か、プロジェクト画面の同期ボタンで行います。

CI の節は、GitLab Runner がコンテナイメージを取得できる場合だけ実施します。取得できない場合は Section 1 だけで終わりです。

## Section 1: pushをEvent Streamで受ける

応用演習8の curl は `Authorization` ヘッダを使います。GitLab の Webhook は秘密の文字列を `X-Gitlab-Token` で送るので、認証情報のタイプが違います。

**Automation Decisions > Infrastructure > 認証情報** で、タイプ `GitLab Event Stream` を作ります。一覧に無いときは、検索欄に `GitLab Event` と入れます。

| 項目 | 入力値 |
|---|---|
| 名前 | `gitlab-push-token` |
| Token | 自分で決めた文字列。次の Secret token に、同じ文字列を入れます |

**Automation Decisions > Event Streams** で追加します。

| 項目 | 入力値 |
|---|---|
| 名前 | `gitlab-push-stream` |
| Event stream type | `GitLab Event Stream` |
| 認証情報 | `gitlab-push-token` |

保存後の詳細に出る URL を控えます。

GitLab の **Settings > Webhooks** を開きます。

| 項目 | 入力値 |
|---|---|
| URL | 控えた Event Stream の URL |
| Secret token | 認証情報の Token と同じ文字列 |
| Trigger | Push events |

**Add webhook** をクリックします。

動作確認は Web IDE で行います。`group_vars/all.yml` の `training_note` は `managed by AAP` のままにして、行末に空白を1つ足すなど、値の変わらない変更でコミットします。数秒後、`gitlab-push-stream` の Events received が増えていれば完了です。プロジェクトのジョブ履歴に、ボタンを押していない同期は増えません。

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
