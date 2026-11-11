# 学習後に自分のリソースを削除する

自分の接頭辞が付いたものだけを消します。他の人のオブジェクトと、Demo で始まるサンプルは消しません。接頭辞は演習1で決めた文字列です。例は `tanaka01` です。以降の `<name_prefix>` は、その文字列に置き換えます。

## Section 1: AWSのサーバを消す

演習4でサーバを作った場合だけ、先にこれを実行します。認証情報 `<name_prefix>-aws-training` を消す前に終わらせます。

ジョブテンプレート `<name_prefix>-remove-aws-servers` が無い場合は、`<name_prefix>-create-aws-servers` と同じ項目で作り、Playbook だけ `remove_aws_servers.yml` にします。**Extra variables** の `resource_prefix` は、作成時と同じ接頭辞です。パスワードは不要です。インベントリーは `<name_prefix>-aws-runner`、認証情報は `<name_prefix>-aws-training` です。

起動して成功すれば、その接頭辞の VPC、サブネット、インスタンス、S3 バケットは削除されています。

## Section 2: AAPのオブジェクトを一括で消す

Web IDE で、プロジェクト `<name_prefix>-aap-training-content` に [sample-content/remove_my_lab.yml](./sample-content/remove_my_lab.yml) を置きます。中の `{{ name_prefix }}` は書き換えません。

**Commit to main** のあと、AAP のプロジェクトを同期します。

認証情報 `<name_prefix>-aap-api` が無い場合は作ります。タイプは `Red Hat Ansible Automation Platform` です。

| 項目 | 入力値 |
|---|---|
| 名前 | `<name_prefix>-aap-api` |
| Red Hat Ansible Automation Platform | `https://<AAPのURL>` |
| OAuth Token | 下で発行した API トークン |
| Verify SSL | チェックを外す |

トークンは **Access Management > API Tokens** の **Create API token** で作ります。Scope は `Write`、ボタンは **Create token** です。文字列は再表示できないので、その場で控えます。Username と Password は空欄です。URL の末尾に `/api/controller/` は付けません。

インベントリー `<name_prefix>-aap-runner` が無い場合は作り、ホスト `localhost` の **変数** に次を入れます。

```yaml
ansible_connection: local
```

**ジョブテンプレートの作成** で次を作ります。表に無い欄は空欄です。

| 項目 | 入力値 |
|---|---|
| 名前 | `<name_prefix>-remove-lab` |
| インベントリー | `<name_prefix>-aap-runner` |
| プロジェクト | `<name_prefix>-aap-training-content` |
| Playbook | `remove_my_lab.yml` |
| 実行環境 | 空欄 |
| 認証情報 | `<name_prefix>-aap-api` |
| Extra variables | 下の1行 |

```yaml
name_prefix: tanaka01
```

`tanaka01` は自分の接頭辞にします。起動画面には Extra variables は出ないので、編集画面で保存してから **テンプレートの起動** を押します。

成功すると、出力の最後に消したオブジェクト名の一覧が出ます。このジョブは、最後に `<name_prefix>-remove-lab`、`<name_prefix>-aap-runner`、`<name_prefix>-aap-api` も消します。作っていない名前は一覧に出ません。ジョブの終盤で失敗した場合は、Section 3 の検索で残っている行を消します。

使った API トークンは、**Access Management > API Tokens** で削除します。自分が今発行したものだけを消します。

Activation の削除で失敗した場合は、**Automation Decisions > ルールブックのアクティベーション** で `<name_prefix>-note-webhook-activation` を無効にしてから、同じジョブをもう一度起動します。

## Section 3: 画面で残りを確認する

各一覧の検索欄に、自分の接頭辞とハイフン（例: `tanaka01-`）を入れます。

| 画面 | 残っていたら |
|---|---|
| Automation Execution > テンプレート | その行を削除する |
| Automation Execution > スケジュール | その行を削除する |
| Automation Execution > プロジェクト | その行を削除する |
| Automation Execution > Infrastructure > インベントリー | その行を削除する |
| Automation Execution > Infrastructure > 認証情報 | その行を削除する |
| Automation Execution > Infrastructure > 実行環境 | `<name_prefix>-custom-ee` だけ削除する。Default execution environment は残す |
| Automation Execution > 管理 > Notifiers | その行を削除する |
| Access Management > チーム | `<name_prefix>-ops-team` を削除する |
| Automation Decisions > ルールブックのアクティベーション | 無効にしてから削除する |
| Automation Decisions > Event Streams | その行を削除する |
| Automation Decisions > プロジェクト | その行を削除する |
| Automation Decisions > Infrastructure > 認証情報 | その行を削除する |

検索が空なら、AAP 上の自分の演習オブジェクトは残っていません。

## Section 4: GitLabのプロジェクトを消す

GitLab のプロジェクト `<name_prefix>-aap-training-content` を開きます。**Settings > General** の **Advanced** を展開し、**Delete project** まで進みます。確認欄には、画面が求めるプロジェクト名を入力して削除します。Webhook もこのプロジェクトと一緒に消えます。

## Section 5: AAPサーバの作業ファイルを消す

演習9で暗号化した人、または応用演習3でイメージを作った人だけ、AAP サーバへ SSH します。`<name_prefix>` は自分の接頭辞に置き換えてから実行します。

```bash
cd /tmp
rm -f '/tmp/<name_prefix>-secrets.yml'
rm -rf '/tmp/<name_prefix>-custom-ee'
podman rmi 'localhost/<name_prefix>-custom-ee:1.0'
```

イメージが無い、という表示なら、そのコマンドはそこまでです。`Default execution environment` のイメージは消しません。
