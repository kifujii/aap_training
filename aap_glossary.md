# AAP用語集

---

## 1. 基本概念

| 用語 | 説明 |
|---|---|
| **Ansible Automation Platform (AAP)** | Red Hatが提供する自動化の統合プラットフォーム。OSS版Ansibleの実行エンジンに、Web UI・RBAC・API・スケジューリング・イベント駆動自動化（EDA）などのエンタープライズ機能を加えたもの。 |
| **Automation Controller** | AAPの中核コンポーネント。Playbookの実行管理（Job Template、Workflow、Inventory、Credential等）を担う。旧称 Ansible Tower。2.5以降のUIでは「**Automation Execution**」メニュー配下に相当する機能が集約されている。 |
| **Event-Driven Ansible (EDA)** | イベント（監視アラート、Webhook等）を検知し、条件に応じて自動的にアクション（Job Template起動等）を実行する仕組み。2.5以降のUIでは「**Automation Decisions**」メニュー配下に相当する。 |
| **統合UI** | AAP 2.5以降で採用された、Controller・EDAを1つのナビゲーションに統合したWeb UI。大きく **Automation Execution**（ジョブ / テンプレート / プロジェクト / Infrastructure 配下のインベントリー・認証情報・実行環境）、**Automation Decisions**（ルールブックのアクティベーション / 決定環境 / Event Streams）、**Access Management**（組織 / チーム / ユーザー）に分かれる。日本語UIでの項目名は [画面ラベル対応表](./ui-labels.md) を参照。 |
| **Automation Hub** | Red Hat が提供する Collection の配布基盤。 |
| **Automation Mesh** | 複数の Execution Node を束ねて Job を実行するネットワーク構成。 |

---

## 2. Controller中心オブジェクト

| 用語 | 説明 |
|---|---|
| **Organization** | AAPにおける最上位の管理区分。User/Team/Project/Inventory/Credential等はいずれかのOrganizationに属する。マルチテナント運用の基本単位。 |
| **User** | AAPにログインする個々のアカウント。System Administrator等のシステムロールと、Organization/Team配下のロールを持つ。 |
| **Team** | 複数のUserをまとめるグループ。Teamに対してロールを付与することで、所属User全員に同じ権限を一括付与できる。 |
| **Credential** | 接続先ホストへのSSH認証情報や、SCM（GitLab等）、Vault、クラウドAPIキーなど、Job実行に必要な認証情報を保存・管理するオブジェクト。**Credential Type**ごとに入力項目が異なる。 |
| **Credential Type** | Credentialの種類を定義するテンプレート（例: Machine、Source Control、Vault、Container Registry）。CredentialはいずれかのCredential Typeに基づいて作成される。 |
| **Project** | 実行対象のPlaybook/Role一式を管理する単位。多くの場合GitLab等のSCMリポジトリと連携（**SCM Project**）し、Sync操作でリポジトリの最新内容を取得する。 |
| **SCM Sync** | Projectに紐付いたSCMリポジトリの内容を、AAP側に取得・反映する操作。画面の同期で実行します。GitLab の Webhook では同期は始まりません（[応用演習6](./advanced/ex6.md)）。 |
| **Revision** | SCM Sync時に取得したリポジトリのコミットハッシュ。Job実行時にどのリビジョンのPlaybookが使われたかを追跡できる。 |
| **Inventory** | Job実行対象のホストとグループを定義する単位。手動でHost/Groupを登録する方法と、**Inventory Source**を使って外部（SCM上のYAMLファイル、クラウドAPI等）から取り込む方法がある。 |
| **Inventory Source** | Inventoryにホスト情報を取り込む方法を定義する設定。本資料では GitLab 上の `inventory.yml` を読む「Sourced from a Project」を使います（[演習6](./basic-controller/ex6.md)）。 |
| **Smart Inventory** | 既存Inventoryのホストを、検索条件（ホスト変数など）に基づいて動的にグルーピングした仮想的なInventory。 |
| **Job Template** | 「どのProjectのどのPlaybookを」「どのInventoryに対して」「どのCredentialを使って」実行するかを定義した、実行可能なテンプレート。AAPにおける最も基本的な実行単位。 |
| **Job** | Job Templateを実際に実行した1回分の記録。実行ログ・PLAY RECAP・ステータス（successful/failed等）を持つ。 |
| **Workflow Job Template** | 複数のJob Template（またはWorkflow内のApproval等）をノードとして繋ぎ、成功/失敗に応じて分岐させながら実行する仕組み。 |
| **Workflow Approval** | Workflow内に設置できる「人による承認」ノード。承認されるまで後続ノードの実行を保留する。 |
| **Ad Hoc Command** | Playbookを介さず、Inventory上のホストに対してモジュールを1回限り直接実行する機能（OSS版の `ansible <host> -m <module>` に相当）。 |

---

## 3. 実行管理・実行制御

| 用語 | 説明 |
|---|---|
| **Extra Variables (Extra Vars)** | Job Template実行時に渡す追加変数。OSS版の `-e` オプションに相当し、優先順位は非常に高い。 |
| **Limit** | 実行対象をInventory内の特定ホスト/グループに絞り込む指定。OSS版の `--limit` に相当。 |
| **Verbosity** | 実行ログの詳細度。OSS版の `-v`〜`-vvvv` に相当するレベルをUI上で選択する。 |
| **Forks** | 並列実行するホスト数の上限。 |
| **Job Type** | Job Templateの実行種別。「Run」（実際に変更を適用）と「Check」（OSS版の `--check` に相当するドライラン）がある。 |
| **Prompt on Launch** | Job Template起動時に、Inventory/Credential/Extra Vars/Limit等の値をオペレーターに入力・選択させる設定。Job Template自体には固定値を持たせず、実行時に都度指定したい場合に使う。 |
| **Survey** | Job Template起動時に、あらかじめ定義した質問フォームを通じて変数値を入力させる機能。入力値はExtra Varsとして渡される。 |
| **Schedule** | Job Template/Workflow Job TemplateをCron形式の設定で定期実行するための機能。 |
| **Notification Template** | Jobの開始/成功/失敗等のタイミングで、Email・Slack・Webhook等に通知を送る設定。Organization/Project/Job Template等の単位で割り当てる。 |
| **Instance Group** | Jobを実行するExecution Nodeの集合。実行負荷の分散や特定ネットワークへの到達性に応じてJobの実行先を制御する際に使う。 |
| **Job Slicing** | 1つのJob Templateの実行を、Inventory内のホストを分割して複数Jobに並列展開する機能。大規模Inventory向け。 |
| **PLAY RECAP** | Job詳細画面に表示される、OSS版Ansibleと同じ形式の実行結果サマリー（ok/changed/failed等）。 |

---

## 4. Execution Environment関連

| 用語 | 説明 |
|---|---|
| **Execution Environment (EE)** | Job実行時に使用される、Ansible本体・Python依存パッケージ・Collectionなどを含んだコンテナイメージ。従来の「コントローラーへの直接インストール」に代わり、実行環境をコンテナとして分離・バージョン管理する仕組み。 |
| **Default Execution Environment** | AAPにあらかじめ組み込まれている標準のEE。多くのJob Templateはこれを使って動作確認できる。 |
| **ansible-builder** | `execution-environment.yml` の定義に基づいて、カスタムEEのコンテナイメージをビルドするCLIツール。 |
| **execution-environment.yml** | カスタムEEに含めるCollection（`requirements.yml`）・Pythonパッケージ（`requirements.txt`）・OSパッケージ（`bindep.txt`）などを定義するansible-builderの設定ファイル。 |
| **Container Registry Credential** | ビルドしたEEイメージをpush/pullするコンテナレジストリの認証情報を保持するCredential Type。 |
| **ansible-navigator** | EEを使ってPlaybookをローカル/CLIから実行・デバッグするためのツール。AAPに登録する前のEE動作確認にも使える。 |
| **Decision Environment (DE)** | EDAのRulebook実行に使われるコンテナイメージ。EEと似た位置づけだが、EDA（`ansible-rulebook`等）の実行に必要なコンポーネントを含む。 |

---

## 5. RBAC・セキュリティ

| 用語 | 説明 |
|---|---|
| **RBAC (Role-Based Access Control)** | User/Teamに対してロールを割り当てることで、Organization/Project/Inventory/Job Template等のオブジェクトへの操作権限を制御する仕組み。 |
| **ビルトインロール** | AAPが標準で用意しているロール。画面では `JobTemplate Execute` や `Project Read Compat` のように、オブジェクト名が前に付く。付ける手順は[応用演習1](./advanced/ex1.md)。 |
| **オブジェクト単位の権限** | 特定のProject・Inventory・Job Template等、個別のオブジェクトに対して直接ロールを付与する方式。Organization全体への権限付与より細かい制御が可能。 |
| **System Administrator** | AAP全体に対する最上位権限を持つシステムロール。 |
| **ansible-vault** | 機密の値を暗号化するコマンド。この資料では `ansible-vault encrypt_string` で値だけを暗号化し、ファイル自体は YAML のまま GitLab に置きます。復号はジョブに付けた Vault 認証情報で行います。 |
| **Vault Credential** | `ansible-vault encrypt_string` で暗号化した値を復号するためのパスワードを保持する認証情報。Job Template に付けると、ジョブの中で値が復号されます。 |
| **Vault ID** | 複数のVaultパスワードを使い分ける際に使う識別子。Credential側・暗号化ファイル側の双方で指定する。 |

---

## 6. API・CLI

| 用語 | 説明 |
|---|---|
| **Controller REST API** | AAPのほぼ全機能をHTTP経由で操作できるAPI。UIで行える操作の大部分はAPI経由でも実行可能。 |
| **APIトークン (OAuth2 Token)** | REST APIを呼び出す際の認証に使うトークン。UIまたはAPI経由で発行し、`Authorization: Bearer <token>` ヘッダーで利用する。 |
| **ansible.controller collection** | Playbook から Controller API を操作するモジュール群（例: `ansible.controller.job_launch`）。既定の実行環境に入っています。応用演習5では、このコレクションを使う Playbook をジョブとして実行します。 |
| **CI/CDパイプライン連携** | GitLab CI等から、Controller REST APIやansible.controller collectionを介してJob Templateを起動するなど、外部の自動化パイプラインとAAPを連携させる構成。 |

---

## 7. Event-Driven Ansible (EDA) 用語

| 用語 | 説明 |
|---|---|
| **Rulebook** | EDAにおける「どのイベントを」「どんな条件で」「どのアクションに繋げるか」をYAML形式で定義したファイル。OSS版Playbookに相当する存在。 |
| **Source** | Rulebookがイベントを受け取る入力元（例: Webhookの受信、ファイルの変更監視、Kafka等）を定義する部分。 |
| **Rule** | Sourceから受け取ったイベントに対する条件（condition）と、条件成立時に実行するアクション（action）の組。 |
| **Condition** | ルールの発火条件。イベントのペイロード内容に対する比較式などで記述する。 |
| **Action** | 条件成立時に実行する処理。Job Templateの起動（`run_job_template`）やデバッグ出力（`debug`）などがある。 |
| **Event Stream** | 外部システム（監視ツール等）からのWebhookイベントをEDAに受け渡すための受信エンドポイント設定。 |
| **Rulebook Activation** | Rulebookを実際に稼働状態にする（イベント待受を開始する）ためのオブジェクト。使用するDecision Environment・Rulebook・関連Credential等を指定する。 |
| **ansible-rulebook** | Rulebookを実行するCLIエンジン。Rulebook ActivationはAAP内部でこれを使ってRulebookを稼働させる。 |

---

## 8. SCM・GitLab連携

| 用語 | 説明 |
|---|---|
| **SCM (Source Control Management)** | Gitなどのバージョン管理システムの総称。AAPのProjectはSCMリポジトリと連携して最新のPlaybook/Role/Inventoryファイルを取得する。 |
| **Source Control認証** | AAPがGitLabを読むときに使う認証情報。GitLabのログインユーザー名とパスワードを登録する。 |
| **Webhook** | 外部の出来事を HTTP で受けてジョブを起動する仕組み。応用演習6では、ジョブテンプレートの Webhook で GitLab のマージリクエストを受けます。この受信ではプロジェクトの同期は始まりません。EDA の受け口は Event Stream で、応用演習7以降です。 |
| **.gitlab-ci.yml** | GitLab CI/CD パイプラインの定義ファイル。 |
