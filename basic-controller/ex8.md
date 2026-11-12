# 演習 8 - 起動時の値、制限、Survey

[演習7](./ex7.md) | [演習9へ](./ex9.md)

`training_note` は `group_vars/all.yml` では `managed by AAP` です。この演習では、節ごとにジョブを起動します。Section 1 はメモの文面、Section 2 は対象ホスト、Section 3 は起動時の質問です。`name_prefix` の行は、どの節でも残します。

## Section 1: Extra variables

`<name_prefix>-write-note-linux` の **テンプレートの編集** を開き、下までスクロールします。入れる欄の名前は **Extra variables** です。「変数」という欄はありません。

![Extra variables の位置。横の起動プロンプトは外したまま](../images/aap/extra-variables.png)

この写真の `training_use_vault: true` は、[演習9](./ex9.md) で入れる行です。この演習では、その行は入れません。演習7で保存した `name_prefix` の行は残します。この行が無いと、ファイルを書く前に失敗します。

欄の中を、次の2行にします。`tanaka01` は、演習7で保存してある自分の接頭辞のままにします。

```yaml
name_prefix: tanaka01
training_note: hello from survey
```

**ジョブテンプレートの保存** を押し、起動します。出力の `Write note.txt` が `changed` になり、`Show the note written to note.txt` に `note.txt の note は hello from survey です` と出ます。

これは `ansible-playbook write_note_linux.yml -e "training_note=hello from survey"` と同じです。`name_prefix` は、ファイルの置き場所に使われます。

確認が終わったら、`training_note` の行だけを消して保存します。`name_prefix` の行は残します。Survey と Extra variables の両方に `training_note` があると、どちらがメモの文面になるか分かりにくくなります。

もう一度起動します。`Show the note written to note.txt` は、`group_vars/all.yml` の既定値どおり `note.txt の note は managed by AAP です` に戻ります。

## Section 2: 制限

もう一度 **テンプレートの編集** を開きます。Extra variables は、次の1行のままです。

```yaml
name_prefix: tanaka01
```

**制限** の右にある **起動プロンプト** にチェックを入れます。ジョブタイプの横の起動プロンプトは外したままです。

![制限の右の起動プロンプトにチェックが入っている。ジョブタイプの横は外したまま](../images/aap/job-limit-prompt.png)

保存して起動すると、**Prompt on Launch** が開きます。この時点の手順は **Prompts** と **Review** だけです。Survey は、次の節で作ります。**制限** に `lin1` と入力して **Next** を押し、**Review** で **Finish** を押します。

![Prompt on Launch の Review。制限は lin1。最後のボタンは Finish。手順は Prompts と Review だけ](../images/aap/launch-finish.png)

写真の `tanaka01` は、演習1の例です。自分の接頭辞に読み替えます。出力の対象が `lin1` だけであることを確認します。`Show the note written to note.txt` は `note.txt の note は managed by AAP です` のままです。この演習の認証情報は `<name_prefix>-linux-ssh` だけです。

## Section 3: Survey

`<name_prefix>-write-note-linux` の **Survey** タブを開き、**Create survey question** をクリックします。Extra variables の `name_prefix` は、このまま残します。

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

起動すると **Prompt on Launch** が開きます。手順は **Prompts**、**Survey**、**Review** です。**Prompts** の **制限** は空欄のまま **Next** を押します。空欄のときは、Playbook の `hosts: linux` が対象です。

![Prompt on Launch の Prompts。制限は空欄](../images/aap/launch-limit.png)

**Survey** の「ファイルに書くメモを入力してください」に `second note` と入力して **Next** を押します。写真は既定の `managed by AAP` のままです。この1回目は、その文字を消して `second note` にします。**Review** の Extra vars に `training_note` が出るのは、この Survey の答えです。テンプレートの Extra variables に書き戻したわけではありません。**Finish** を押すとジョブが始まります。`Show the note written to note.txt` に `note.txt の note は second note です` と出ることを確認します。

![Prompt on Launch の Survey。既定の答えは managed by AAP](../images/aap/launch-survey.png)

終わったら、もう一度起動します。**制限** は空欄、**Survey** は既定の `managed by AAP` のまま **Finish** まで進みます。`Show the note written to note.txt` が `note.txt の note は managed by AAP です` に戻ることを確認します。

この起動画面は、`<name_prefix>-write-note-linux` を起動するたびに出ます。以降の演習でも、制限と Survey は、その演習に書いてある値にします。

`<name_prefix>-write-note-windows` の Extra variables にも、演習7で入れた `name_prefix` の行が残っています。同じ質問を1つ作り、同じ2回の起動をします。Windows のテンプレートには制限の起動プロンプトを付けていないので、**Prompts** は出ません。最初の手順は **Survey** です。1回目の `Show the note written to note.txt` は `note.txt の note は second note です`、2回目は `note.txt の note は managed by AAP です` です。

![write-note-windows の起動。Survey から始まり、制限の欄は無い](../images/aap/launch-survey-windows.png)

## 次

[演習9](./ex9.md) で、ファイルに書く秘密の値だけを暗号化して渡します。
