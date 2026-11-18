# 演習 4 - AWSでサーバを作る

[演習3](./ex3.md) | [ホスト登録へ戻る](./ex3.md#add-hosts)

> **前提:** [演習2](./ex2.md) の AWS 用と GitLab 用の認証情報、[演習3](./ex3.md) Section 3 の `<name_prefix>-aws-runner`

> [!IMPORTANT]
> **対象の Linux と Windows がすでにある（既存サーバを使用する）場合は、この演習を全部飛ばして [演習5](../basic-controller/ex5.md) へ進みます。**

この演習では、AWS に次のリソースを作ります。演習1で決めた接頭辞を `resource_prefix` として渡し、各リソースの名前の先頭に付けるので、同じアカウントを複数人で使ってもリソースが分かれます。

| リソース | 名前 |
|---|---|
| VPC | `<name_prefix>-vpc` |
| サブネット | `<name_prefix>-subnet` |
| インターネットゲートウェイ | `<name_prefix>-igw` |
| ルートテーブル | `<name_prefix>-rt` |
| セキュリティグループ | `<name_prefix>-aap`（TCP 22、TCP 5985、応用演習2で使う TCP 1025） |
| Linux 1台 | `<name_prefix>-linux`（SSH、ユーザー `ansible`） |
| Windows 1台 | `<name_prefix>-windows`（PSRP、NTLM、ユーザー `Administrator`） |

作ったサーバのアドレスはジョブのログに出るので、控えておきます。

ジョブは数分で終わります。Windows の起動が遅いときは、PSRP の待ち受けが開くまで待つため、15分ほどかかることがあります。

サーバのパスワードは自分で決め、このあと Extra variables に入れます。テンプレートの **詳細** を初期値の `0 (正常)` のまま起動すれば、パスワードそのものはタスクの出力に出ません。ただし、ジョブの詳細とジョブテンプレートの Extra variables には、入れた値が平文で残ります。そのため、この演習のサーバ専用の値にして、本番で使っているパスワードは入れないでください。ジョブテンプレートに残った値は、Section 3 の最後で消します。実際の運用で起動のたびに秘密の値を渡すときは、Survey の Answer type を Password にすると、入れた値がジョブの詳細に平文で残りません。Survey の作り方は [演習8](../basic-controller/ex8.md) で扱います。

開始前に、[演習3](./ex3.md) の Section 3 まで終わっていることを確認します。ジョブテンプレートのインベントリー欄では、そこで作った `<name_prefix>-aws-runner` を選びます。この演習でインベントリーは作りません。

## Section 1: PlaybookをGitLabに置く

ここでは、サーバ作成に必要なファイルだけ先に置きます。残りのファイルは、[演習5](../basic-controller/ex5.md) で同じプロジェクトに足します。

1. GitLab で **Create new…** > **New project/repository** > **Create blank project** を開く
2. Project name は `<name_prefix>-aap-training-content`、Visibility は `Private`。README の初期化は付けたままにする

![空のプロジェクトを作る画面](../images/gitlab/blank-project.png)

3. 右上の **Code** を開き、**Web IDE** をクリックする。メニュー名が **Edit** の環境では、その中の **Web IDE** を選ぶ

![Web IDE。左の一覧でファイルを開き、中央で中身を編集する。コミットは左端の Source Control から行う](../images/gitlab/web-ide.png)

4. 次の4ファイルを作り、[sample-content/](../sample-content/) の内容を貼り付ける
   * `create_aws_servers.yml`
   * `remove_aws_servers.yml`
   * `requirements.yml`
   * `group_vars/all.yml`
5. **Source Control** から **Commit to main** をクリックする
6. プロジェクトの画面に戻り、右上の **Code** を開いて、**Clone with HTTPS** の URL を控える。Section 2 で使う

`requirements.yml` は、ジョブが使う `amazon.aws` の版を 10.3.0 に固定するファイルです。既定の実行環境にも 10.3.0 が入っているので、このファイルが無くてもジョブは動きますが、置いておけば、既定の実行環境が更新されても同じ版で動きます。AWS で作る場合だけ置きます。

`group_vars/all.yml` の `allow_cidr` は `0.0.0.0/0` で、作ったサーバの SSH（TCP 22）、PSRP（TCP 5985）、応用演習2の検証用メールサーバ（TCP 1025）への接続を、どの接続元からも許可する設定です。この演習では、この値のまま使います。AAP がサーバへ接続するときの送信元アドレスが分かっている場合は、`0.0.0.0/0` をそのアドレスに `/32` を付けた値（例: `198.51.100.20/32`）に変えてからコミットします。実際の運用では、接続元のアドレスだけを指定します。学習が終わったら、Section 5 でサーバを消します。

<details>
<summary>git CLIで行う場合</summary>

```bash
git clone https://gitlab.example.local/<namespace>/<name_prefix>-aap-training-content.git
cd <name_prefix>-aap-training-content
# 上の4ファイルだけをコピーする
git add create_aws_servers.yml remove_aws_servers.yml requirements.yml group_vars/all.yml
git commit -m "Add AWS server creation playbooks"
git push
```

</details>

## Section 2: AAPにプロジェクトを登録する

**Automation Execution > プロジェクト** を開き、**プロジェクトの作成** をクリックします。

![Project 作成画面。ソースコントロールの URL と認証情報の位置](../images/aap/project-create-git.png)

| 項目 | 入力値 |
|---|---|
| 名前 | `<name_prefix>-aap-training-content` |
| ソースコントロールタイプ | Git |
| ソースコントロールURL | Section 1 で控えた **Clone with HTTPS** の URL |
| ソースコントロールブランチ | `main` |
| ソースコントロール認証情報 | `<name_prefix>-gitlab-scm` |
| 起動時のリビジョン更新 | 有効 |
| クリーン | 有効 |

保存したあと、同期を実行し、成功することを確認します。失敗したら URL、ユーザー名、パスワード、ブランチ名を見直してください。

## Section 3: ジョブテンプレートを作って実行する

**Automation Execution > テンプレート** を開き、**テンプレートの作成** > **ジョブテンプレートの作成** をクリックします。

![ジョブテンプレートの作成画面](../images/aap/job-template-create.png)

| 項目 | 入力値 |
|---|---|
| 名前 | `<name_prefix>-create-aws-servers` |
| インベントリー | `<name_prefix>-aws-runner` |
| プロジェクト | `<name_prefix>-aap-training-content` |
| Playbook | `create_aws_servers.yml` |
| 実行環境 | 空欄（既定のまま） |
| 認証情報 | `<name_prefix>-aws-training` だけ。Machine は付けない |
| Extra variables | 下の YAML |

```yaml
resource_prefix: sample01
linux_login_password: <Linuxのパスワード>
windows_login_password: <Windowsのパスワード>
```

`sample01` は例です。演習1で決めた接頭辞と同じ文字列にします。`<Linuxのパスワード>` と `<Windowsのパスワード>` は、自分で決めた値に置き換えます。`<` と `>` は残しません。パスワードの条件は次のとおりです。条件に合わないと、最初のタスク `Required variables are set` で止まります。

* 英数字だけで、8文字以上32文字以内
* 英大文字、英小文字、数字を、それぞれ1文字以上含む（Windows は、この3種類が無いパスワードを設定できません）

実行環境は空欄のままにして、Playbook をシステムの既定の実行環境の中で動かします。実行環境については [AAPのしくみ](../aap_overview.md) で説明しています。

リージョンは `group_vars/all.yml` に書かれた `ap-northeast-1` のまま使うので、Extra variables に入れるのは上の3行だけです。

**テンプレートの起動** をクリックします。成功すると、最後のタスクに次のような行が出ます。

```
resource_prefix: sample01
Linuxの名前: sample01-linux
Linuxの接続先: 203.0.113.10
Linuxのユーザー: ansible
Windowsの名前: sample01-windows
Windowsの接続先: 203.0.113.11
Windowsのユーザー: Administrator
パスワードは、このジョブの Extra variables に入れた値です。
```

`sample01`、`203.0.113.10`、`203.0.113.11` は例です。ログに出た2つのアドレスとユーザー名を控えます。パスワードは Extra variables に入れた2つの値で、ジョブの詳細の Extra variables でも確認できます。

値を控えたら、テンプレートを開ける他の人にパスワードが見えないように、`<name_prefix>-create-aws-servers` の **テンプレートの編集** を開き、Extra variables から `linux_login_password` と `windows_login_password` の2行を消して保存します。`resource_prefix` の行は残します。もう一度このジョブを起動するときは、2行を入れ直します。

失敗したときは、失敗したタスクの `msg` を見ます。よくある原因は、`resource_prefix` の文字種、パスワードの条件（記号が入っている、英大文字・英小文字・数字のどれかが無い）、AWS のキーに EC2 の作成権限が無いこと、VPC の数がアカウントの上限（初期値はリージョンあたり5つ）に達していることです。VPC の上限に達したときは、AWS のコンソールで使っていない VPC を消すか、`group_vars/all.yml` の `ec2_region` を別のリージョンに変えてから、もう一度起動します。どちらもできないときは、既存サーバを用意し、[README](../README.md) の「演習の順番」の表にある、既存サーバを使用する場合の順で進めます。

## Section 4: ログイン用の認証情報を作る

[演習2](./ex2.md) と同じ作成画面で、Machine 認証情報を2つ作ります。

| 名前 | ユーザ名 | パスワード |
|---|---|---|
| `<name_prefix>-linux-ssh` | `ansible` | `linux_login_password` に入れた値 |
| `<name_prefix>-windows-psrp` | `Administrator` | `windows_login_password` に入れた値 |

**Privilege Escalation Method** は空欄のままにします。

## Section 5: 不要になったら削除する

AWS のリソースは、残しておくと課金が続きます。学習が終わったら、同じ要領でジョブテンプレート `<name_prefix>-remove-aws-servers` を作り、Playbook を `remove_aws_servers.yml` にして実行します。Extra variables の `resource_prefix` は作成時と同じ値にします。パスワードは要りません。認証情報は `<name_prefix>-aws-training`、インベントリーは `<name_prefix>-aws-runner` を選びます。

## 次

[演習3の Section 4](./ex3.md#add-hosts) に戻り、控えたアドレスを `lin1` と `win1` として登録し、疎通まで確認します。そのあと [演習5](../basic-controller/ex5.md) へ進みます。
