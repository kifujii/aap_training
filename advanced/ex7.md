# 応用演習 7 - ジョブテンプレートのWebhook

[応用演習6](./ex6.md) | [応用演習8へ](./ex8.md)

ジョブテンプレートには、GitLab からの HTTP を受けて自分を起動する Webhook があります。ここでは `verify-note-linux` に付け、マージリクエストを作成したときにジョブが起動することを確認します。操作は Automation Execution のジョブテンプレートだけです。

マージリクエストを作っても、プロジェクトの同期は始まりません。ジョブが使う Playbook は、すでに同期済みの内容です。

## Section 1: ジョブテンプレートでWebhookを有効にする

`verify-note-linux` の **テンプレートの編集** を開きます。

**Webhook の有効化** にチェックを入れます。

| 項目 | 入力値 |
|---|---|
| Webhook service | `GitLab` |
| Webhook credential | 空欄 |

**ジョブテンプレートの保存** をクリックします。詳細画面に **Webhook URL** と **Webhook key** が出ます。両方を控えます。Webhook key は、GitLab の Secret token に入れる文字列です。

## Section 2: GitLabに受け口を登録する

GitLab のプロジェクト `aap-training-content` で **Settings > Webhooks** を開きます。

| 項目 | 入力値 |
|---|---|
| URL | Section 1 の Webhook URL |
| Secret token | Section 1 の Webhook key |
| Trigger | **Merge request events** だけオン |

Push events が最初からオンなら外します。**Add webhook** をクリックします。

追加された webhook の Recent Deliveries は、この時点では空で構いません。

## Section 3: マージリクエストを作る

Web IDE を開きます。

1. 画面左下のブランチ名 `main` をクリックし、**Create new branch...** で `webhook-check` を作って切り替えます
2. `group_vars/all.yml` の末尾に空行を1つ足します。`training_note` は `managed by AAP` のままにします
3. **Source Control** を開き、メッセージ `Trigger job from a merge request` でコミットします。コミット先は `webhook-check` です
4. 画面に **Create merge request** が出たら、それを開きます。出ない場合は、プロジェクトの **Merge requests** > **New merge request** を開き、Source branch を `webhook-check`、Target branch を `main` にして **Compare branches and continue**、続けて **Create merge request** を押します

## Section 4: ジョブを確認する

AAP の **ジョブ** を開きます。起動ボタンを押していない `verify-note-linux` が増えていれば、Webhook で起動しています。そのジョブの詳細を開き、Extra variables に `awx_webhook_payload` があること、その中の `object_kind` が `merge_request` であることを確認します。

ジョブが増えない場合は、GitLab の webhook の Recent Deliveries を見ます。失敗しているときは、URL と Secret token を、AAP の Webhook URL、Webhook key と見比べます。証明書のエラーなら、GitLab 側の **Enable SSL verification** を外して保存し、マージリクエストの説明に1文字足して保存します。Merge request events がもう一度送られます。

確認が終わったら、`verify-note-linux` の **Webhook の有効化** を外して保存します。外したあとは、同じ URL へ送ってもジョブは起動しません。

## 次

[応用演習8](./ex8.md) では、別の受け口として Event Stream を作ります。
