# 応用演習 4 - REST API

[応用演習3](./ex3.md) | [応用演習5へ](./ex5.md)

> **前提:** [演習8](../basic-controller/ex8.md) の制限の起動プロンプトと Survey

ジョブの実行を、API からも行います。ここで発行するのは AAP の API トークンです。GitLab のパスワードとは別です。

この演習のコマンドは、[事前準備](../prepare_training_environment.md) で控えた作業用サーバで実行します。

## Section 1: APIの場所

```bash
curl -sk https://<AAPのURL>/api/ | python3 -m json.tool
```

出力の `apis` に、AAP の中の API の場所が並びます。`-k` は、AAP の証明書を検証しない指定です。自己証明書の環境でも接続できるようにするために付けます。実際の運用では、CA 証明書を指定して検証します。

この資料の例は `https://<AAPのURL>/api/controller/v2` です。`/api/v2/` は、ゲートウェイ構成では 404 になります。以降、このベースを `AAP_API_ROOT` と書きます。

## Section 2: トークン

**Access Management > API Tokens** を開き、**Create API token** をクリックします。

![トークン作成。説明は任意、OAuth application は空欄、Scope は Write。Create token を押す前の画面](../images/aap/api-token-create.png)

| 項目 | 入力値 |
|---|---|
| 説明 | 空欄で構いません |
| OAuth application | 空欄 |
| Scope | `Write` |

**Create token** を押すと、文字列が1度だけ表示されます。再表示できないので、その場で控えます。控えたあと、画面を閉じます。

トークンは、コマンドの引数に書かずに入力します。引数に書くと、シェルの履歴に残ります。1行目を実行すると入力待ちになるので、控えたトークンを貼り付けて Enter を押します。入力した文字は画面に出ません。

```bash
read -rsp 'AAP token: ' AAP_TOKEN; echo
export AAP_TOKEN
export AAP_API_ROOT="https://<AAPのURL>/api/controller/v2"
```

## Section 3: テンプレートのIDを見る

```bash
curl -sk -H "Authorization: Bearer ${AAP_TOKEN}" \
    "${AAP_API_ROOT}/job_templates/?name=<name_prefix>-write-note-linux" | python3 -m json.tool
```

`results` の `id` を控えます。以降の `<テンプレートID>` は、その数字に置き換えます。

## Section 4: 起動する

```bash
curl -sk -X POST \
    -H "Authorization: Bearer ${AAP_TOKEN}" \
    -H "Content-Type: application/json" \
    -d '{"extra_vars": {"training_note": "from api"}, "limit": "lin1"}' \
    "${AAP_API_ROOT}/job_templates/<テンプレートID>/launch/" | python3 -m json.tool
```

応答の `job` がジョブ ID です。

```bash
curl -sk -H "Authorization: Bearer ${AAP_TOKEN}" \
    "${AAP_API_ROOT}/jobs/<ジョブID>/" | python3 -m json.tool
```

`status` が `successful` になるまで、同じコマンドを繰り返します。ジョブ出力の `Show the note written to note.txt` に `note.txt の note は from api です` と出れば、API からの Extra variables が届いています。

確認後、`<name_prefix>-write-note-linux` を起動してメモを戻します。**Prompts** の **制限** は空欄のまま **Next** を押し、**Survey** は `managed by AAP` のまま **Finish** を押します。

## 次

[応用演習5](./ex5.md)
