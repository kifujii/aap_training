# 応用演習 5 - Playbook からジョブを起動する

[応用演習4](./ex4.md) | [応用演習6へ](./ex6.md)

> **前提:** [応用演習4](./ex4.md) の API トークン、[演習8](../basic-controller/ex8.md) の制限の起動プロンプトと Survey

この演習では、応用演習4で行った API の呼び出しを、Playbook を実行するジョブから行います。既定の実行環境に入っている `ansible.controller` コレクションを使って `<name_prefix>-write-note-linux` を起動するので、実行環境の欄は空欄のまま、システムの既定の実行環境を使います。

## Section 1: 起動用のインベントリー

この Playbook は `localhost` で動き、そこから AAP の API を呼び出します。そのため、起動用のインベントリーに `localhost` を登録し、変数に `ansible_connection: local` を設定します。

**インベントリーの作成** で `<name_prefix>-aap-runner` を作ります。保存したあと、**ホスト** タブの **ホストの作成** で次を追加します。

| 項目 | 入力値 |
|---|---|
| 名前 | `localhost` |
| 変数 | 下の1行 |

```yaml
ansible_connection: local
```

ホストの作成画面の欄名は **変数** で、ジョブテンプレートの Extra variables とは別の欄です。

## Section 2: AAPの認証情報

**認証情報の作成** を開き、認証情報タイプ `Red Hat Ansible Automation Platform` の認証情報を作ります。

| 項目 | 入力値 |
|---|---|
| 名前 | `<name_prefix>-aap-api` |
| Red Hat Ansible Automation Platform | `https://<AAPのURL>` |
| OAuth Token | [応用演習4](./ex4.md) で控えた API トークン |
| Verify SSL | チェックを外す |

URL には、ブラウザで開いている AAP のアドレスを入れます。末尾に `/api/controller/` は付けません。Username と Password は空欄にします。Verify SSL を外すのは、自己証明書などで、演習環境の証明書を実行環境が信頼していない場合に接続できるようにするためです。

<a id="caution-verify-ssl"></a>

> [!CAUTION]
> **研修だけの設定です。** Verify SSL を外すと、AAP の証明書を検証せずに API トークンを送ります。[削除手順](../cleanup.md) の認証情報も同じです。実際の運用では、AAP に信頼できる CA の証明書を使い、Verify SSL にはチェックを入れたままにします。

## Section 3: Playbookとジョブテンプレート

Web IDE で `launch_note.yml` を開き、次の内容にします。ファイルが無い場合は作ります。トークンはファイルに書かず、ジョブテンプレートに付けた認証情報 `<name_prefix>-aap-api` から渡します。

```yaml
---
- name: Launch write-note-linux from a playbook
  hosts: localhost
  gather_facts: false
  environment:
    CONTROLLER_OPTIONAL_API_URLPATTERN_PREFIX: /api/controller/
    CONTROLLER_REQUEST_TIMEOUT: "30"

  tasks:
    - name: Launch write-note-linux
      ansible.controller.job_launch:
        name: "{{ name_prefix }}-write-note-linux"
        extra_vars:
          training_note: from controller collection
      register: launch_result

    - name: Wait for the job
      ansible.controller.job_wait:
        job_id: "{{ launch_result.id }}"

    - name: Show the launched job id
      ansible.builtin.debug:
        msg: "job {{ launch_result.id }} finished"
```

ゲートウェイ構成では API が `/api/controller/v2/` にあるため、`CONTROLLER_OPTIONAL_API_URLPATTERN_PREFIX` でそのパスを指定しています。このパスは認証情報の URL には書きません。認証情報のタイムアウト欄は空欄のままにします。欄が空欄だと空の文字列が渡り、`ansible.controller` のモジュールが秒数として読めずに失敗するため、秒数は Playbook の `CONTROLLER_REQUEST_TIMEOUT` で指定します。`{{ name_prefix }}` は、このファイルでは書き換えません。

**Commit to main** でコミットし、プロジェクト `<name_prefix>-aap-training-content` を同期します。

<details>
<summary>git CLIで行う場合</summary>

```bash
git add launch_note.yml
git commit -m "Launch write-note-linux from a playbook"
git push
```

</details>

**ジョブテンプレートの作成** で次を作ります。表に無い欄は空欄のままにします。

| 項目 | 入力値 |
|---|---|
| 名前 | `<name_prefix>-launch-note` |
| インベントリー | `<name_prefix>-aap-runner` |
| プロジェクト | `<name_prefix>-aap-training-content` |
| Playbook | `launch_note.yml` |
| 実行環境 | 空欄 |
| 認証情報 | `<name_prefix>-aap-api` |
| Extra variables | 下の1行 |

```yaml
name_prefix: sample01
```

`sample01` は演習1で決めた接頭辞に置き換えます。起動画面には Extra variables が表示されないため、値は編集画面で入れて保存してから起動します。

認証情報は `<name_prefix>-aap-api` だけにします。`localhost` には SSH でログインしないので、Machine 認証情報は付けません。

**テンプレートの起動** で実行します。出力の最後に `job <番号> finished` と表示されれば、起動したジョブの終了を待つところまで終わっています。ジョブ一覧には、`<name_prefix>-launch-note` のジョブと、そこから起動された `<name_prefix>-write-note-linux` のジョブの両方が追加されます。起動されたジョブの `Show the note written to note.txt` に `note.txt の note は from controller collection です` と出れば完了です。

終わったら、`<name_prefix>-write-note-linux` を起動してメモを `managed by AAP` に戻します。**Prompts** の **制限** は空欄のまま **Next** を押し、**Survey** は `managed by AAP` のまま **Finish** を押します。

## 次

[応用演習6](./ex6.md)
