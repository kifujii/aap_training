# 宿題

基礎演習と応用演習で作ったものを、1台の Linux（`lin1`）と1台の Windows（`win1`）に対して、課題1から順に足していきます。各課題の「完了」を全部満たしたら次へ進みます。

サーバがすでにある場合は [演習4](../basic-intro/ex4.md) を使わず、手元のアドレスを `inventory.yml` に書きます。サーバが無い場合は演習4を済ませ、ログに出たアドレスを使います。`requirements.yml` を置くのは、AWS で作る場合だけです。

ファイルの編集は GitLab の Web IDE で行い、AAP のプロジェクトを同期してからジョブを起動します。

## 課題1 - note.txt を両方のOSに書く

### やること

1. `inventory.yml` の `lin1.ansible_host` と `win1.ansible_host` を、自分のアドレスにする。サンプルの `203.0.113.10` と `203.0.113.11` のままでは完了にしない
2. Windows の接続変数を、自分の環境に合わせる
   * ドメインに参加していない、または AWS で作った: `ansible_psrp_auth: ntlm`、ポート `5985`、プロトコル `http`
   * ドメインに参加している: `ansible_psrp_auth: kerberos`、ポート `5986`、プロトコル `https`。`krb5.conf` のレルムと KDC を自分の値にする
3. 認証情報 `linux-ssh` と `windows-psrp`、プロジェクト `aap-training-content`、インベントリー `training-inventory`、ソース `gitlab-inventory` を作る
4. ジョブテンプレート `write-note-linux` と `write-note-windows` を作り、それぞれ1回起動する
5. もう一度ずつ起動する

### 完了

* `lin1` の `/tmp/aap-training/note.txt` が、次の3行である。アドレスの行は、inventory に書いた値と一致する

```
hostname: lin1
address: <lin1 の ansible_host>
note: managed by AAP
```

* `win1` の `C:\temp\aap-training\note.txt` が、同じ形で1行目だけ `hostname: win1` である
* 2回目のジョブで、note.txt を書くタスクの `changed` が `0` である
* `verify-note-linux` と `verify-note-windows` が成功する

## 課題2 - ホストを1台増やす

増えたホストにも、同じ `note.txt` が書かれることを確認します。

### サーバがすでにある場合

3台目の Linux があるときは、Web IDE で `linux` グループに次を追加します。アドレスだけ自分の3台目にします。Playbook は変えません。

```yaml
lin2:
  ansible_host: 203.0.113.12
```

3台目が無い場合は、ホスト追加の代わりに次を行います。`inventory.yml` の `lin1` に、`ansible_host` と同じ階層で `note_owner: <自分の名前>` を足します。`write_note_linux.yml` の `note.txt` の `content` には、`note:` の次の行として次を足します。

```
owner: {{ note_owner | default('none') }}
```

再実行して、その行が `note.txt` に出れば、この読み替えの完了です。以降の完了条件の `lin2` は、`lin1` の `owner:` 行に読み替えます。

### AWSで作った場合

Web IDE で `create_aws_servers.yml` の Linux を作るタスクをコピーし、タスク名と `name` を変えます。

* タスク名: `Create the second Linux server`
* `name`: `"{{ resource_prefix }}-linux-2"`
* ユーザーデータはそのまま（ログインユーザーは同じ `ansible`）

`create-aws-servers` を、演習4と同じ `resource_prefix` とパスワードで再実行します。ジョブ末尾のまとめに出るアドレスは、1台目の Linux です。2台目のアドレスは、タスク `Create the second Linux server` の結果にある `public_ip_address` です。`inventory.yml` に `lin2` を足し、そのアドレスを `ansible_host` にします。

### 完了

* ソース `gitlab-inventory` を同期し、ホスト一覧に `lin2` がある
* `write-note-linux` を起動し、`lin1` と `lin2` の両方に `/tmp/aap-training/note.txt` がある
* `lin2` の1行目は `hostname: lin2` である
* `verify-note-linux` が `lin1` と `lin2` の両方で成功する

## 課題3 - WorkflowとSurvey

### やること

1. `write-note-linux` と `write-note-windows` の Survey に、質問 `ファイルに書くメモを入力してください`、変数 `training_note`、既定値 `managed by AAP` を作る
2. ワークフロー `note-workflow` を、次の順で作る
   * 開始から `write-note-linux` と `write-note-windows`
   * 両方の成功の先に承認ノード。名前 `note-review`。説明 `note.txt を確認したら承認してください`
   * 承認の成功から `verify-note-linux` と `verify-note-windows`
3. ビジュアライザーで `write-note-linux` のノードを開き、**Edit step** から **Survey** まで進む。`ファイルに書くメモを入力してください` に `homework note` と入れ、最後まで進めて保存する。`write-note-windows` のノードも同じ値にする。左上の **Save** を押す。ワークフローの起動画面には、この Survey は出ない
4. `note-workflow` を起動する
5. **Workflow Approvals** で `note-review` を承認する

### 完了

* 承認するまで verify のジョブが始まっていない
* 承認後、4つのジョブが成功し、ワークフローが成功で終わる
* `lin1` の note.txt の `note:` が `homework note` である
* `win1` の note.txt の `note:` も `homework note` である
* 確認後、2つの write ノードの Survey を `managed by AAP` に戻し、ワークフローをもう一度起動して承認してある

## 課題4 - 権限、Vault、通知

### RBAC

1. チーム `ops-team` を作る
2. `write-note-linux` と `write-note-windows` に、`ops-team` の **JobTemplate Execute** を付ける。`note-workflow` には **WorkflowJobTemplate Execute** を付ける
3. プロジェクト `aap-training-content` には **Project Read Compat**、インベントリー `training-inventory` には **Inventory Read Compat** だけを付ける
4. `ops-team` のユーザーでログインする

完了:

* そのユーザーは `write-note-linux` を起動できる
* そのユーザーはテンプレートの Playbook 名を変更できない

### Vault

1. AAP サーバへ SSH し、`cd /tmp` してから `ansible-vault encrypt_string` で値だけを暗号化する。平文の例は次の1行で、`TrainingSecret1` は自分の英数字に変えてよい。コマンドが聞くパスワードは、この文字列とは別の復号用パスワードにする

```bash
cd /tmp
ansible-vault encrypt_string 'TrainingSecret1' --name training_secret | tee /tmp/secrets.yml
```

2. `/tmp/secrets.yml` の先頭が `training_secret: !vault |` になっていることを確認し、その全文を Web IDE で `secrets.yml` としてコミットする。平文の `TrainingSecret1` はコミットしない。ホストに `ansible-vault` が無い場合は、演習9と同じく、名前に `controller` を含むコンテナで `encrypt_string` を実行し、画面に出た全文をコピーする
3. 認証情報 `vault-training` に、`encrypt_string` のときに入力したパスワードを入れる
4. `write-note-linux` と `write-note-windows` に `vault-training` を追加し、編集画面の **Extra variables** に `training_use_vault: true` を入れて保存してから起動する

完了:

* Linux の `/tmp/aap-training/secret.txt` が、自分が書いた `training_secret` の値と一致し、モードが `0600` である
* Windows の `C:\temp\aap-training\secret.txt` も同じ値である
* ジョブログに、その値がそのまま出ていない

### 通知

自分の環境に合わせて、どちらか一方を行います。

サーバがすでにある場合:

1. Notifier `note-mail` を作り、Notification type は `メール`、ホストは環境内のメールサーバ、Recipient list は自分が見られるアドレス、Sender email は差出人、Port は配布されたポート（無指定なら `25`）にする
2. `write-note-linux` の失敗時の通知に `note-mail` を付ける
3. 制限に `lin-missing` と入れて起動し、失敗させる
4. メールが届いたら、制限を空に戻して成功させる

完了: 失敗を知らせるメールが1通届き、その後のジョブは成功している。

AWS で作った場合:

1. ジョブテンプレート `notify-aws` を作る。インベントリーは `aws-runner`、Playbook は `notify_aws.yml`、認証情報は `aws-training`、**Extra variables** の `resource_prefix` は演習4と同じ値
2. **Extra variables** に、`resource_prefix` の行を残したまま次の行を足して保存する。起動画面にはこの欄は出ない

```yaml
notify_message: homework notification
```

3. 起動する。確認が終わったら、足した `notify_message` の行だけを消して保存する

完了: ログに `bucket:`、`object: notification.txt`、`contents: homework notification` の3つが出ている。

## 課題5 - スケジュール

### やること

1. `verify-note-linux` に、名前 `daily-verify-linux`、毎日 `07:00` のスケジュールを作る
2. `verify-note-windows` に、名前 `daily-verify-windows`、毎日 `07:00` のスケジュールを作る
3. 動作確認用に、開始時刻を数分後にした予定を1つ追加し、ジョブ一覧に verify が増えることを見る
4. 動作確認用の予定は削除し、毎日 07:00 の2つだけを残す

完了:

* スケジュール一覧に `daily-verify-linux` と `daily-verify-windows` があり、両方とも有効である
* 書く方の `write-note-linux` と `write-note-windows` にはスケジュールが無い

### 任意: EDA

[応用演習9](../advanced/ex9.md) まで終わっている場合だけ実施します。

Event Stream `check-webhook-stream` へ、次の JSON を送ります。

```json
{"status":"success","host":"lin1"}
```

完了: `verify-note-linux` が自動で始まり、制限が `lin1` である。確認後、Activation `note-webhook-activation` を無効にしてある。

## 暗号化前の見本

[example_config/secrets.yml.example](./example_config/secrets.yml.example) は、暗号化する前の形の見本です。課題4で GitLab に置く `secrets.yml` は、`encrypt_string` の出力だけにします。
