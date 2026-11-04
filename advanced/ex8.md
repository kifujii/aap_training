# 応用演習 8 - CLI/Automation連携

この演習では、[応用演習7](./ex7.md)で扱ったREST APIを、Playbookから `ansible.controller` collectionを使って操作する方法と、EEを使ってPlaybookをローカルから実行する `ansible-navigator` の活用方法を学びます。

**前提条件:** [応用演習7](./ex7.md)を完了していること

> **注意:** この演習はAAPの画面操作ではなく、CLIが使えるローカル環境（[応用演習6](./ex6.md)で使ったEEビルド用サーバ等）での作業が中心です。

[前に戻る](./ex7.md) | [次へ進む](./ex9.md)

---

## Section 1: ansible.controller collectionの導入

### Step 1: Collectionをインストールする

```bash
$ ansible-galaxy collection install ansible.controller
```

このCollectionには、Organization/Project/Inventory/Job Template/Credentialなど、AAPの主要オブジェクトをAnsible Playbookから操作するためのモジュール群が含まれています。

### Step 2: 接続情報を変数ファイルにまとめる

```bash
$ cat << 'EOF' > controller_auth.yml
controller_host: "https://<AAPのURL>"
controller_oauthtoken: "{{ lookup('env', 'AAP_TOKEN') }}"
controller_validate_certs: false
EOF
```

**Note:** `controller_validate_certs: false` は自己署名証明書等を使う検証環境向けの設定です。本番運用では証明書を正しく検証できる構成にしてください。

## Section 2: controller_job_launchモジュールでJob Templateを起動する

### Step 1: Playbookを作成する

```bash
$ cat << 'EOF' > launch_via_collection.yml
---
- name: AAPのJob Templateをansible.controller collectionから起動する
  hosts: localhost
  connection: local
  gather_facts: false
  vars_files:
    - controller_auth.yml

  tasks:
    - name: apply-baselineを起動する
      ansible.controller.job_launch:
        name: apply-baseline
        controller_host: "{{ controller_host }}"
        controller_oauthtoken: "{{ controller_oauthtoken }}"
        validate_certs: "{{ controller_validate_certs }}"
        extra_vars:
          host_timezone: UTC
      register: launch_result

    - name: 起動結果を表示
      debug:
        var: launch_result
EOF
```

### Step 2: 実行する

```bash
$ ansible-playbook launch_via_collection.yml
```

`launch_result` に、起動したJobのID・ステータス等が格納されます。

## Section 3: Jobの完了を待つ

`ansible.controller` collectionには、Jobの完了を待機するモジュールも用意されています。

```yaml
    - name: Jobの完了を待機する
      ansible.controller.job_wait:
        job_id: "{{ launch_result.id }}"
        controller_host: "{{ controller_host }}"
        controller_oauthtoken: "{{ controller_oauthtoken }}"
        validate_certs: "{{ controller_validate_certs }}"
      register: wait_result

    - name: 最終ステータスを表示
      debug:
        msg: "Job {{ launch_result.id }} finished with status: {{ wait_result.status }}"
```

**Note:** CI/CDパイプライン（[応用演習11](./ex11.md)）からAAPのJobをトリガーし、その完了を待って後続処理（通知、次のステージへの昇格判断など）を行う、といった連携がこの仕組みで実現できます。

## Section 4: ansible-navigatorの活用

`ansible-navigator` は、EEを使ってPlaybookをローカルCLIから実行・デバッグするためのツールです。AAPを経由せずに動作確認したい場合や、EEの内容そのものを調査したい場合に役立ちます（[応用演習6](./ex6.md)でも利用しました）。

### Step 1: 対話モードで実行する

```bash
$ cd aap-training-content
$ ansible-navigator run site.yml -i inventory.yml --eei localhost/custom-ee:1.0
```

対話UIが起動し、Play一覧やTask一覧を矢印キーで辿りながら、各タスクの詳細結果を確認できます。

### Step 2: ドキュメント参照モードを使う

```bash
$ ansible-navigator doc ansible.builtin.template --eei localhost/custom-ee:1.0
```

EEイメージ内にインストールされているモジュールの `ansible-doc` 相当の情報を、ローカルのAnsible環境を汚さずに確認できます。

### Step 3: inventory確認モードを使う

```bash
$ ansible-navigator inventory -i inventory.yml --eei localhost/custom-ee:1.0
```

EE経由でインベントリの構造（グループ・ホスト）を確認できます。

---

ここまでで、Playbook/CLIからAAPを操作する方法と、EEをローカルで活用する方法を習得しました。次の演習では、イベント駆動型自動化（EDA）の基本を扱います。

[次へ進む](./ex9.md)
