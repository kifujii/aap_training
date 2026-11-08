# 応用演習 6 - PlaybookからJobを起動する

[応用演習5](./ex5.md) | [応用演習7へ](./ex7.md)

応用演習5の API 呼び出しを、Playbook から行います。作業場所は AAP サーバです。

`ansible-galaxy collection install` はインターネットへ出ます。出られない場合は、この演習は読み物にして [応用演習7](./ex7.md) へ進んでください。

## Section 1: コレクション

AAP サーバに SSH し、次を実行します。

```bash
ansible-galaxy collection install ansible.controller
```

同じディレクトリに `controller_auth.yml` を作ります。トークンはファイルに書かず、環境変数のままにします。

```yaml
---
controller_host: "https://<AAPのURL>"
controller_oauthtoken: "{{ lookup('env', 'AAP_TOKEN') }}"
controller_validate_certs: false
```

`controller_validate_certs: false` は、演習用の証明書向けです。

## Section 2: 起動するPlaybook

`launch_note.yml`:

```yaml
---
- name: Launch write-note-linux from a playbook
  hosts: localhost
  connection: local
  gather_facts: false
  vars_files:
    - controller_auth.yml

  tasks:
    - name: Launch write-note-linux
      ansible.controller.job_launch:
        name: write-note-linux
        controller_host: "{{ controller_host }}"
        controller_oauthtoken: "{{ controller_oauthtoken }}"
        validate_certs: "{{ controller_validate_certs }}"
        extra_vars:
          training_note: from controller collection
      register: launch_result

    - name: Wait for the job
      ansible.controller.job_wait:
        job_id: "{{ launch_result.id }}"
        controller_host: "{{ controller_host }}"
        controller_oauthtoken: "{{ controller_oauthtoken }}"
        validate_certs: "{{ controller_validate_certs }}"

    - name: Show the launched job id
      ansible.builtin.debug:
        msg: "job {{ launch_result.id }} finished"
```

```bash
export AAP_TOKEN="<応用演習5で控えたトークン>"
ansible-playbook launch_note.yml
```

AAP のジョブ一覧に `write-note-linux` が増え、`note:` が `from controller collection` になれば完了です。終わったら Survey で `managed by AAP` に戻します。

この2つのファイルは演習用の作業ディレクトリに置きます。GitLab の `aap-training-content` には入れません。トークンが混ざるのを防ぐためです。

## 次

[応用演習7](./ex7.md)
