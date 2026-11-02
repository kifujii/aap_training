# 応用演習 6 - カスタムEEの作成

この演習では、`ansible-builder` を使って独自のCollectionを含むカスタムExecution Environment（EE）を作成し、AAPに登録します。

**前提条件:** [応用演習5](./ex5.md)を完了していること、かつ以下の環境が利用可能であること（[事前準備](../prepare_training_environment.md)参照）

* `podman` または `docker` が使えるLinuxサーバー
* インターネット接続（ベースイメージ・Collectionの取得のため）
* コンテナイメージをpushできるコンテナレジストリ

> **注意:** この演習はAAPの画面操作ではなく、別のLinuxサーバー上でのCLI作業が中心です。

[前に戻る](./ex5.md) | [次へ進む](./ex7.md)

---

## Section 1: 必要なツールの確認・導入

```bash
$ python3 -m pip install --user ansible-builder ansible-navigator
$ ansible-builder --version
$ ansible-navigator --version
$ podman --version
```

いずれかが導入されていない場合は、上記の `pip install` コマンドで導入してください（社内ミラー等、環境に応じてインデックスURLを調整してください）。

## Section 2: execution-environment.ymlの作成

### Step 1: 作業ディレクトリを作成する

```bash
$ mkdir ~/custom-ee && cd ~/custom-ee
```

### Step 2: 必要なCollectionを定義する

```bash
$ cat << 'EOF' > requirements.yml
---
collections:
  - name: community.general
    version: ">=8.0.0"
  - name: ansible.posix
EOF
```

### Step 3: execution-environment.ymlを作成する

```bash
$ cat << 'EOF' > execution-environment.yml
---
version: 3

images:
  base_image:
    name: registry.redhat.io/ansible-automation-platform-25/ee-minimal-rhel9:latest

dependencies:
  galaxy: requirements.yml
EOF
```

**Note:** `base_image` にはAAPが提供する最小構成のEEイメージ（`ee-minimal`）を指定し、そこに必要なCollectionだけを追加していくのが推奨される方法です。社内レジストリにミラーされたベースイメージがある場合は、そのイメージ名に置き換えてください。

## Section 3: ansible-builderでビルドする

### Step 1: ビルドを実行する

```bash
$ ansible-builder build \
    -f execution-environment.yml \
    -t custom-ee:1.0 \
    --container-runtime podman
```

ビルドが進行すると、ベースイメージの取得、Collectionのインストール、イメージのレイヤー構築が順に実行されます。

### Step 2: ビルド結果を確認する

```bash
$ podman images | grep custom-ee
localhost/custom-ee   1.0   xxxxxxxxxxxx   2 minutes ago   512 MB
```

### Step 3: ansible-navigatorでローカル動作確認する

AAPに登録する前に、ビルドしたEEがsample-contentの `site.yml` を正しく実行できるか、ローカルで確認します。

```bash
$ cd aap-training-content   # sample-contentをpushしたリポジトリのクローン
$ ansible-navigator run site.yml \
    -i inventory.yml \
    --eei localhost/custom-ee:1.0 \
    --mode stdout
```

**Note:** `ansible-navigator` はEEコンテナの中でPlaybookを実行するためのCLIツールです。`--mode stdout` を指定すると、対話UIではなく通常のターミナル出力形式で結果を確認できます。

## Section 4: コンテナレジストリへのpush

### Step 1: レジストリにログインする

```bash
$ podman login <レジストリのホスト名>
Username: <レジストリのユーザー名>
Password: <レジストリのパスワードまたはトークン>
```

### Step 2: タグ付けしてpushする

```bash
$ podman tag custom-ee:1.0 <レジストリのホスト名>/aap-training/custom-ee:1.0
$ podman push <レジストリのホスト名>/aap-training/custom-ee:1.0
```

## Section 5: AAPへの登録

### Step 1: Container Registry Credentialを作成する

**Automation Execution > Credentials** →「Add」で、以下を入力します。

| 項目 | 入力値 |
|---|---|
| Name | `custom-ee-registry-credential` |
| Credential Type | `Container Registry` |
| Authentication URL | レジストリのホスト名 |
| Username / Password (or Token) | レジストリのログイン情報 |

### Step 2: Execution Environmentを登録する

**Automation Execution > Infrastructure > Execution Environments** →「Add」で、以下を入力します。

| 項目 | 入力値 |
|---|---|
| Name | `custom-ee` |
| Image | `<レジストリのホスト名>/aap-training/custom-ee:1.0` |
| Pull | `Always`（研修中は更新を確実に反映するため） |
| Registry Credential | `custom-ee-registry-credential` |

「Save」をクリックして保存します。

## Section 6: Job Templateへの割当と実行確認

### Step 1: Job Templateに割り当てる

`deploy-web-stack` の編集画面で「Execution Environment」欄に `custom-ee` を選択し、保存します。

### Step 2: 実行して確認する

「Launch」してJobを実行し、正常終了することを確認します。Job詳細画面の「Details」タブで、使用されたExecution Environmentが `custom-ee` になっていることを確認してください。

### Step 3: 動作を元に戻す（任意）

研修終了後は、意図せず他の演習に影響しないよう、Execution Environmentの指定を空欄（デフォルトに戻す）にしておくとよいでしょう。

---

ここまでで、カスタムEEのビルドからAAPへの登録・利用までの一連の流れを習得しました。次の演習では、REST APIを使ったAAPの操作を扱います。

[次へ進む](./ex7.md)
