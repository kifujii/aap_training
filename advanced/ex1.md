# 応用演習 1 - RBAC

[演習11](../basic-controller/ex11.md) | [応用演習2へ](./ex2.md)

実行だけできるチームと、設定を変えられる人を分けます。

## Section 1: ロール

| ロール | できること |
|---|---|
| Execute | Job Template の起動。編集はできない |
| Read | 閲覧 |
| Use | Job Template 作成時に、その認証情報やプロジェクトを選べる |
| Admin | そのオブジェクトの編集 |
| Approve | Workflow の承認 |

## Section 2: チームを作る

**Access Management > チーム** でチームを作ります。

| 項目 | 入力値 |
|---|---|
| 名前 | `ops-team` |
| 組織 | 演習1の組織 |

**ユーザー** タブで、実行確認に使うユーザーを追加します。別のユーザーが無い場合は、自分を追加して、あとで権限の差を画面で確認します。

## Section 3: 実行権限だけを付ける

`write-note-linux` の **Team Access** で、`ops-team` に **Execute** を付けます。`write-note-windows` と `note-workflow` にも、同じ Execute を付けます。

`aap-training-content` プロジェクトのアクセスには **Read** を付けます。`training-inventory` にも **Read** を付けます。

Execute だけでは、Playbook や認証情報の変更はできません。

## Section 4: 確認する

`ops-team` のユーザーでログインし、次を確認します。

* `write-note-linux` の **テンプレートの起動** はできる
* テンプレートの編集はできない
* プロジェクトの同期ボタンは押せない

自分の管理者ユーザーに戻り、編集できることも確認します。

## 次

[応用演習2](./ex2.md) で、失敗を通知します。
