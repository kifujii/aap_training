# AWSでサーバを作る場合

手順は [演習4](./basic-intro/ex4.md) です。対象の Linux と Windows がすでにある場合、このページの作業は不要です。

演習4が作るものは次のとおりです。名前の先頭には、受講者ごとに違う `resource_prefix` が付きます。

| リソース | 名前の例（prefix が `tanaka01` のとき） |
|---|---|
| VPC | `tanaka01-vpc` |
| サブネット | `tanaka01-subnet` |
| インターネットゲートウェイ | `tanaka01-igw` |
| ルートテーブル | `tanaka01-rt` |
| セキュリティグループ | `tanaka01-aap`（TCP 22 と TCP 5985） |
| Linux | `tanaka01-linux`（ユーザー `ansible`） |
| Windows | `tanaka01-windows`（ユーザー `Administrator`、PSRP + NTLM） |

Playbook は [sample-content/create_aws_servers.yml](./sample-content/create_aws_servers.yml) です。削除は [sample-content/remove_aws_servers.yml](./sample-content/remove_aws_servers.yml) です。

`amazon.aws` は 10.3.0 に固定します。AAP 2.7 の既定実行環境は ansible-core 2.16 で、これより新しい版はサポート外です。実行環境の欄は空欄（既定）のままにします。
