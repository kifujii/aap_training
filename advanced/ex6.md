# 応用演習 6 - PlaybookからJobを起動する

[応用演習5](./ex5.md) | [応用演習7へ](./ex7.md)

応用演習5の API 呼び出しを、Playbook のジョブとして実行します。既定の実行環境には `ansible.controller` が入っているので、その中から `write-note-linux` を起動します。実行環境の欄は空欄のまま、システムの既定を使います。

## Section 1: 起動用のインベントリー

Playbook は `localhost` で動き、AAP の API を呼びます。接続先の変数は `ansible_connection: local` です。

**インベントリーの作成** で `aap-runner` を作ります。保存したあと、**ホスト** タブの **ホストの作成** で次を追加します。

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
| 名前 | `aap-api` |
| Red Hat Ansible Automation Platform | `https://<AAPのURL>` |
| OAuth Token | [応用演習5](./ex5.md) で控えた API トークン |
| Verify SSL | チェックを外す |

URL は、ブラウザで開いている AAP のアドレスです。末尾に `/api/controller/` は付けません。Username と Password は空欄にします。Verify SSL を外すのは、演習環境の証明書を実行環境が信頼していない場合に接続できるようにするためです。

## Section 3: Playbookとジョブテンプレート

Web IDE で `launch_note.yml` を開き、次の内容にします。ファイルが無い場合は作ります。トークンはファイルに書きません。ジョブに付けた `aap-api` が渡します。

```yaml
---
- name: Launch write-note-linux from a playbook
  hosts: localhost
  gather_facts: false
  environment:
    CONTROLLER_OPTIONAL_API_URLPATTERN_PREFIX: /api/controller/

  tasks:
    - name: Launch write-note-linux
      ansible.controller.job_launch:
        name: write-note-linux
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

`CONTROLLER_OPTIONAL_API_URLPATTERN_PREFIX` は、ゲートウェイ構成の API が `/api/controller/v2/` にあるための指定です。認証情報の URL には、このパスを重ねて書きません。

**Commit to main** でコミットし、プロジェクト `aap-training-content` を同期します。

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
| 名前 | `launch-note` |
| インベントリー | `aap-runner` |
| プロジェクト | `aap-training-content` |
| Playbook | `launch_note.yml` |
| 実行環境 | 空欄 |
| 認証情報 | `aap-api` |

認証情報は `aap-api` だけにします。`localhost` はローカル接続です。

**テンプレートの起動** で実行します。出力の最後に `job <番号> finished` と出れば、待ちまで終わっています。ジョブ一覧には、この `launch-note` と、その中から起動された `write-note-linux` の両方が増えます。`lin1` の `note:` が `from controller collection` になっていれば完了です。

終わったら、`write-note-linux` を Survey から起動し、メモを `managed by AAP` に戻します。

## 次

[応用演習7](./ex7.md)
