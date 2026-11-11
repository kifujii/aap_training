# 応用演習 5 - REST API

[応用演習4](./ex4.md) | [応用演習6へ](./ex6.md)

画面の起動を、API からも行います。ここで発行するのは AAP の API トークンです。GitLab のパスワードとは別です。

## Section 1: APIの場所

```bash
curl -sk https://<AAPのURL>/api/ | jq
```

応答は `jq` で整形します。このコマンドは、`curl` を実行する端末で動かします。

この資料の例は `https://<AAPのURL>/api/controller/v2` です。`/api/v2/` は、ゲートウェイ構成では 404 になります。以降、このベースを `AAP_API_ROOT` と書きます。

## Section 2: トークン

**Access Management > API Tokens** を開き、**Create API token** をクリックします。

![トークン作成。説明は任意、OAuth application は空欄、Scope は Write。Create token を押す前の画面](../images/aap/api-token-create.png)

| 項目 | 入力値 |
|---|---|
| 説明 | 空欄で構いません |
| OAuth application | 空欄 |
| Scope | `Write` |

**Create token** を押すと、文字列が1度だけ表示されます。再表示できないので、その場で控えます。この資料の写真には、その文字列は写していません。控えたあと、画面を閉じます。

```bash
export AAP_TOKEN="<控えたトークン>"
export AAP_API_ROOT="https://<AAPのURL>/api/controller/v2"
```

## Section 3: テンプレートのIDを見る

```bash
curl -sk -H "Authorization: Bearer ${AAP_TOKEN}" \
    "${AAP_API_ROOT}/job_templates/?name=write-note-linux" | jq
```

`results` の `id` を控えます。以降の `7` は、その数字に置き換えます。

## Section 4: 起動する

```bash
curl -sk -X POST \
    -H "Authorization: Bearer ${AAP_TOKEN}" \
    -H "Content-Type: application/json" \
    -d '{"extra_vars": {"training_note": "from api"}, "limit": "lin1"}' \
    "${AAP_API_ROOT}/job_templates/7/launch/" | jq
```

応答の `job` がジョブ ID です。

```bash
curl -sk -H "Authorization: Bearer ${AAP_TOKEN}" \
    "${AAP_API_ROOT}/jobs/<ジョブID>/" | jq
```

`status` が `successful` になるまで、同じコマンドを繰り返します。`lin1` の `note.txt` の `note:` が `from api` になっていれば、API からの Extra variables が届いています。

確認後、Survey から `managed by AAP` を入れて `write-note-linux` を一度起動し、メモを戻します。

## 次

[応用演習6](./ex6.md)
