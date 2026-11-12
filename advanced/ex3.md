# 応用演習 3 - カスタム実行環境

[応用演習2](./ex2.md) | [応用演習4へ](./ex4.md)

実行環境の欄が空欄のジョブは、システムの既定の実行環境で動きます。使った名前は、ジョブ詳細に出ます。ここでは、その既定にコレクションを足したイメージを1つ作り、テンプレートに付けて、ジョブ詳細の名前が変わることを確認します。定義ファイルは、`podman` と `ansible-builder` が使えるサーバに置きます。この研修では AAP サーバの `/tmp/<name_prefix>-custom-ee` です。確認が終わったら、実行環境は空欄に戻します。

次のどちらかなら、この演習は飛ばして既定の実行環境のまま [応用演習4](./ex4.md) へ進みます。

* そのサーバからコンテナイメージを取得できない
* `podman` が無い、または `ansible-builder` を入れられない

## Section 1: サーバでツールを確認する

この研修では AAP サーバへ SSH します。

```bash
ssh <ユーザー>@<AAPサーバ>
cd /tmp
podman --version
ansible-builder --version
```

`podman --version` が失敗したときは、この演習はここで終わります。

`ansible-builder --version` が失敗したときは、ホストに入れます。ビルドはこのホストで行うので、controller のコンテナの中だけにあっても、ここから先には進めません。

```bash
python3 -m pip install --user 'ansible-builder>=3,<4'
export PATH="$HOME/.local/bin:$PATH"
ansible-builder --version
```

このインストールが失敗したときも、この演習はここで終わります。

## Section 2: 定義ファイル

`/tmp/<name_prefix>-custom-ee` を作り、その中に定義ファイルを置きます。

```bash
mkdir -p '/tmp/<name_prefix>-custom-ee' && cd '/tmp/<name_prefix>-custom-ee'
```

`requirements.yml`:

```yaml
---
collections:
  - name: community.general
    version: "8.6.1"
```

`pkgmgr`:

```bash
#!/bin/bash
cmd=()
for arg in "$@"; do
  case "$arg" in
    python|python-*|python3|python3-*|dnf|dnf-*) ;;
    *) cmd+=("$arg") ;;
  esac
done
if printf '%s\n' "${cmd[@]}" | grep -qx install; then
  has_pkg=0
  for arg in "${cmd[@]}"; do
    case "$arg" in
      -*|install|clean|makecache|update|remove) ;;
      *) has_pkg=1 ;;
    esac
  done
  if [ "$has_pkg" -eq 0 ]; then
    exit 0
  fi
fi
exec /usr/bin/microdnf "${cmd[@]}"
```

```bash
chmod 755 pkgmgr
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

additional_build_files:
  - src: pkgmgr
    dest: scripts

additional_build_steps:
  prepend_base:
    - COPY _build/scripts/pkgmgr /usr/local/bin/pkgmgr
    - RUN chmod 755 /usr/local/bin/pkgmgr
    - ENV PKGMGR=/usr/local/bin/pkgmgr
  prepend_builder:
    - ENV PKGMGR=/usr/local/bin/pkgmgr
  prepend_final:
    - ENV PKGMGR=/usr/local/bin/pkgmgr

options:
  package_manager_path: /usr/local/bin/pkgmgr
  skip_pip_install: true
```

ベースは `ee-supported` にします。このイメージのパッケージ管理は `microdnf` で、Python は 3.12 です。`ansible-builder` は既定で `/usr/bin/dnf` を呼びます。`dnf` のパッケージを入れると、この Python が置き換わり、ビルドは `pip` のところで止まります。`pkgmgr` は `microdnf` を呼び、`python` と `dnf` のパッケージ名は渡しません。`pip` はベースイメージに入っているので、入れ直しません。`ee-minimal` から作ると、既定のイメージに入っている `ansible.windows` や `amazon.aws` が欠けることがあります。`amazon.aws` をこのファイルへ足す場合も、版は `10.3.0` です。

## Section 3: ビルドする

```bash
cd '/tmp/<name_prefix>-custom-ee'
export PATH="$HOME/.local/bin:$PATH"
systemctl --user show podman.service -p Environment
```

`CONTAINERS_STORAGE_CONF=` のあとにパスが出ているときは、そのパスを `<storage.conf>` に入れて、ビルドの前に実行します。出ていないときは、このコマンドは実行しません。保存先が違うと、イメージはできてもジョブが `image not known` になります。

```bash
export CONTAINERS_STORAGE_CONF=<storage.conf>
```

```bash
ansible-builder build \
    -f execution-environment.yml \
    -t '<name_prefix>-custom-ee:1.0' \
    --container-runtime podman
podman images | grep '<name_prefix>-custom-ee'
```

ビルドが終わるまで数分かかります。

## Section 4: AAPへ登録する

この研修では AAP サーバでビルドしています。その場合は push せず、次の値で登録します。レジストリへ push できる場合は、push したイメージ名を登録します。

| 項目 | 入力値 |
|---|---|
| 名前 | `<name_prefix>-custom-ee` |
| Image | `localhost/<name_prefix>-custom-ee:1.0` |
| Pull | `Never` |

**実行環境** の作成画面で保存します。レジストリを使う場合は、認証情報タイプ `Container Registry`、名前 `<name_prefix>-registry` を作り、Pull は `Always`、Image は push した名前にします。

## Section 5: 既定の実行環境で失敗し、カスタム実行環境で成功する

`community.general` は、既定の実行環境にはありません。先にその状態で失敗することを見てから、Section 4 で登録した実行環境に切り替えます。`<name_prefix>-write-note-linux` の実行環境は空欄のままにします。以降の演習は、その既定の実行環境で進めます。

Web IDE で `use_ini_file.yml` をプロジェクトの直下に作り、[sample-content/use_ini_file.yml](../sample-content/use_ini_file.yml) の内容を貼ります。**Commit to main** のあと、AAP のプロジェクトを同期します。

**ジョブテンプレートの作成** で、次を作ります。表に無い欄は空欄です。実行環境も空欄です。

| 項目 | 入力値 |
|---|---|
| 名前 | `<name_prefix>-use-ini` |
| インベントリー | `<name_prefix>-training-inventory` |
| プロジェクト | `<name_prefix>-aap-training-content` |
| Playbook | `use_ini_file.yml` |
| 実行環境 | 空欄 |
| 認証情報 | `<name_prefix>-linux-ssh` |
| Extra variables | 下の1行 |

```yaml
name_prefix: tanaka01
```

`tanaka01` は、自分の接頭辞にします。**テンプレートの起動** を押します。Survey も制限も出ません。

1回目は失敗します。出力に `community.general.ini_file` があり、`couldn't resolve module` または `was not found` と出ていれば、既定の実行環境にこのコレクションが無い、という確認はできています。対象ホストがいない、と出たときは、同じインベントリーにホスト `localhost` を追加し、変数を `ansible_connection: local` にして、もう一度起動します。

失敗を確認したら、このテンプレートの **テンプレートの編集** で、実行環境に `<name_prefix>-custom-ee` を選んで保存します。もう一度起動します。成功すると、`Show the value written with community.general` に次の行が出ます。

```
settings.ini の source は custom ee です
```

ジョブ詳細の実行環境が `<name_prefix>-custom-ee` になっていることも確認します。ジョブが `image not known` で失敗したときは、Section 3 の `CONTAINERS_STORAGE_CONF` を設定して、ビルドからやり直します。

確認が終わったら、`<name_prefix>-use-ini` の実行環境を空欄に戻します。

## 次

[応用演習4](./ex4.md)
