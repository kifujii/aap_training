# sample-content

本資料の演習（[基礎演習4](../basic-controller/ex4.md)以降）で使用するサンプルAnsibleコンテンツです。操作対象の用意方は [操作対象サーバ（AWS）](../aws_targets.md) を参照してください。

## 構成

```
sample-content/
├── inventory.yml          # SCMのInventory Sourceで読み込むインベントリ（基礎演習5）
├── group_vars/all.yml     # タイムゾーン、パッケージ、EC2用変数の既定値
├── requirements.yml       # provision_ec2.yml が使う amazon.aws
├── provision_ec2.yml      # 経路A。EC2を作成する。既存サーバでは実行しない
├── site.yml               # 登録済みホストへ初期設定を適用する（基礎演習6）
├── verify.yml             # 時刻同期、chronyd、ディスク空きの点検
├── patch.yml              # 更新一覧の確認。適用は patch_apply=true のときだけ
└── roles/
    ├── baseline/          # タイムゾーン、パッケージ、chronyd、MOTD、管理ユーザー
    └── access/            # sshd の接続制限
```

`site.yml` はサーバを作りません。`provision_ec2.yml` だけがEC2を作成します。

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
2. 左側のファイルツリーで「New file」をクリックし、ファイルパスを入力して1つずつ作成する（例: `inventory.yml`、`site.yml`、`roles/baseline/tasks/main.yml` のようにディレクトリを含むパスを入力すると、自動的にフォルダも作成される）
3. 各ファイルの内容は、このGitHub上の研修資料リポジトリ（またはクローン済みのローカルコピー）を別タブで開き、該当ファイルの内容をコピー＆ペーストする
4. 必要なファイル（`inventory.yml`、`group_vars/all.yml`、`site.yml`、`verify.yml`、`patch.yml`、`provision_ec2.yml`、`requirements.yml`、`roles/baseline/**`、`roles/access/**`）をすべて作成したら、左側のサイドバーの「Source Control」アイコンを開き、コミットメッセージ（例: `Initial commit: sample content for AAP training`）を入力して「Commit to main」をクリックする

**Note:** Web IDEにはファイルを丸ごとドラッグ＆ドロップでアップロードする機能（GitLabのバージョンによって対応状況が異なります）もあります。使える場合は、ローカルにcloneした`sample-content/`フォルダをまとめてドラッグ＆ドロップすることで、1ファイルずつ作成する手間を省けます。

3. 経路Bでは、`inventory.yml` の `ansible_host` を既存サーバのIPまたはDNS名に書き換えてpushする。経路Aでは、`provision_ec2.yml` 実行後にログのプライベートIPへ書き換えて、もう一度pushする。

> **Note:** 以降の演習（基礎演習4〜、応用演習、宿題）は、このリポジトリがAAPのProjectとして登録されていることを前提に進みます。
