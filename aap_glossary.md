# AAP用語集

[READMEへ](./README.md)

このページは、用語を引くための一覧です。AAP の全体像は、先に [AAPのしくみ](./aap_overview.md) で説明しています。「画面での名前」の列は AAP 2.7 の日本語 UI の表示に合わせており、画面で英語のまま表示される名前は英語で書いています。

---

## 1. 基本

| 用語 | 画面での名前 | 説明 |
|---|---|---|
| **Ansible Automation Platform (AAP)** | — | Red Hat が提供する自動化の基盤。Ansible の実行に、Web の画面、権限の管理、API、スケジュール、イベント駆動の自動化（EDA）などを加えたもの。 |
| **Automation Controller** | **Automation Execution** | Playbook の実行を管理する部分。ジョブテンプレート、ワークフロー、インベントリー、認証情報などを扱う。旧称 Ansible Tower。 |
| **Event-Driven Ansible (EDA)** | **Automation Decisions** | 届いたイベント（監視のアラート、Webhook など）の内容を見て、ジョブテンプレートの起動などを行う仕組み。[応用演習7](./advanced/ex7.md) から使う。 |
| **Platform Gateway** | **Access Management** ほか | AAP 2.5 以降で、ログイン、ユーザー、チーム、組織をまとめて管理する部分。API の入り口も兼ね、controller の API は `/api/controller/v2/` になる。 |

---

## 2. Automation Execution のオブジェクト

| 用語 | 画面での名前 | 説明 |
|---|---|---|
| **Organization** | **組織** | AAP の中の管理の単位。作るオブジェクトは、いずれかの組織に属する。この資料では [演習1](./basic-intro/ex1.md) で控えた組織を選ぶ。 |
| **User** | **ユーザー** | AAP にログインするアカウント。ユーザータイプ（システム管理者、標準ユーザーなど）と、オブジェクトごとのロールを持つ。 |
| **Team** | **チーム** | 複数のユーザーをまとめたもの。チームにロールを付けると、所属するユーザー全員に同じ権限が付く（[応用演習1](./advanced/ex1.md)）。 |
| **Credential** | **認証情報** | サーバへのログイン、GitLab の読み取り、Vault の復号、クラウドの API キーなど、ジョブに必要な秘密の値を保存するオブジェクト。種類（認証情報タイプ）ごとに入力欄が違う。 |
| **Credential Type** | **認証情報タイプ** | 認証情報の種類。この資料では Machine、Source Control、Vault、Amazon Web Services、Red Hat Ansible Automation Platform などを使う。 |
| **Project** | **プロジェクト** | Playbook 一式の置き場所。この資料では GitLab のリポジトリと結び付け、同期で最新の内容を取ってくる。 |
| **Project Sync** | 同期 | プロジェクトに結び付けたリポジトリの内容を、AAP に取ってくる操作。プロジェクトの **起動時のリビジョン更新** が有効なら、ジョブの起動時にも同期する。GitLab の Webhook そのものは同期しない（[応用演習6](./advanced/ex6.md)）。 |
| **Revision** | **リビジョン** | 同期で取ってきたリポジトリのコミット。ジョブの **詳細** で、どのコミットの Playbook を使ったかが分かる。 |
| **Inventory** | **インベントリー** | ジョブの対象のホストとグループの一覧。画面で登録する方法と、インベントリソースで取り込む方法がある。 |
| **Inventory Source** | **ソース**（インベントリーのタブ） | インベントリーにホストを取り込む方法の設定。この資料では、GitLab の `inventory.yml` を読む **Sourced from a Project** を使う（[演習6](./basic-controller/ex6.md)）。 |
| **Job Template** | **テンプレート**（ジョブテンプレート） | どのプロジェクトの Playbook を、どのインベントリーに、どの認証情報で実行するかをまとめた定義。起動するとジョブが1件できる。 |
| **Job** | **ジョブ** | ジョブテンプレートを1回起動した記録。出力、ステータス、使った Extra variables などが残る。 |
| **Workflow Job Template** | **ワークフロージョブテンプレート** | 複数のジョブテンプレートや承認をノードとしてつなぎ、成功や失敗に応じて次へ進める定義（[演習11](./basic-controller/ex11.md)）。 |
| **Workflow Approval** | **Workflow Approvals** | ワークフローの途中に置く、人による承認。承認されるまで、後ろのノードは動かない。 |
| **Ad Hoc Command** | **Run command** | Playbook を書かずに、インベントリーのホストにモジュールを1回だけ実行する機能。`ansible <ホスト> -m <モジュール>` に当たる（[演習3](./basic-intro/ex3.md)）。 |

---

## 3. 実行の設定

| 用語 | 画面での名前 | 説明 |
|---|---|---|
| **Extra Variables** | **Extra variables** | ジョブに渡す追加の変数。`ansible-playbook` の `-e` に当たり、Ansible の変数の中で最も優先される。優先順位は [演習8](./basic-controller/ex8.md) の表にある。 |
| **Limit** | **制限** | 実行の対象を、インベントリーの中の特定のホストやグループに絞る指定。`--limit` に当たる。 |
| **Verbosity** | **詳細** | 出力の詳しさ。`0 (正常)` から選び、`1 (詳細)` は `-v` に当たる。 |
| **Job Type** | **ジョブタイプ** | ジョブの種類。**実行**（変更する）と **チェック**（`--check` に当たる。変更しない）がある。 |
| **Prompt on Launch** | **起動プロンプト**（チェック）／ **Prompt on Launch**（起動時の画面） | 欄の横の **起動プロンプト** にチェックを入れると、その値を起動のたびに入力できる。入力する画面の名前が **Prompt on Launch**。 |
| **Survey** | **Survey** | 起動のたびに質問を出し、答えを変数としてジョブに渡す機能（[演習8](./basic-controller/ex8.md)）。 |
| **Schedule** | **スケジュール** | ジョブテンプレートやワークフローを、決めた時刻に起動する設定。繰り返しは iCal の RRULE 形式（例: `RRULE:FREQ=DAILY`）で保存される（[演習10](./basic-controller/ex10.md)）。 |
| **Notification Template** | **Notifiers** | ジョブの開始、成功、失敗などを、メールなどで知らせる設定。ジョブテンプレートの **通知** タブで付ける（[応用演習2](./advanced/ex2.md)）。 |
| **Privilege Escalation** | **Privilege Escalation Method**（認証情報） | sudo などで root の権限に切り替えてタスクを実行すること。Playbook の `become: true` と、認証情報の方式を組み合わせて使う（[演習7](./basic-controller/ex7.md)）。 |
| **PLAY RECAP** | 出力の最後 | ジョブの出力の最後に出る、ホストごとの結果のまとめ（ok、changed、failed など）。 |

---

## 4. 実行環境

| 用語 | 画面での名前 | 説明 |
|---|---|---|
| **Execution Environment (EE)** | **実行環境** | ジョブの中で Ansible を動かすコンテナイメージ。Ansible 本体、Python のライブラリ、コレクションが入っている。以前の版で使っていた Python の仮想環境（venv）に代わるもの。 |
| **Default Execution Environment** | `Default execution environment` | 実行環境の欄を空欄にしたジョブが使う、既定の実行環境。この資料の既定には `ansible.controller` と `amazon.aws` が入っていて、`community.general` は入っていない。 |
| **ansible-builder** | — | `execution-environment.yml` の定義から、実行環境のイメージを作るコマンド（[応用演習3](./advanced/ex3.md)）。 |
| **execution-environment.yml** | — | 実行環境に入れるコレクション、Python のライブラリ、OS のパッケージなどを書く、ansible-builder の定義ファイル。 |
| **Container Registry Credential** | 認証情報タイプ `Container Registry` | 実行環境のイメージを置くコンテナレジストリへの認証情報。 |
| **Decision Environment (DE)** | **決定環境** | EDA のルールブックを動かすコンテナイメージ。ジョブの実行環境とは別のもの。 |

---

## 5. 権限と秘密の値

| 用語 | 画面での名前 | 説明 |
|---|---|---|
| **RBAC (Role-Based Access Control)** | **Team Access**、**User Access** | ユーザーやチームにロールを付けて、オブジェクトごとに操作できる範囲を決める仕組み（[応用演習1](./advanced/ex1.md)）。 |
| **Role** | `JobTemplate Execute` など | 付けられる権限のまとまり。画面では、`JobTemplate Execute`、`Project Read Compat` のように、オブジェクトの種類が前に付く。`Compat` が付くロールは、以前の版のロールに合わせて用意された組み込みのロール。 |
| **System Administrator** | ユーザータイプ **システム管理者** | AAP 全体を操作できるユーザー。ロールに関係なく、すべてを見て、変えられる。 |
| **ansible-vault** | — | 秘密の値を暗号化するコマンド。この資料では `ansible-vault encrypt_string` で値だけを暗号化し、ファイルは YAML のまま GitLab に置く（[演習9](./basic-controller/ex9.md)）。 |
| **Vault Credential** | 認証情報タイプ `Vault` | 暗号化した値を復号するパスワードを保存する認証情報。ジョブテンプレートに付けると、ジョブの中で値が復号される。 |
| **no_log** | — | タスクの結果をジョブの出力に出さない指定。秘密の値を扱うタスクに付ける（[演習9](./basic-controller/ex9.md)）。 |

---

## 6. API

| 用語 | 画面での名前 | 説明 |
|---|---|---|
| **Controller REST API** | — | 画面でできる操作の大部分を、HTTP で行える API。AAP 2.5 以降は Platform Gateway を通り、`https://<AAPのURL>/api/controller/v2/` になる（[応用演習4](./advanced/ex4.md)）。 |
| **API Token** | **API Tokens** | API を呼ぶときの認証に使うトークン。**Access Management > API Tokens** で発行し、`Authorization: Bearer <トークン>` で送る。発行時に1度だけ表示され、使い終わったら削除する。 |
| **ansible.controller** | — | Playbook から AAP の API を操作するコレクション（例: `ansible.controller.job_launch`）。既定の実行環境に入っている（[応用演習5](./advanced/ex5.md)）。 |
| **Webhook**（ジョブテンプレート） | **Webhook の有効化** | GitLab などからの HTTP リクエストを受けて、ジョブテンプレートを起動する仕組み（[応用演習6](./advanced/ex6.md)）。 |

---

## 7. EDA

| 用語 | 画面での名前 | 説明 |
|---|---|---|
| **Rulebook** | **ルールブック** | どのイベントを、どんな条件で、どの処理につなげるかを書いた YAML ファイル。 |
| **Source** | ルールブックの `sources` | ルールブックがイベントを受け取る入り口の定義。この資料では `ansible.eda.webhook` を Event Stream に置き換えて使う。 |
| **Rule / Condition / Action** | ルールブックの `rules` | イベントに対する条件（condition）と、条件に合ったときの処理（action）の組。処理には、ジョブテンプレートの起動（`run_job_template`）や、ログへの出力（`debug`）がある。 |
| **Event Stream** | **Event Streams** | 外からのイベントを HTTP で受け取る、EDA の受け口。URL と、送る側が付けるトークンを持つ（[応用演習7](./advanced/ex7.md)）。 |
| **Rulebook Activation** | **ルールブックのアクティベーション** | ルールブックを動かし続けて、イベントを待ち受ける設定。決定環境、ルールブック、認証情報、Event Stream を選ぶ。 |
| **Rule Audit** | **ルール監査** | ルールが条件に合って処理を行った記録。 |

---

## 8. GitLab

| 用語 | 画面での名前 | 説明 |
|---|---|---|
| **Web IDE** | **Web IDE**（GitLab） | ブラウザでファイルを作り、コミットできる GitLab のエディター。この資料のファイル作成は、すべて Web IDE で行う。 |
| **Source Control 認証情報** | 認証情報タイプ `Source Control` | AAP が GitLab のリポジトリを読むときの認証情報。この資料では、GitLab のログインのユーザー名とパスワードを入れる。 |
| **Merge Request** | **Merge requests**（GitLab） | ブランチの変更を、別のブランチに取り込む依頼。[応用演習6](./advanced/ex6.md) では、これを合図にジョブを起動する。 |

---

## 9. 参考（この資料の演習では使わない用語）

| 用語 | 説明 |
|---|---|
| **Automation Hub** | Red Hat 認定のコレクションと実行環境の配布元。社内に置く private automation hub もあり、カスタム実行環境の置き場所としても使える。 |
| **Automation Mesh** | 複数の実行ノードをつないでジョブを動かす、AAP のネットワーク構成。 |
| **Instance Group** | ジョブを動かすノードの集まり。負荷の分散や、特定のネットワークに届くノードでジョブを動かすときに使う。 |
| **Job Slicing** | 1つのジョブを、ホストを分けた複数のジョブに分けて並列に動かす機能。ホストが多いときに使う。 |
| **Forks** | 1つのジョブの中で、同時に処理するホストの数の上限。 |
| **Constructed Inventory** | 既存のインベントリーから、条件で絞ったホストを集めたインベントリー。以前の Smart Inventory に代わるもの。画面では **構築済みインベントリー**。 |
| **ansible-navigator** | 実行環境を使って、手元で Playbook を実行・確認するコマンド。 |
| **Vault ID** | 複数の Vault のパスワードを使い分けるときの識別子。認証情報と暗号化した値の両方で指定する。 |
| **CI/CD との連携** | GitLab CI などから、API や `ansible.controller` でジョブテンプレートを起動する構成。 |
