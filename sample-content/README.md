# sample-content

本資料の演習（[基礎演習4](../basic-controller/ex4.md)以降）で使用するサンプルAnsibleコンテンツです。

## 構成

```
sample-content/
├── inventory.yml       # SCMベースのInventory Sourceで読み込むYAMLインベントリ（基礎演習5）
├── site.yml            # Job Templateで実行するメインPlaybook（基礎演習6）
└── roles/
    ├── nginx/           # Webサーバー構築用role
    └── haproxy/         # ロードバランサー構築用role
```

## 使い方

1. GitLabに演習用のProject（リポジトリ）を作成する（[基礎演習4](../basic-controller/ex4.md)）
2. このディレクトリの内容をそのProjectにpushする

```bash
$ cd sample-content
$ git init
$ git remote add origin <GitLabプロジェクトのURL>
$ git add .
$ git commit -m "Initial commit: sample content for AAP training"
$ git push -u origin main
```

3. `inventory.yml` 内のホスト名・IPアドレスを、実際の管理対象ノードの値に書き換えてpushする（[事前準備](../prepare_training_environment.md)で確認した値を使用）

> **Note:** 以降の演習（基礎演習4〜、応用演習、宿題）は、このリポジトリがAAPのProjectとして登録されていることを前提に進みます。
