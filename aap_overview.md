# AAPのしくみ

[READMEへ](./README.md)

演習に入る前に、画面のメニューと、1回のジョブが何の組み合わせかを読みます。用語を引くときは [AAP用語集](./aap_glossary.md)、ボタン名は [画面ラベル対応表](./ui-labels.md) です。操作の手順は各演習にあります。

## Ansible を画面から実行する

手元では `ansible-playbook` で Playbook を実行します。AAP（Ansible Automation Platform）は、その実行を Web の画面に置いたものです。Playbook と `inventory.yml` は GitLab に置きます。認証情報と、実行したログは AAP に残ります。

この資料の題材は、Linux と Windows の決まったディレクトリへファイルを書くことです。

## 3つのメニュー

ログイン後の左メニューは、次の3つです。

| メニュー | 役割 | この資料で使うとき |
|---|---|---|
| **Automation Execution** | Playbook を実行する。ジョブ、テンプレート、プロジェクト、インベントリー、認証情報、実行環境 | [演習2](./basic-intro/ex2.md) から |
| **Automation Decisions** | 届いたイベントの内容を見て、ジョブを起動する | [応用演習7](./advanced/ex7.md) から |
| **Access Management** | ログインするユーザー、組織、チーム | [演習1](./basic-intro/ex1.md) で組織名。[応用演習1](./advanced/ex1.md) で権限 |

作るオブジェクトは、演習1で控える組織のどれか1つに属します。

## ジョブテンプレートとジョブ

ジョブテンプレートは、次の4つを1つにまとめた実行の定義です。**テンプレートの起動** を押すたびに、ジョブが1件できます。ジョブには出力と、成功か失敗が残ります。

| 部品 | 画面の名前 | この資料での中身 |
|---|---|---|
| どの Playbook か | プロジェクト、Playbook | GitLab の `<name_prefix>-aap-training-content` |
| どのホストか | インベントリー | `<name_prefix>-training-inventory` の `lin1` と `win1` |
| どうログインするか | 認証情報 | 種類ごとに欄が違います。サーバへは `Machine`、GitLab を読むときは `Source Control`、暗号の解除は `Vault` |
| どの Ansible で動かすか | 実行環境 | 空欄なら、システムの既定の実行環境 |

```
プロジェクトの Playbook
  + インベントリー
  + 認証情報
  + 実行環境（空欄なら既定）
        ↓
  ジョブテンプレートを起動
        ↓
  ジョブ（1回の出力）
```

手元のコマンドとの対応は、次のとおりです。

| 手元 | AAP |
|---|---|
| `ansible-playbook write_note_linux.yml -i inventory` | ジョブテンプレートの Playbook とインベントリー |
| `-e name_prefix=<name_prefix>` | **Extra variables**。起動時に質問する形が Survey（[演習8](./basic-controller/ex8.md)） |
| `--limit lin1` | **制限** |
| `--check` | **ジョブタイプ** の **チェック** |

1つのジョブに付けられる Machine 認証情報は1つです。Linux と Windows は、別のジョブテンプレートにします。

## 実行環境

Playbook は、実行環境の中で動きます。実行環境は、Ansible 本体とコレクションが入ったコンテナイメージです。画面では **Automation Execution > Infrastructure > 実行環境** に一覧があります。

ジョブテンプレートの実行環境を空欄にすると、システムの既定の実行環境で動きます。使った名前は、ジョブ詳細に出ます。この資料でファイルを書くジョブは、実行環境を空欄のまま進めます。

この資料で空欄のまま使う既定の実行環境には、`ansible.controller` と `amazon.aws` が入っています。`community.general` は入っていません。入っていないコレクションを使うときは、イメージを追加して実行環境として登録します。その確認は [応用演習3](./advanced/ex3.md) です。

AWS でサーバを作るジョブも、この実行環境の中で動きます。作ったサーバへログインするのは、そのあとの演習です。

## あとの演習で足すもの

基本の4つに、次を足していきます。

| 機能 | 何をするか | 演習 |
|---|---|---|
| インベントリソース | GitLab の `inventory.yml` を、インベントリーへ取り込む | [演習6](./basic-controller/ex6.md) |
| Survey | 起動時に質問し、答えを Extra variables として渡す | [演習8](./basic-controller/ex8.md) |
| Vault | 秘密の値を暗号化したまま GitLab に置く。復号のパスワードは Vault 認証情報 | [演習9](./basic-controller/ex9.md) |
| スケジュール | 決めた時刻に、ジョブテンプレートを起動する | [演習10](./basic-controller/ex10.md) |
| ワークフロー | 複数のジョブテンプレートを線でつなぐ。途中に人の承認も置ける | [演習11](./basic-controller/ex11.md) |
| チームとロール | 起動できる人と、編集できる人を分ける | [応用演習1](./advanced/ex1.md) |
| 通知 | 失敗を、メールまたはログで知らせる | [応用演習2](./advanced/ex2.md) |
| API | 画面と同じ起動を、HTTP で行う。Playbook からも起動できる | [応用演習4](./advanced/ex4.md)、[応用演習5](./advanced/ex5.md) |
| Webhook と EDA | 外から届いたイベントでジョブを起動する | [応用演習6](./advanced/ex6.md) から [応用演習8](./advanced/ex8.md) |

EDA のルールブックは、**決定環境** という別のコンテナイメージの中で待ちます。ジョブを起動したあとの Playbook は、実行環境の中で動きます。

## 次

[事前準備](./prepare_training_environment.md) で値を控え、[演習1](./basic-intro/ex1.md) へ進みます。
