# 学習後に自分のリソースを削除する

[READMEへ](./README.md)

自分の接頭辞が付いたものだけを消します。他の人のオブジェクトと、Demo で始まるサンプルは消しません。接頭辞は演習1で決めた文字列です。以降の `<name_prefix>` は、その文字列に置き換えます。

## Section 1: AWSのサーバを消す

演習4でサーバを作った場合だけ、先にこれを実行します。認証情報 `<name_prefix>-aws-training` を消す前に終わらせます。

ジョブテンプレート `<name_prefix>-remove-aws-servers` が無い場合は、`<name_prefix>-create-aws-servers` と同じ項目で作り、Playbook だけ `remove_aws_servers.yml` にします。**Extra variables** の `resource_prefix` は、作成時と同じ接頭辞です。パスワードは不要です。インベントリーは `<name_prefix>-aws-runner`、認証情報は `<name_prefix>-aws-training` です。

起動して成功すれば、その接頭辞のインスタンス、セキュリティグループ、ルートテーブル、サブネット、インターネットゲートウェイ、VPC は削除されています。応用演習2の Mailpit と受け取ったメールも、Linux サーバと一緒に消えます。

## Section 2: サーバに書いたファイルを消す

既存サーバを使用した場合だけ行います。AWS でサーバを作った場合は、Section 1 でサーバごと消えているので、Section 3 へ進みます。

演習で作った `/tmp/<name_prefix>-aap-training`（Linux）と `C:\temp\<name_prefix>-aap-training`（Windows）を、中の `note.txt` と `secret.txt` ごと消します。他の人のディレクトリは消しません。

Web IDE で、プロジェクト `<name_prefix>-aap-training-content` に [sample-content/remove_my_notes.yml](./sample-content/remove_my_notes.yml) を置きます。中の `{{ name_prefix }}` は書き換えません。**Commit to main** のあと、AAP のプロジェクトを同期します。

**ジョブテンプレートの作成** で、次の2つを作ります。表に無い欄は空欄です。

| 項目 | Linux 用 | Windows 用 |
|---|---|---|
| 名前 | `<name_prefix>-remove-notes-linux` | `<name_prefix>-remove-notes-windows` |
| インベントリー | `<name_prefix>-training-inventory` | `<name_prefix>-training-inventory` |
| プロジェクト | `<name_prefix>-aap-training-content` | `<name_prefix>-aap-training-content` |
| Playbook | `remove_my_notes.yml` | `remove_my_notes.yml` |
| 実行環境 | 空欄 | 空欄 |
| 認証情報 | `<name_prefix>-linux-ssh` | `<name_prefix>-windows-psrp` |
| 制限 | `linux` | `windows` |
| Extra variables | `name_prefix: tanaka01` | `name_prefix: tanaka01` |

`tanaka01` は自分の接頭辞にします。認証情報が OS ごとに違うので、制限で片方のグループだけを対象にします。

両方を起動します。Linux 用は `Remove /tmp/<name_prefix>-aap-training` が `changed`、Windows 用は `Remove C:\temp\<name_prefix>-aap-training` が `changed` になれば消えています。もう一方の OS の play は、対象ホストが無いので `skipping` です。2回目に起動すると、どちらも `ok` になります。

## Section 3: AAPのオブジェクトを一括で消す

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

成功すると、出力の最後に2つの一覧が出ます。`Show the names that were deleted` は削除できた名前、`Show the names that remain` は見つかったのに削除できなかった名前です。作っていない名前は、どちらにも出ません。このジョブは、実行中の自分自身が使っているプロジェクト、インベントリー `<name_prefix>-aap-runner`、テンプレート `<name_prefix>-remove-lab` を消せません。この3つは「残った名前」に出ます。残った名前は Section 4 で、画面から消します。ジョブが失敗した場合も、Section 4 の検索で残っている行を消します。

このジョブは、もう一度起動しても動きません。1回目で GitLab の認証情報 `<name_prefix>-gitlab-scm` を消すので、2回目は起動時のプロジェクトの同期で失敗します。残ったものは、Section 4 の画面で消します。

使った API トークンは、**Access Management > API Tokens** で削除します。応用演習4で発行し、`<name_prefix>-aap-api` と `<name_prefix>-eda-controller` に入れたトークンと、このページで発行したトークンの両方です。他の人のトークンは、この一覧には出ません。

Activation を消したあとの Event Stream と EDA のプロジェクトは、Activation の削除が終わるまで「使用中」になります。ジョブは数回待ってからやり直します。それでも残ったときは、Section 4 の表の **Automation Decisions** の行で消します。

## Section 4: 画面で残りを確認する

各一覧の検索欄に、自分の接頭辞の直後へハイフンを付けた文字列を入れます。

| 画面 | 残っていたら |
|---|---|
| Automation Execution > ジョブ | 演習4の `<name_prefix>-create-aws-servers` のジョブには、Extra variables にパスワードが残っています。行にチェックを入れ、一覧の上の点が3つのボタン（**toolbar actions**）から **Delete jobs** で削除する。ほかの自分のジョブも、同じ方法で消せる |
| Automation Execution > テンプレート | その行を削除する |
| Automation Execution > スケジュール | その行を削除する |
| Automation Execution > プロジェクト | その行を削除する |
| Automation Execution > Infrastructure > インベントリー | その行を削除する |
| Automation Execution > Infrastructure > 認証情報 | その行を削除する |
| Automation Execution > Infrastructure > 実行環境 | `<name_prefix>-custom-ee` だけ削除する。Default execution environment は残す |
| Automation Execution > 管理 > Notifiers | その行を削除する |
| Access Management > チーム | `<name_prefix>-ops-team` を削除する |
| Access Management > ユーザー | `<name_prefix>-ops` を削除する |
| Automation Decisions > ルールブックのアクティベーション | 無効にしてから削除する |
| Automation Decisions > Event Streams | その行を削除する |
| Automation Decisions > プロジェクト | その行を削除する |
| Automation Decisions > Infrastructure > 認証情報 | その行を削除する |

検索が空なら、AAP 上の自分の演習オブジェクトは残っていません。

## Section 5: GitLabのプロジェクトを消す

GitLab のプロジェクト `<name_prefix>-aap-training-content` を開きます。**Settings > General** の **Advanced** を展開し、**Delete project** まで進みます。確認欄には、画面が求めるプロジェクト名を入力して削除します。Webhook もこのプロジェクトと一緒に消えます。GitLab の設定によっては、すぐには消えず、名前の後ろに `-deletion_scheduled-<番号>` が付いた「削除予定」になります。猶予期間が過ぎると消えます。名前が変わるので、同じ名前のプロジェクトを作り直しても重なりません。

## Section 6: 作業ファイルを消す

演習9で暗号化した人は作業用サーバへ、応用演習3でイメージを作った人は AAP サーバへ、それぞれ SSH します。`<name_prefix>` は自分の接頭辞に置き換えてから実行します。

```bash
cd /tmp
rm -f '/tmp/<name_prefix>-secrets.yml'
rm -rf '/tmp/<name_prefix>-custom-ee'
```

応用演習3でイメージを作った人は、続けて、ビルドしたときと同じ保存先を指定してからイメージを消します。`<storage.conf>` は、応用演習3の Section 3 で `systemctl --user show podman.service -p Environment` に出たパスです。出ていなかった人は、`export` の行は実行しません。

```bash
export CONTAINERS_STORAGE_CONF=<storage.conf>
podman rmi 'localhost/<name_prefix>-custom-ee:1.0'
```

イメージが無い（`image not known`）、という表示なら、そのコマンドはそこまでです。AAP は、Section 3 で実行環境 `<name_prefix>-custom-ee` を削除したときに、そのイメージも片付けます。`Default execution environment` のイメージは消しません。
