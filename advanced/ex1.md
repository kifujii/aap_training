# 応用演習 1 - RBAC

[演習11](../basic-controller/ex11.md) | [応用演習2へ](./ex2.md)

実行だけできるチームと、設定を変えられる人を分けます。

## Section 1: 画面上のロール名

一覧に Read や Execute という短い名前はありません。付けるオブジェクトの名前が前に付きます。

| 画面上の名前 | 付ける場所 | できること |
|---|---|---|
| JobTemplate Execute | `<name_prefix>-write-note-linux`、`<name_prefix>-write-note-windows` | 起動できる。編集はできない |
| WorkflowJobTemplate Execute | `<name_prefix>-note-workflow` | ワークフローを起動できる |
| Project Read Compat | `<name_prefix>-aap-training-content` | プロジェクトを見られる |
| Inventory Read Compat | `<name_prefix>-training-inventory` | インベントリーを見られる |

## Section 2: チームを作る

**Access Management > チーム** でチームを作ります。

| 項目 | 入力値 |
|---|---|
| 名前 | `<name_prefix>-ops-team` |
| 組織 | 演習1の組織 |

**ユーザー** タブで、実行確認に使うユーザーを追加します。別のユーザーが無い場合は、自分を追加して、あとで権限の差を画面で確認します。

## Section 3: 実行権限だけを付ける

`<name_prefix>-write-note-linux` の **Team Access** を開きます。

![Team Access。Assign teams からチームとロールを選ぶ](../images/aap/team-access.png)

**Assign teams** をクリックし、`<name_prefix>-ops-team` にチェックを入れて **Next** を押します。

![チームを選ぶ。チェックを入れて Next](../images/aap/team-access-assign.png)

**JobTemplate Execute** にチェックを入れ、**Next**、確認画面で **Finish** を押します。Admin にはチェックを入れません。

![ジョブテンプレートのロール。付けるのは JobTemplate Execute](../images/aap/team-access-roles.png)

`<name_prefix>-write-note-windows` にも **JobTemplate Execute** を付けます。`<name_prefix>-note-workflow` の Team Access では、名前が **WorkflowJobTemplate Execute** です。

プロジェクト `<name_prefix>-aap-training-content` の Team Access では **Project Read Compat** を付けます。インベントリー `<name_prefix>-training-inventory` では **Inventory Read Compat** です。

![プロジェクトのロール。付けるのは Project Read Compat](../images/aap/team-access-project-roles.png)

JobTemplate Execute だけでは、Playbook や認証情報の変更はできません。

## Section 4: 確認する

`<name_prefix>-ops-team` のユーザーでログインし、次を確認します。

* `<name_prefix>-write-note-linux` の **テンプレートの起動** はできる
* テンプレートの編集はできない
* プロジェクトの同期ボタンは押せない

自分の管理者ユーザーに戻り、編集できることも確認します。

## 次

[応用演習2](./ex2.md) で、失敗を通知します。
