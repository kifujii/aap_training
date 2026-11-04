# 演習 7 - Job Template詳細設定とSurvey

[前に戻る](./ex6.md)

---

この演習では、Job Templateの詳細設定（Extra Variables、Limit、Verbosity、Prompt on Launch）と、起動時に対話的に変数を入力させる **Survey** 機能を扱います。

## Section 1: Extra Variablesによる変数の上書き

[sample-content/roles/nginx/defaults/main.yml](../sample-content/roles/nginx/defaults/main.yml) で定義されている `nginx_site_version`（デフォルト値 `v1.0`）を、Job Template側から上書きしてみましょう。

### Step 1: Job Templateの編集画面を開く

`deploy-web-stack` の編集画面を開きます。

### Step 2: Variablesに入力する

「Variables」欄に、YAML形式で以下のように入力します。

```yaml
nginx_site_version: "v1.1"
```

「Save」をクリックします。

### Step 3: 実行して確認する

**テンプレートの起動** で実行し、PLAY RECAPで `web1`〜`web3` の `copy index.html` タスクが `changed` になることを確認してください（`nginx_site_version` の値が変わったため）。

**Note:** これはOSS版Ansibleの `ansible-playbook site.yml -e "nginx_site_version=v1.1"` に相当する操作です。Job Template画面の「Variables」欄は、Extra Variables（`-e`）として渡されます。[用語集 - 変数の優先順位の考え方](../aap_glossary.md)も参照してください。

## Section 2: Limitによる対象ホストの絞り込み

### Step 1: Limitを指定して実行する

**テンプレートの編集** の **制限** に `web1` と入力して実行してみましょう。起動プロンプトを有効にしている場合は、起動画面で入力します。

### Step 2: 結果を確認する

PLAY RECAPに `web1` のみが表示され、`web2`・`web3`・`lb` は実行対象外になっていることを確認してください。OSS版Ansibleの `--limit` オプションに相当します。

## Section 3: Verbosityの変更

**テンプレートの編集** を開き、**詳細** を `3 (Debug)` 程度に上げて実行してみましょう。SSH接続の詳細や、各タスクの内部的なモジュール呼び出し情報など、より詳細なログが出力されることを確認してください。確認後は `0 (正常)` に戻しておきます。

## Section 4: 起動プロンプト

毎回同じ Extra variables を固定するのではなく、実行時に都度値を選べるようにしたい場合は **起動プロンプト** を使います。英語UIの Prompt on launch です。

### Step 1: 起動プロンプトを有効化する

**テンプレートの編集** で、以下の項目の **起動プロンプト** にチェックを入れます。

* Extra variables
* 制限

保存すると、それらの固定値は、起動時の入力を促すプレースホルダーとして扱われるようになります。

### Step 2: 起動時の挙動を確認する

**テンプレートの起動** をクリックすると、実行前に入力画面が表示され、Extra variables と制限をその場で指定できることを確認してください。空欄のまま起動すれば、デフォルト値（role側のdefaults）が使われます。

**Note:** 起動プロンプトは、「どの項目を固定値にするか」「どの項目を実行者に委ねるか」を設計するための仕組みです。権限が低いユーザーにJob Templateの実行だけを許可しつつ、設定の詳細は起動時に入力させる、といった運用に使えます。

## Section 5: Surveyの作成

Surveyを使うと、Extra Variablesを直接YAMLで書かせるのではなく、フォーム形式の質問で入力させることができます。

### Step 1: Surveyを有効化する

Job Templateの **Survey** タブを開きます。質問がまだないときは **Create survey question** が表示されます。

### Step 2: 質問を追加する

**Create survey question** をクリックし、以下のように入力します。

| 項目 | 入力値 |
|---|---|
| Question | `Webサイトのバージョンを入力してください` |
| Answer variable name | `nginx_site_version` |
| Answer type | `Text` |
| Default answer | `v1.0` |
| Required | 有効 |

「Save」をクリックして質問を保存し、Surveyタブ自体も保存します。

### Step 3: Surveyを使って実行する

`deploy-web-stack` の **テンプレートの起動** をクリックすると、今度はSurveyの入力フォームが表示されます。`v1.2` など任意の値を入力して実行してみましょう。

### Step 4: 結果を確認する

PLAY RECAPで `web1`〜`web3` の `copy index.html` タスクが `changed` になることを確認してください。Surveyで入力した値が `nginx_site_version` としてExtra Variablesに渡され、[sample-content/roles/nginx/templates/index.html.j2](../sample-content/roles/nginx/templates/index.html.j2) に反映されています。

**Note:** SurveyはJob Template単位で設定するため、「Prompt on Launch + Variables（YAML直接入力）」と比べて、実行者がYAMLの書式を知らなくても安全に値を入力できるという利点があります。[宿題 課題3](../homework/hm1.md)以降でも活用します。

---

ここまでで、AAPの基本操作（basic-intro・basic-controller）が完了しました。次は目的に応じて、応用演習（RBAC、Vault、Workflow、Execution Environment、API/CLI、EDA、GitLab連携応用）に進んでください。

[応用演習へ進む](../advanced/ex1.md)
