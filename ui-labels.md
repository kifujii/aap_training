# 画面ラベル対応表

本資料の画面手順は、**AAP 2.7 の日本語UI** と **GitLab 19 の英語UI** で確認しています。AAP 2.5 以降でもオブジェクトの役割は同じですが、メニュー名はバージョンと言語で表記が変わります。英語UIの場合は、括弧内の英語名を手がかりにしてください。

画面上部にサブスクリプション期限の黄色い帯が出ることがあります。演習手順とは無関係なので、読み飛ばしてください。一覧に Demo Project や Demo Inventory のような既存オブジェクトがあっても問題ありません。演習では別名で新規作成します。

## AAPの左メニュー

| グループ | 開くと出てくる項目 |
|---|---|
| **Overview** | ログイン直後の概況 |
| **Automation Execution** | **ジョブ**、**テンプレート**、**スケジュール**、**プロジェクト**、**Infrastructure**、**管理** |
| Infrastructure の中 | Topology View、**インベントリー**、ホスト、インスタンスグループ、インスタンス、**実行環境**、**認証情報**、Credential Types |
| 管理 の中 | Activity Stream、Deprecations、Workflow Approvals、**Notifiers**、Management Jobs |
| **Automation Decisions** | **ルール監査**、**ルールブックのアクティベーション**、プロジェクト、**決定環境**、**Event Streams**、Infrastructure |
| **Access Management** | Authentication Methods、**組織**、**チーム**、**ユーザー**、ロール、OAuth Applications、**API Tokens** |
| **設定** | Subscription、Platform gateway、Feature Flags、User Preferences |

よく使う画面への入り方は次のとおりです。

| やりたいこと | メニュー |
|---|---|
| 認証情報を作る | Automation Execution > Infrastructure > **認証情報** |
| インベントリーを作る | Automation Execution > Infrastructure > **インベントリー** |
| Projectを作る | Automation Execution > **プロジェクト** |
| Job Templateを作る | Automation Execution > **テンプレート** |
| 実行履歴を見る | Automation Execution > **ジョブ** |
| 組織・チーム・ユーザー | Access Management > **組織** / **チーム** / **ユーザー** |
| APIトークン | Access Management > **API Tokens** |
| 通知の定義 | Automation Execution > 管理 > **Notifiers** |
| Execution Environment | Automation Execution > Infrastructure > **実行環境** |
| Rulebook Activation | Automation Decisions > **ルールブックのアクティベーション** |

## ボタン名

資料中の操作は、次のボタン名で進めてください。

| 操作 | 画面上のボタン |
|---|---|
| 認証情報の新規作成 | **認証情報の作成** |
| インベントリーの新規作成 | **インベントリーの作成** を開き、もう一度 **インベントリーの作成**（スマートインベントリーや構築済みインベントリーではない方） |
| Projectの新規作成 | **プロジェクトの作成** |
| Job Templateの新規作成 | **テンプレートの作成** > **ジョブテンプレートの作成** |
| Workflowの新規作成 | **テンプレートの作成** > **ワークフロージョブテンプレートの作成** |
| Job Templateの実行 | **テンプレートの起動** |
| Job Templateの変更 | **テンプレートの編集** |
| アドホック実行 | インベントリーの **ホスト** タブにある **Run command** |
| Surveyの質問追加 | **Create survey question** |
| APIトークンの発行 | **Create API token** |

インベントリー詳細のタブは **グループ**、**ホスト**、**ソース** です。Job Template詳細のタブは **詳細**、**Team Access**、**User Access**、**スケジュール**、**ジョブ**、**Survey**、**通知** です。

「起動時に値を入力させる」チェックは **起動プロンプト** です。英語UIの Prompt on launch に相当します。ログの詳細度は **詳細** で、既定値は `0 (正常)` です。

## GitLab

この確認環境のログイン後画面は英語です。言語設定が日本語の場合は、同じ位置の日本語ラベルに読み替えてください。

| 操作 | 手順 |
|---|---|
| 新規リポジトリ | 右上 **Create new…** > **New project/repository** > **Create blank project** |
| Personal Access Token | 右上のユーザーメニューから User Settings を開き、左メニュー **Personal access tokens** |

`sample-content` を `git push` する場合は、**Initialize repository with a README** のチェックを外してから **Create project** してください。チェックを付けたまま push すると、空でないリポジトリへの拒否で失敗します。Web IDE だけでファイルを作る場合は、README 初期化を付けたままでも進められます。SAST と Secret Detection は、この研修ではオフのままで構いません。
