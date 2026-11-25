# 演習 6 - インベントリソース

[演習5](./ex5.md) | [演習7へ](./ex7.md)

> **前提:** [演習3](../basic-intro/ex3.md) のインベントリー、[演習5](./ex5.md) のプロジェクト

演習3では画面からホストを登録しました。この演習では、GitLab の `inventory.yml` をインベントリーのソースにして、同期するたびに AAP のホスト一覧がこのファイルの内容に合うようにします。

## Section 1: ソースを作る

`<name_prefix>-training-inventory` を開き、**ソース** タブでソースを追加します。

| 項目 | 入力値 |
|---|---|
| 名前 | `<name_prefix>-gitlab-inventory` |
| ソース | `Sourced from a Project` |
| プロジェクト | `<name_prefix>-aap-training-content` |
| Inventory path | `inventory.yml` |
| Overwrite | 有効 |
| Overwrite variables | 有効 |
| Update on launch | 有効 |

日本語UIでも、ファイルを指定する欄の名前は **Inventory path** で、「インベントリファイル」という欄はありません。

![インベントリソースの編集。ソースは Sourced from a Project、Inventory path は inventory.yml。Overwrite、Overwrite variables、Update on launch にチェックが入っている](../images/aap/inventory-source.png)

写真のプロジェクト名には接頭辞が写っていませんが、選ぶのは `<name_prefix>-aap-training-content` です。認証情報の欄は空欄のままにし、**Save source** を押して保存します。

Overwrite を有効にすると、このソースが前回の同期で取り込んだホストやグループのうち、`inventory.yml` から削除したものが消えるようになります。画面から追加しただけで、ファイルに一度も書いていないホストは残ります。演習3で画面から入れた `lin1` と `win1` は、ファイルに同じ名前で書いてあれば残り、変数はファイルの内容で上書きされます。

## Section 2: 同期する

ソースの同期を実行します。成功すると、グループ `linux` と `windows`、ホスト `lin1` と `win1` が表示されます。`inventory.yml` で2つのグループをまとめている親グループ `targets` も表示されます。`lin1` と `win1` を開き、次の値がファイルに書いた値と同じになっていることを確認します。

* `ansible_host` が、自分のアドレスになっていること。Kerberos の Windows は、演習3で保存した FQDN になっていること
* Windows の `ansible_psrp_auth`、`ansible_psrp_protocol`、`ansible_psrp_port` が、演習3で保存した3つと一致していること。NTLM のままなら `ntlm`、`http`、`5985`、Kerberos に置き換えた人は `kerberos`、`https`、`5986`、Kerberos で HTTP に切り替えた人は `kerberos`、`http`、`5985`

まだ `203.0.113.10` のまま、または Windows の3つのどれかが演習3と違うときは、Web IDE で `inventory.yml` を直し、コミットしてから、もう一度同期します。

AWS でサーバを作った場合は、演習3で作った `<name_prefix>-aws-runner` には、このソースを付けません。付けると、同期で `localhost` が消えます。

## 次

[演習7](./ex7.md) で、`note.txt` に書き込むジョブテンプレートを作って実行します。
