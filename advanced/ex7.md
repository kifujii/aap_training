# 応用演習 7 - REST APIの活用

この演習では、AAPのREST APIを使ってJob Templateの起動やJobの状態取得を行います。UIで行っている操作の多くはAPI経由でも実行できることを体験します。

**前提条件:** 基礎演習1〜7を完了していること

[前に戻る](./ex6.md) | [次へ進む](./ex8.md)

---

## Section 1: APIのルートとバージョンについて

AAPのREST APIは、バージョンや構成によって公開パスが異なる場合があります。はじめに、お使いの環境のAPIルートを確認してください。

```bash
$ curl -sk https://<AAPのURL>/api/ | python3 -m json.tool
```

代表的なパスの例（環境によって異なります）。

| パス例 | 内容 |
|---|---|
| `/api/controller/v2/` | AAP 2.5以降のController API。2.7のゲートウェイ構成ではこちらを使う |
| `/api/v2/` | 旧Controller単体のパス。ゲートウェイ構成では 404 になる |
| `/api/gateway/v1/` | AAPゲートウェイ（認証・ユーザー管理等）のAPI。`/api/gateway/v1/ping/` でプラットフォームのバージョンを確認できる |

本資料では、以降 `<API_ROOT>` という表記で、お使いの環境に合わせたAPIベースパスを表すことにします（例: `https://aap.example.local/api/controller/v2`）。

## Section 2: APIトークンの発行

### Step 1: UIからトークンを発行する

**Access Management > API Tokens** を開き、**Create API token** をクリックします。

| 項目 | 入力値 |
|---|---|
| Application | 空欄でよい（Personal Access Token） |
| Scope | `Write`（起動操作を行うため） |

発行されたトークン文字列を控えます（再表示できないため、必ずこの場でコピーしてください）。

### Step 2: トークンを環境変数に設定する

```bash
$ export AAP_TOKEN="<発行したトークン>"
$ export AAP_API_ROOT="https://<AAPのURL>/api/controller/v2"
```

## Section 3: curlでJob Template一覧を取得する

```bash
$ curl -sk -H "Authorization: Bearer ${AAP_TOKEN}" \
    "${AAP_API_ROOT}/job_templates/" | python3 -m json.tool
```

レスポンス中の `results` 配列から、[基礎演習6](../basic-controller/ex6.md)で作成した `apply-baseline` の `id` を確認してください。

```json
{
  "id": 7,
  "name": "apply-baseline",
  "...": "..."
}
```

## Section 4: curlでJob Templateを起動する

### Step 1: Launch APIを呼び出す

```bash
$ curl -sk -X POST \
    -H "Authorization: Bearer ${AAP_TOKEN}" \
    -H "Content-Type: application/json" \
    -d '{}' \
    "${AAP_API_ROOT}/job_templates/7/launch/" | python3 -m json.tool
```

レスポンスに起動したJobの `id` が含まれます。

```json
{
  "job": 42,
  "...": "..."
}
```

### Step 2: Extra VariablesとLimitを指定して起動する

Survey/Prompt on Launchを有効にしている場合は、`extra_vars` や `limit` をリクエストボディに含めて渡せます。

```bash
$ curl -sk -X POST \
    -H "Authorization: Bearer ${AAP_TOKEN}" \
    -H "Content-Type: application/json" \
    -d '{
          "extra_vars": {"host_timezone": "UTC"},
          "limit": "srv1"
        }' \
    "${AAP_API_ROOT}/job_templates/7/launch/" | python3 -m json.tool
```

**Note:** [基礎演習7](../basic-controller/ex7.md)で設定したSurveyの回答変数も、同じ `extra_vars` の形式でAPI経由から渡すことができます。UIのSurvey入力フォームは、内部的にはこのAPIリクエストを生成しているとイメージすると理解しやすいです。

## Section 5: Jobの状態を取得する

### Step 1: Jobの詳細を取得する

```bash
$ curl -sk -H "Authorization: Bearer ${AAP_TOKEN}" \
    "${AAP_API_ROOT}/jobs/42/" | python3 -m json.tool
```

レスポンスの `status` フィールドで、`pending` → `running` → `successful`（または `failed`）という遷移を確認できます。

### Step 2: ポーリングして完了を待つ（簡易スクリプト）

```bash
$ while true; do
    status=$(curl -sk -H "Authorization: Bearer ${AAP_TOKEN}" \
      "${AAP_API_ROOT}/jobs/42/" | python3 -c "import sys,json;print(json.load(sys.stdin)['status'])")
    echo "status: ${status}"
    [ "${status}" = "successful" ] || [ "${status}" = "failed" ] && break
    sleep 5
  done
```

### Step 3: 実行ログ（stdout）を取得する

```bash
$ curl -sk -H "Authorization: Bearer ${AAP_TOKEN}" \
    "${AAP_API_ROOT}/jobs/42/stdout/?format=txt"
```

## Section 6: APIドキュメントの参照

ブラウザでAPIルート（例: `https://<AAPのURL>/api/controller/v2/`）に直接アクセスすると、Browsable API（HTML形式のAPIドキュメント兼簡易テストフォーム）が表示されます。各エンドポイントで利用可能なHTTPメソッドやパラメータを確認できます。

---

ここまでで、REST APIを使ったJob Templateの起動・状態確認ができるようになりました。次の演習では、Playbookの中からAAPを操作するCLI/collectionの活用方法を扱います。

[次へ進む](./ex8.md)
