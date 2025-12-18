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

## main.tf でやっていること

この `main.tf` では、以下のリソースを定義しています。

### 1. AWS Provider の設定
- AWS Provider（hashicorp/aws）を使用
- リージョン：`us-east-1`
- AWS CLI の profile（`aws180`）を利用

### 2. SSH キーペアの作成
- `tls_private_key` を使って RSA 4096bit の秘密鍵を生成
- AWS に Key Pair（`tf-key`）を登録
- ローカルに `tf-key.pem` を出力（権限 600）

※ 秘密鍵ファイルは Git 管理しません。

### 3. EC2 作成用 Module の呼び出し
- `modules/redirect_ec2` モジュールを使用
- 作成した Key Pair 名を module に渡す
- Apache のリダイレクト設定は module 側で実施

