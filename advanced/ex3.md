# 応用演習 3 - ScheduleとNotification

この演習では、Job Templateを定期実行する **Schedule** と、Jobの結果に応じて通知を送る **Notification Template** の設定方法を学びます。

**前提条件:** 応用演習1〜2を完了していること

[前に戻る](./ex2.md) | [次へ進む](./ex4.md)

---

## Section 1: Scheduleの作成

### Step 1: Schedulesタブを開く

`deploy-web-stack` Job Templateの詳細画面 →「Schedules」タブ →「Add」をクリックします。

### Step 2: 入力する

| 項目 | 入力値 |
|---|---|
| Name | `daily-config-check` |
| Start Date/Time | 任意の開始日時 |
| Repeat Frequency | `Day`（毎日実行） |
| Run Every | `1` |

AAPの画面では、Cron形式（`rrule`）を直接意識せずGUIで繰り返し設定を組み立てられますが、内部的にはOSS版の `cron` と同様の考え方（曜日・時刻・間隔）で管理されています。

「Save」をクリックして保存します。

### Step 3: 確認する

Schedule一覧に `daily-config-check` が表示され、「Next Run」に次回実行予定時刻が表示されていることを確認してください。

**Note:** 研修中に実際の定期実行を待つ必要はありません。「Next Run」が正しく計算されていることと、一覧に表示されることを確認できれば十分です。確認後は、誤って本番稼働させないよう、Scheduleを一時的に無効化（Toggle OFF）しておきましょう。

## Section 2: Notification Templateの作成

AAPは、Jobの開始・成功・失敗、Workflow Approvalの承認待ちなど、さまざまなイベントをトリガーに通知を送信できます。

### Step 1: Notification Template作成画面を開く

**Access Management > Administration > Notifications**（配置はバージョンにより異なる場合があります）を開き、「Add」をクリックします。

### Step 2: 入力する（Webhookの例）

| 項目 | 入力値 |
|---|---|
| Name | `job-failure-webhook` |
| Organization | 基礎演習1で確認したOrganization |
| Notification Type | `Webhook` |
| Target URL | 通知先のWebhook URL（Slack Incoming Webhook、社内チャットツール等） |
| HTTP Headers | 必要に応じて認証ヘッダーを設定 |

Email・Slack専用の設定項目を使う場合は、Notification Typeをそれぞれ `Email` / `Slack` に変更し、SMTPサーバー情報やWebhook URLを入力してください。

「Save」をクリックして保存します。

## Section 3: Notificationの割り当て

Notification Templateは作成しただけでは何も起こりません。どのオブジェクトの、どのイベントで通知するかを**割り当てる**必要があります。

### Step 1: Job Templateへの割り当て

`deploy-web-stack` の詳細画面 →「Notifications」タブを開きます。`job-failure-webhook` の「Error」列のトグルを有効化します。

| イベント | 説明 |
|---|---|
| Start | Job開始時に通知 |
| Success | Job成功時に通知 |
| Error | Job失敗時に通知 |

### Step 2: Organizationレベルでの割り当て（任意）

Organizationの詳細画面でも同様にNotificationを割り当てることができます。Organization単位で設定すると、配下のすべてのProject/Job Templateに一括適用されます。個別のJob Template単位の設定は、Organization単位の設定より優先されます。

## Section 4: 動作確認

### Step 1: 意図的に失敗させる

`deploy-web-stack` のCredentialを一時的に誤った値（存在しないCredential、または無効化したCredential）に変更し、「Launch」してJobを失敗させます。

### Step 2: 通知を確認する

設定した通知先（Webhook/Slack/Email）に、Jobの失敗を知らせる通知が届くことを確認してください。

### Step 3: 設定を元に戻す

確認後は、Credentialを `managed-nodes-ssh` に戻し、正常に実行できる状態に戻してください。

**Note:** 本番運用では、Error通知をSlack等の運用チャンネルに割り当てておくことで、Job失敗を即座に検知できます。[応用演習9〜10（EDA）](./ex9.md)では、通知を受け取るだけでなく、イベントに応じて自動的に別のアクションを実行する仕組みを扱います。

---

ここまでで、定期実行と通知の設定ができるようになりました。次の演習では、複数のJob Templateを連携させるWorkflow Job Templateを扱います。

[次へ進む](./ex4.md)
