# 応用演習 3 - Execution Environment

[応用演習2](./ex2.md) | [応用演習4へ](./ex4.md)

Job は、AAP サーバに直接入れた Ansible ではなく、Execution Environment というコンテナイメージの中で動きます。テンプレートごとにイメージを分けられます。

## Section 1: 既定の実行環境を見る

**Automation Execution > Infrastructure > 実行環境** を開きます。`Default execution environment` を開き、Image と Pull を見ます。AAP 2.7 の例は `registry.redhat.io/ansible-automation-platform-27/ee-supported-rhel9:latest` です。

このイメージには、この資料の Linux / Windows のファイル配置と、`amazon.aws` 10.3.0 が入っています。基礎演習のテンプレートは、実行環境を空欄にしてこの既定を使います。

## Section 2: テンプレートでの指定

`write-note-linux` の編集画面で、実行環境の欄が空欄であることを確認します。空欄のときは、プロジェクトの指定、それも空ならシステムの既定、の順で選ばれます。

ジョブの詳細にも、使った実行環境の名前が出ます。

## 次

イメージを追加したい場合だけ [応用演習4](./ex4.md) へ進みます。既定のままで足りる場合は、[応用演習5](./ex5.md) へ進んで構いません。
