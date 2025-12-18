# Apache Redirect on EC2 (Terraform)

このリポジトリは、Terraform を使って  
**Apache によるリダイレクト設定を行った EC2** を構築するためのものです。

HCP Terraform は使用せず、**ローカル実行前提**で運用します。

---

## 構成

- Terraform
- AWS EC2
- Apache
- Terraform module（modules/redirect_ec2）

---

## ディレクトリ構成

```text
.
├── main.tf
├── output.tf
├── .terraform.lock.hcl
├── modules/
│   └── redirect_ec2/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│       └── userdata/
│           └── apache_redirect.sh.tmpl

---

## main.tf について

この `main.tf` は、この Terraform 構成のエントリーポイントです。

ここでは以下のことを行っています。

- AWS Provider の設定（リージョン指定、AWS CLI の profile 利用）
- SSH 接続用のキーペアを Terraform で生成
- EC2 作成処理を `modules/redirect_ec2` モジュールに委譲

EC2 の詳細設定や Apache のリダイレクト設定は  
`modules/redirect_ec2` 配下で定義しています。

