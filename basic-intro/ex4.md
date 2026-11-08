# 演習 4 - インベントリと疎通確認

[演習2](./ex2.md) | [演習5へ](../basic-controller/ex5.md)

AAP から Linux と Windows へ届くことを確認します。

| 自分の状況 | アドレスの出どころ |
|---|---|
| サーバがすでにある | 事前に控えたアドレス。[演習3](./ex3.md) は飛ばして、ここから再開 |
| 演習3で AWS に作った | ジョブログに出た「Linuxの接続先」「Windowsの接続先」 |

演習3で `aws-runner` を作った場合、それは AWS 用の実行場所です。この演習では、それとは別に `training-inventory` を作ります。`aws-runner` に Linux や Windows は追加しません。

## Section 1: インベントリーを作る

**Automation Execution > Infrastructure > インベントリー** を開き、**インベントリーの作成** > **インベントリーの作成** をクリックします。

![インベントリーの作成。名前と組織の位置](../images/aap/inventory-create.png)

| 項目 | 入力値 |
|---|---|
| 名前 | `training-inventory` |
| 組織 | 演習1で確認した組織 |

保存します。

## Section 2: グループを作る

`training-inventory` の **グループ** タブで、次の2つを作ります。

| グループ | 入れるホスト |
|---|---|
| `linux` | Linux |
| `windows` | Windows |

## Section 3: ホストを追加する

**ホスト** タブでホストを追加します。

![ホストの追加。名前、変数の位置](../images/aap/host-add.png)

Linux:

| 項目 | 入力値 |
|---|---|
| 名前 | `lin1` |
| 変数 | 下の YAML |

```yaml
ansible_host: 203.0.113.10
```

`203.0.113.10` は例です。自分の Linux のアドレスに置き換えてから保存し、グループ `linux` に入れます。

Windows（ドメインに参加していない場合。AWS で作ったサーバはこちら）:

| 項目 | 入力値 |
|---|---|
| 名前 | `win1` |
| 変数 | 下の YAML |

```yaml
ansible_host: 203.0.113.11
ansible_connection: psrp
ansible_psrp_auth: ntlm
ansible_psrp_protocol: http
ansible_psrp_port: 5985
ansible_psrp_cert_validation: ignore
```

アドレスを自分の値に置き換え、グループ `windows` に入れます。認証は NTLM です。接続プラグインは `psrp` にします。

### ドメインに参加している Windows の場合

上の変数の代わりに、次を使います。認証は Kerberos です。ユーザー名は演習2の `windows-psrp` に入れた `ユーザー@レルム` です。

```yaml
ansible_host: 203.0.113.11
ansible_connection: psrp
ansible_psrp_auth: kerberos
ansible_psrp_protocol: https
ansible_psrp_port: 5986
ansible_psrp_cert_validation: ignore
```

レルムと KDC は、[sample-content/krb5.conf.example](../sample-content/krb5.conf.example) を `krb5.conf` という名前で後の演習のプロジェクトに置きます。置き方は [演習5](../basic-controller/ex5.md) でファイルを追加するときで構いません。中の `EXAMPLE.COM` と `dc.example.com` は、自分の環境の値に書き換えます。

Kerberos の場合、アドホック実行では `krb5.conf` を渡せません。`win_ping` は飛ばし、[演習5](../basic-controller/ex5.md) で `krb5.conf` を置いたあと、[演習7](../basic-controller/ex7.md) の `write-note-windows` が成功すれば接続できています。NTLM の場合は、このまま Section 5 で `win_ping` を実行します。

## Section 4: 認証情報をホストに紐付ける必要はない

ジョブテンプレートに認証情報を付けるのは [演習7](../basic-controller/ex7.md) です。この演習の疎通確認では、実行時に認証情報を選びます。

## Section 5: 疎通確認

`training-inventory` の **ホスト** タブを開き、`lin1` にチェックを入れます。**Run command** をクリックします。

![Run command。モジュール名、認証情報、起動ボタンの位置](../images/aap/run-command.png)

| 項目 | Linux | Windows |
|---|---|---|
| モジュール | `ping` | `ansible.windows.win_ping` |
| 認証情報 | `linux-ssh` | `windows-psrp` |

起動し、`SUCCESS` と表示されることを確認します。Windows を NTLM で接続する場合は、`lin1` のチェックを外し、`win1` だけを選んでから実行します。1つの実行に付けられる Machine 認証情報は1つなので、Linux と Windows は別々に確認します。Kerberos の場合は、ここでの `win_ping` は飛ばします。

`ping` が `SUCCESS` なら、SSH のユーザー名とパスワード、アドレスが合っています。`win_ping` が `SUCCESS` なら、PSRP の認証とポートが合っています。

失敗したときの確認箇所:

* `ansible_host` が、控えたアドレスと一致しているか
* 認証情報のユーザー名とパスワード
* Windows の `ansible_psrp_auth` が、参加状況（NTLM か Kerberos か）と一致しているか
* セキュリティグループやファイアウォールが、AAP から TCP 22（Linux）または 5985（NTLM）または 5986（Kerberos）を許可しているか

## 次

[演習5](../basic-controller/ex5.md) で、Playbook を GitLab に置きます。
