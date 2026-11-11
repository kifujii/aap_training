# 応用演習 4 - カスタム実行環境

[応用演習3](./ex3.md) | [応用演習5へ](./ex5.md)

実行環境を1つ作り、AAP に登録します。定義ファイルは AAP サーバの `/tmp/custom-ee` に置きます。

次のどちらかなら、この演習は飛ばして既定の実行環境のまま [応用演習5](./ex5.md) へ進みます。

* AAP サーバからコンテナイメージを取得できない
* `ansible-builder` も `podman` も無い

## Section 1: AAPサーバでツールを確認する

```bash
ssh <ユーザー>@<AAPサーバ>
cd /tmp
ansible-builder --version
podman --version
```

ホストに無い場合は、`podman ps` で controller を含むコンテナを探し、その中に `ansible-builder` があるかを見ます。どちらにも無ければ、この演習はここで終わります。

## Section 2: 定義ファイル

`/tmp/custom-ee` を作り、その中に定義ファイルを置きます。

```bash
mkdir -p /tmp/custom-ee && cd /tmp/custom-ee
```

`requirements.yml`:

```yaml
---
collections:
  - name: community.general
    version: "8.6.1"
```

`execution-environment.yml`:

```yaml
---
version: 3

images:
  base_image:
    name: registry.redhat.io/ansible-automation-platform-27/ee-supported-rhel9:latest

dependencies:
  galaxy: requirements.yml
```

ベースは `ee-supported` にします。`ee-minimal` から作ると、既定のイメージに入っている `ansible.windows` や `amazon.aws` が欠けることがあります。`amazon.aws` をこのファイルへ足す場合も、版は `10.3.0` です。

## Section 3: ビルドする

```bash
cd /tmp/custom-ee
ansible-builder build \
    -f execution-environment.yml \
    -t custom-ee:1.0 \
    --container-runtime podman
podman images | grep custom-ee
```

## Section 4: AAPへ登録する

レジストリへ push できる場合は、push したイメージ名を登録します。AAP の実行ノードがこの AAP サーバ自身である場合は、push せずに次の値でも登録できます。

| 項目 | 入力値 |
|---|---|
| 名前 | `custom-ee` |
| Image | `localhost/custom-ee:1.0` |
| Pull | `Never` |

**実行環境** の作成画面で保存します。レジストリを使う場合は、認証情報タイプ `Container Registry` を作り、Pull は `Always`、Image は push した名前にします。

## Section 5: テンプレートへ付ける

`write-note-linux` の実行環境に `custom-ee` を選び、起動して成功することを確認します。ジョブ詳細の実行環境が `custom-ee` になっていれば、指定は効いています。

確認が終わったら、実行環境を空欄に戻します。以降の演習は既定の実行環境で進めます。

## 次

[応用演習5](./ex5.md)
