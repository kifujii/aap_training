# sample-content

本資料の演習（[基礎演習4](../basic-controller/ex4.md)以降）で使用するサンプルAnsibleコンテンツです。

## 構成

```
sample-content/
├── inventory.yml       # SCMベースのInventory Sourceで読み込むYAMLインベントリ（基礎演習5）
├── site.yml            # Job Templateで実行するメインPlaybook（基礎演習6）
├── verify.yml           # デプロイ後の確認用Playbook（応用演習4, 7, 8, 10で使用）
└── roles/
    ├── nginx/           # Webサーバー構築用role
    └── haproxy/         # ロードバランサー構築用role
```

## 使い方

本資料のGitLab関連演習は、**git CLIが使える環境（PCまたはLinuxサーバ）での操作を基本**としています。CLIが使えない、あるいはCLIを使わずに完結させたい場合は、GitLabの**Web IDE**（ブラウザ上のエディタ）で同じ作業ができます。どちらの方法でも以降の演習には影響しません。

1. GitLabに演習用のProject（リポジトリ）を作成する（[基礎演習4](../basic-controller/ex4.md)）
2. このディレクトリの内容をそのProjectにpushする

### 方法A: git CLIで行う場合（基本）

```bash
$ cd sample-content
$ git init
$ git remote add origin <GitLabプロジェクトのURL>
$ git add .
$ git commit -m "Initial commit: sample content for AAP training"
$ git branch -M main
$ git push -u origin main
```

### 方法B: GitLab Web IDEで行う場合（代替）

git CLIを使わず、ブラウザだけで完結させたい場合はこちらの方法を使います。

1. 作成したGitLab Projectのトップ画面を開き、「Edit」ボタン（またはプロジェクト名の下にあるドロップダウン）から「Web IDE」を選択して開く
2. 左側のファイルツリーで「New file」をクリックし、ファイルパスを入力して1つずつ作成する（例: `inventory.yml`、`site.yml`、`roles/nginx/tasks/main.yml` のようにディレクトリを含むパスを入力すると、自動的にフォルダも作成される）
3. 各ファイルの内容は、このGitHub上の研修資料リポジトリ（またはクローン済みのローカルコピー）を別タブで開き、該当ファイルの内容をコピー＆ペーストする
4. 必要なファイル（`inventory.yml`、`site.yml`、`verify.yml`、`roles/nginx/**`、`roles/haproxy/**`）をすべて作成したら、左側のサイドバーの「Source Control」アイコンを開き、コミットメッセージ（例: `Initial commit: sample content for AAP training`）を入力して「Commit to main」をクリックする

**Note:** Web IDEにはファイルを丸ごとドラッグ＆ドロップでアップロードする機能（GitLabのバージョンによって対応状況が異なります）もあります。使える場合は、ローカルにcloneした`sample-content/`フォルダをまとめてドラッグ＆ドロップすることで、1ファイルずつ作成する手間を省けます。

3. `inventory.yml` 内のホスト名・IPアドレスを、実際の管理対象ノードの値に書き換えてpushする（[事前準備](../prepare_training_environment.md)で確認した値を使用）。Web IDEの場合も同様に、エディタ上で直接書き換えてコミットすれば反映されます。

> **Note:** 以降の演習（基礎演習4〜、応用演習、宿題）は、このリポジトリがAAPのProjectとして登録されていることを前提に進みます。
