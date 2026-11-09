# 演習 8 - 起動時の値、制限、Survey

[演習7](./ex7.md) | [演習9へ](./ex9.md)

`training_note` は `group_vars/all.yml` では `managed by AAP` です。ジョブの起動時に別の文字列へ変え、対象ホストを1台に絞ります。

## Section 1: Extra variables

`write-note-linux` の **テンプレートの編集** を開き、下までスクロールします。入れる欄の名前は **Extra variables** です。「変数」という欄はありません。

![Extra variables の位置。横の起動プロンプトは外したまま](../images/aap/extra-variables.png)

この写真の `training_use_vault: true` は、[演習9](./ex9.md) で入れる行です。この演習では、その行は入れず、欄の中を次の1行だけにします。

```yaml
training_note: hello from survey
```

**ジョブテンプレートの保存** を押し、起動します。出力の `Write note.txt` が `changed` になり、ファイルの `note:` の行が `hello from survey` になります。

これは `ansible-playbook write_note_linux.yml -e "training_note=hello from survey"` と同じです。

確認が終わったら、Extra variables の中を空にして保存します。Survey と両方に値があると、どちらが効くか分かりにくくなります。

## Section 2: 制限

もう一度 **テンプレートの編集** を開きます。**制限** の右にある **起動プロンプト** にチェックを入れます。ジョブタイプの横の起動プロンプトは外したままです。

![制限の右の起動プロンプトにチェックが入っている。ジョブタイプの横は外したまま](../images/aap/job-type.png)

保存して起動すると、実行前に制限の入力欄が出ます。`lin1` と入力して起動し、出力の対象が `lin1` だけであることを確認します。写真の認証情報に `vault-training` が見えるのは演習9のあとです。この演習では `linux-ssh` だけです。

## Section 3: Survey

`write-note-linux` の **Survey** タブを開き、**Create survey question** をクリックします。

![Survey の質問。Question、Answer variable name、Default answer](../images/aap/survey-question.png)

| 項目 | 入力値 |
|---|---|
| Question | `ファイルに書くメモを入力してください` |
| Answer variable name | `training_note` |
| Answer type | Text |
| Required | 有効 |
| Default answer | `managed by AAP` |

**Save survey question** を押します。一覧に戻ったら、左の **Survey enabled** をオンにします。オフのままだと、起動時に質問が出ません。

![Survey enabled がオンで、質問が1件ある状態](../images/aap/survey-enabled.png)

起動すると入力欄が出ます。`second note` と入力して実行し、`note.txt` の `note:` が `second note` になることを確認します。終わったら、もう一度起動して既定の `managed by AAP` に戻します。

`write-note-windows` にも、同じ質問を1つ作り、同じ確認をします。

## 次

[演習9](./ex9.md) で、パスワードを暗号化したファイルとして渡します。
