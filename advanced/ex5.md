# 応用演習 5 - PlaybookからJobを起動する

[応用演習4](./ex4.md) | [応用演習6へ](./ex6.md)

応用演習4の API 呼び出しを、Playbook のジョブとして実行します。既定の実行環境には `ansible.controller` が入っているので、その中から `<name_prefix>-write-note-linux` を起動します。実行環境の欄は空欄のまま、システムの既定を使います。

## Section 1: 起動用のインベントリー

Playbook は `localhost` で動き、AAP の API を呼びます。接続先の変数は `ansible_connection: local` です。

**インベントリーの作成** で `<name_prefix>-aap-runner` を作ります。保存したあと、**ホスト** タブの **ホストの作成** で次を追加します。

| 項目 | 入力値 |
|---|---|
| 名前 | `localhost` |
| 変数 | 下の2行 |

```yaml
ansible_connection: local
```

ホストの欄名は **変数** です。ジョブテンプレートの Extra variables とは別の欄です。

## Section 2: AAPの認証情報

**認証情報の作成** を開きます。タイプは `Red Hat Ansible Automation Platform` です。

| 項目 | 入力値 |
|---|---|
| 名前 | `<name_prefix>-aap-api` |
| Red Hat Ansible Automation Platform | `https://<AAPのURL>` |
| OAuth Token | [応用演習4](./ex4.md) で控えた API トークン |
| Verify SSL | チェックを外す |

URL は、ブラウザで開いている AAP のアドレスです。末尾に `/api/controller/` は付けません。Username と Password は空欄にします。Verify SSL を外すのは、自己証明書などで、演習環境の証明書を実行環境が信頼していない場合に接続できるようにするためです。

## Section 3: Playbookとジョブテンプレート

Web IDE で `launch_note.yml` を開き、次の内容にします。ファイルが無い場合は作ります。トークンはファイルに書きません。ジョブに付けた `<name_prefix>-aap-api` が渡します。

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

`CONTROLLER_OPTIONAL_API_URLPATTERN_PREFIX` は、ゲートウェイ構成の API が `/api/controller/v2/` にあるための指定です。認証情報の URL には、このパスを書きません。認証情報のタイムアウト欄は空欄のままにします。空欄の値は文字列として渡され、モジュールが秒数に変換できず失敗するため、`CONTROLLER_REQUEST_TIMEOUT` で秒数を指定します。`{{ name_prefix }}` は、このファイルでは書き換えません。

**Commit to main** でコミットし、プロジェクト `<name_prefix>-aap-training-content` を同期します。

<details>
<summary>git CLIで行う場合</summary>

```bash
git add launch_note.yml
git commit -m "Launch write-note-linux from a playbook"
git push
```

</details>

**ジョブテンプレートの作成** で次を作ります。表に無い欄は空欄です。

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
name_prefix: tanaka01
```

`tanaka01` は演習1で決めた接頭辞にします。起動画面には Extra variables は出ないので、編集画面で保存してから起動します。

認証情報は `<name_prefix>-aap-api` だけにします。`localhost` はローカル接続です。

**テンプレートの起動** で実行します。出力の最後に `job <番号> finished` と出れば、待ちまで終わっています。ジョブ一覧には、この `<name_prefix>-launch-note` と、その中から起動された `<name_prefix>-write-note-linux` の両方が増えます。起動されたジョブの `Show the note written to note.txt` に `note.txt の note は from controller collection です` と出れば完了です。

終わったら、`<name_prefix>-write-note-linux` を起動してメモを `managed by AAP` に戻します。**Prompts** の **制限** は空欄のまま **Next** を押し、**Survey** は `managed by AAP` のまま **Finish** を押します。

## 次

[応用演習6](./ex6.md)
