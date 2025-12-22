# Apache Redirect on EC2 (Terraform)

このリポジトリは、Terraform を使って  
Apache によるリダイレクト設定を行った EC2 を構築するためのものです。

HCP Terraform は使用せず、ローカル実行前提で運用します。

---

## この Terraform でやっていること

- AWS 上に EC2 を 18台作成する
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
全体の流れと役割の定義のみを担っています。

---

##改善点 ami変更
-リージョン別による検証結果
検証は末尾2.56.0.198のAWS180で検証
almalinuxで検証したがt2/t3.nanoは使えるリージョンが限られている
未検証だが、t4g.nanoでも限られている可能性もあり、
t3.microは問題なく使えるらしい

⇒未検証だが「リージョン名+末尾のアルファベット」によっては使える使えないがあるので
それを変更すれば出来るかも？
以下のコマンドで在庫確認可能

*********
for r in \
us-east-1 us-east-2 us-west-1 us-west-2 \
ap-northeast-1 ap-northeast-2 ap-northeast-3 \
ap-southeast-1 ap-southeast-2 ap-south-1 \
ca-central-1 \
eu-central-1 eu-west-1 eu-west-2 eu-west-3 eu-north-1 \
sa-east-1
do
  echo "===== $r ====="
  aws ec2 describe-instance-type-offerings \
    --region $r \
    --location-type availability-zone \
    --filters Name=instance-type,Values=t3.nano \
    --query 'InstanceTypeOfferings[].Location' \
    --output text
done
*********



・次の手段として
⇒t3.nanoはリージョンアルファベット末尾abcで有れば出来る想定なので
サブネットで固定する必要がある
*********
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true

  availability_zone = "us-east-1a"  # ← ここ

  tags = { Name = "Public" }
}
*********


⇒上記で出来ない場合は次に移行する
リージョン別にAMIイメージを作成してterraformで指定して構築する


---

# Redirect EC2 (Low Cost / Multi-Region)

Terraform で **低コストなリダイレクト専用 EC2** を  
**複数リージョン対応**で作成する構成。

最小構成（t2.nano + 最小EBS）を前提とし、  
AMI は **リージョン依存を吸収する設計**にしている。

---

## 構成概要

- OS: **AlmaLinux 9**
- Instance Type: **t2.nano**
- Root Volume: **10GB or 20GB (gp3)**
- 用途: Apache による HTTP リダイレクト
- デプロイ方式: Terraform
- 対応リージョン: **全リージョン**

---

## 設計方針

### なぜ AlmaLinux？
- Amazon Linux 2023 は **ルートボリューム最小 30GB**
- 20GB / 10GB に縮小できない
- **AlmaLinux は最小 8〜10GB の AMI が存在**

→ **EBS サイズを下げてコスト削減するため AlmaLinux を採用**

---

## AMI（全リージョン対応）

AMI ID はリージョンごとに異なるため、  
**ID を固定せず検索条件で取得**する。

```hcl
data "aws_ami" "almalinux" {
  most_recent = true
  owners      = ["764336703387"] # AlmaLinux OS Foundation

  filter {
    name   = "name"
    values = ["AlmaLinux OS 9*"]
  }

  filter {
    name   = "architecture"
    values = ["x86_64"]
  }

  filter {
    name   = "root-device-type"
    values = ["ebs"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}


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

