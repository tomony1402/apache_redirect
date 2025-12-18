
👉 **重要**  
```（バッククォート3つ）  
は  
**必ず「開始」と「終了」をセットで書く**

---

## ここからが本題  
## 👇「あなたが伝えたいこと」を全部入れた完成形 README

**これをそのまま README.md に貼っていい**  
（書き方の見本としても使える）

```md
# Apache Redirect on EC2 (Terraform)

このリポジトリは、Terraform を使って  
Apache によるリダイレクト設定を行った EC2 を構築するためのものです。

HCP Terraform は使用せず、ローカル実行前提で運用します。

---

## この Terraform でやっていること

- AWS 上に EC2 を 1 台作成する
- Apache をインストールし、リダイレクト設定を行う
- EC2 作成処理は `modules/redirect_ec2` に切り出している
- SSH 接続用のキーペアを Terraform で生成する

---

## main.tf について

`main.tf` は、この Terraform 構成全体のエントリーポイントです。

このファイルでは、使用する AWS Provider の設定を行い、  
Terraform で生成した SSH 用キーペアを利用できるようにした上で、  
EC2 の作成処理を `modules/redirect_ec2` モジュールに委譲しています。

個々のリソースの詳細設定は持たず、  
「どのクラウドを使うか」「どの認証情報を使うか」「どの module を実行するか」  
といった全体の流れと役割の定義のみを担っています。

---

## ディレクトリ構成

```text
.
├── main.tf
├── output.tf
├── .terraform.lock.hcl
├── modules
│   └── redirect_ec2
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│       └── userdata
│           └── apache_redirect.sh.tmpl

