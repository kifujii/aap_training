# 操作対象サーバ（AWS）

この研修の操作対象は、Webサイトを公開するサーバではありません。AWS上のLinuxサーバに対して、時刻同期、基本パッケージ、ログインバナー、SSHの基本設定、状態点検を行います。

サーバの用意には2つの経路があります。どちらか一方だけ実施してください。

| 経路 | いつ使うか | やること |
|---|---|---|
| **A. 新規構築** | 操作対象がまだない | `provision_ec2.yml` でEC2を作成し、払い出されたプライベートIPをインベントリに書く |
| **B. 既存サーバ** | EC2やその他のLinuxがすでにある | 構築Playbookは実行しない。インベントリの `ansible_host` に、そのサーバのIPまたはDNS名を書く |

以降の基礎演習（インベントリ、Project、Job Template）は、どちらの経路でも同じです。違うのはインベントリに載せるアドレスの出どころだけです。

## 共通の前提

* OSはRHEL系（RHELまたは同等）を想定しています。`chronyd` と `dnf` を使います。
* SSHできる管理ユーザー（例: `ec2-user` または `ansible`）があり、`sudo` できること
* AAPのExecution Nodeから、そのユーザーでTCP/22へ到達できること
* 認証はSSH鍵を基本にします。[基礎演習2](./basic-intro/ex2.md)のMachine Credentialに、その秘密鍵を登録します

## 経路A: EC2を新規作成する

### 控えておく値

| 項目 | 例 | 用途 |
|---|---|---|
| リージョン | `ap-northeast-1` | モジュールの `region` |
| VPC ID | `vpc-xxxxxxxx` | セキュリティグループ |
| Subnet ID | `subnet-xxxxxxxx` | インスタンスの配置先。AAPからSSHできるサブネットにする |
| AMI ID | `ami-xxxxxxxx` | 使用するRHEL系イメージ。コンソールのAMIカタログで確認し、資料の例をそのまま使わない |
| キーペア名 | `aap-training` | インスタンスに渡す公開鍵の名前。対になる秘密鍵をMachine Credentialに登録する |
| SSHを許可するCIDR | AAPのExecution Nodeが属するCIDR | セキュリティグループのinbound |

IAM権限は、セキュリティグループの作成・参照、EC2のRunInstances、DescribeInstances、CreateTagsが必要です。アクセスキーを使う場合は、そのキーにこれらの権限を付けます。

### AWS用Credential

[基礎演習2](./basic-intro/ex2.md)の手順で、認証情報タイプ **Amazon Web Services** のCredentialを作ります。名前の例は `aws-training` です。アクセスキーとシークレットアクセスキーを入れます。

Playbook側の `amazon.aws` モジュールは、JobにこのCredentialが付いていると、実行時にAWSの環境変数として受け取ります。アクセスキーをPlaybookやGitLabに書かないでください。

### Execution Environment

`provision_ec2.yml` は `amazon.aws` collectionを使います。AAP 2.7 の既定Execution Environment（`ee-supported-rhel9`）には `amazon.aws` 10.3.0 が入っているので、実行環境は既定のままで構築Jobを実行できます。

[sample-content/requirements.yml](./sample-content/requirements.yml) はこの 10.3.0 に固定しています。Projectを同期すると、このファイルに書いたcollectionがEE内の同じ名前のcollectionより先に使われます。下限だけを書いて新しい版を許すと、ansible-core 2.16 ではサポートされない版が入り、Jobの先頭に `does not support Ansible version 2.16.19` と出ます。

既定以外のEEを使うときは、そのEEの ansible-core に合う版かを確認してください。自分でEEを作る手順は [応用演習6](./advanced/ex6.md) です。

### 変数

`provision_ec2.yml` は、次の変数がプレースホルダのままだと点検タスクで停止します。Job TemplateのExtra variablesか、GitLab上のグループ変数で実値に置き換えてください。

```yaml
ec2_region: ap-northeast-1
ec2_vpc_id: vpc-xxxxxxxx
ec2_subnet_id: subnet-xxxxxxxx
ec2_ami_id: ami-xxxxxxxx
ec2_key_name: aap-training
ec2_ssh_ingress_cidr: 10.0.0.0/16
ec2_instance_type: t3.micro
ec2_assign_public_ip: false
ec2_instance_names:
  - srv1
  - srv2
```

`ec2_assign_public_ip` は、プライベートサブネットでAAPから届くなら `false` のままにします。パブリックIPが必要なネットワークだけ `true` にします。

### Job Template

| 項目 | 値 |
|---|---|
| 名前 | `provision-ec2` |
| インベントリー | `localhost` を含むもの。Playの対象は `localhost`、接続は `local` |
| Playbook | `provision_ec2.yml` |
| 認証情報 | `aws-training`（Amazon Web Services） |
| 実行環境 | 空欄（AAP 2.7 の既定EE） |

インベントリの例は [sample-content/inventory.yml](./sample-content/inventory.yml) の `automation` グループです。Machine CredentialはlocalhostへのSSHには使いません。

実行に成功すると、各インスタンスのインスタンスIDとプライベートIPがログに出ます。そのIPを、経路Bと同じように `managed` グループの `ansible_host` へ書き、GitLabへpushしてInventory Sourceを同期します。

同じPlaybookを再実行しても、`name` が同じインスタンスは作り直されません。台数を増やすときは `ec2_instance_names` に名前を足します。

## 経路B: 既存サーバをインベントリに指定する

`provision_ec2.yml` は実行しません。AWS Credentialも、この経路だけでは不要です。

[sample-content/inventory.yml](./sample-content/inventory.yml) の `managed` グループを、実際のサーバに書き換えます。

```yaml
managed:
  hosts:
    srv1:
      ansible_host: 10.0.1.11
    srv2:
      ansible_host: 10.0.1.12
```

* ホスト名（`srv1`）はインベントリ上の名前です。OSのhostnameと一致していなくても構いません
* `ansible_host` はAAPからSSHするときのIPまたはDNS名です
* 台数が1台でも、以降の演習は実施できます。制限（Limit）の例は、あるホスト名に読み替えてください
* EC2以外（オンプレミスや別クラウド）でも、SSHとsudoができればこの経路で進めます

書き換えたファイルをGitLabへpushし、[基礎演習5](./basic-controller/ex5.md)のInventory Sourceを同期すると、AAPのインベントリーにそのホストが並びます。

## 構築後に共通で行うこと

1. Machine Credentialのユーザー名と秘密鍵が、対象サーバのログインユーザーと一致している
2. インベントリーの **ホスト** で **Run command**、モジュール `ping` が成功する（[基礎演習3](./basic-intro/ex3.md)）
3. Job Template `apply-baseline` で [sample-content/site.yml](./sample-content/site.yml) を実行する（[基礎演習6](./basic-controller/ex6.md)）

`site.yml` はサーバを新規作成しません。登録済みホストの設定をそろえるPlaybookです。

## 任意: EC2をインベントリソースにする

経路Aでインスタンスに `Project=aap-training` タグが付いたあと、IPを `inventory.yml` に転記する代わりに、AAPのインベントリソース **Amazon EC2** で取り込めます。YAMLの `training-inventory` とは **別のインベントリー**（例: `ec2-managed`）にしてください。同じインベントリーで `Overwrite` 付きのソースを二つ有効にすると、同期のたびに相手のホストを消します。

`provision-ec2` 用の `localhost` は、このEC2ソースには出てきません。構築Jobは `training-inventory` のままにします。`apply-baseline` のインベントリーだけ `ec2-managed` に切り替えます。

ソースの認証情報は `aws-training` です。実行環境は `amazon.aws` を含むものにします。Source variables の例は次のとおりです。

```yaml
---
plugin: amazon.aws.aws_ec2
regions:
  - ap-northeast-1
filters:
  "tag:Project":
    - aap-training
hostnames:
  - tag:Name
compose:
  ansible_host: private_ip_address
keyed_groups:
  - key: tags.Role
    prefix: ""
    separator: ""
```

`regions` は `ec2_region` に合わせます。同期後、ホスト名が `srv1` / `srv2` になり、グループ `managed` に入っていれば、`site.yml` の `hosts: managed` がそのまま使えます。グループ名が違う場合は、`prefix` を調整して再同期します。EC2以外の既存サーバは、このソースでは列挙されません。その場合は経路BのYAMLを使います。
