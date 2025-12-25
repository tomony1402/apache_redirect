Terraform configuration for deploying ultra low-cost Apache-based redirect EC2 instances
using AlmaLinux, supporting all AWS regions.

# Apache Redirect on EC2 (Terraform)

このリポジトリは、Terraform を使って  
Apache によるリダイレクト設定を行った EC2 を構築するためのものです。  
各 EC2 には IAM ロールを付与し、AWS Systems Manager (SSM) での管理を前提としています。

HCP Terraform は使用せず、ローカル実行前提で運用します。

---

## この Terraform でやっていること

- AWS 上に EC2 を 18台作成する
- Apache をインストールし、リダイレクト設定を行う
- IAM インスタンスプロフィールを付与し、SSM 管理を有効化する
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

<details>
<summary>🔍 Terraform での AMI 取得コードを表示</summary>

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
```
</details>

---

## EC2定義

<details>
<summary>🛠️ EC2 作成の Terraform コードを表示</summary>

```hcl
data "aws_region" "current" {}

resource "aws_instance" "web" {
  for_each = local.redirect_domains

  # ★ AMI を AlmaLinux に変更
  ami           = data.aws_ami.almalinux.id
  instance_type = "t2.nano"
  key_name      = var.key_name

  iam_instance_profile = "ec2-ssm-kondo"
  associate_public_ip_address = true

  vpc_security_group_ids = [
    aws_security_group.redirect.id
  ]

  # ★ 20GB にできる
  root_block_device {
    volume_size = 10
    volume_type = "gp3"
  }

   user_data = templatefile(
    "${path.module}/userdata/apache_redirect.sh.tmpl",
    {
      # each.value（ドメイン名）ではなく、each.key（kensho1, kensho2など）を渡す
      redirect_domain = each.key
      region          = data.aws_region.current.name
    }
  )

  tags = {
    Name = each.key
  }
}
```

</details>

### 全リージョン対応確認（AWS CLI）

<details> 
<summary>💻 全リージョンの AMI 存在確認コマンド（AWS CLI）</summary>

```bash
for r in $(aws ec2 describe-regions --query "Regions[].RegionName" --output text); do
  count=$(aws ec2 describe-images \
    --region "$r" \
    --owners 764336703387 \
    --filters "Name=name,Values=AlmaLinux OS 9*" \
              "Name=architecture,Values=x86_64" \
    --query "length(Images)" \
    --output text)
  printf "%-20s : %s\n" "$r" "$count"
done

```
</details>

## AlmaLinux サポート期間

本構成では AlmaLinux 9 を使用している。

- AlmaLinux 9
  - フルサポート：～ 2027 年
  - セキュリティ更新：～ 2032 年

RHEL と同等のライフサイクルを持ち、
長期運用を前提とした構成に適している。

## 料金目安（無料枠外・東京）

以下は **ap-northeast-1（東京リージョン）**  
**オンデマンド / 月720時間稼働**を想定した概算。

### EC2 インスタンス

| インスタンスタイプ | 月額目安 |
|------------------|----------|
| t2.nano | 約 600〜700円 |
| t3.micro | 約 1,400円 |

---

### EBS（gp3）

| サイズ | 月額目安 |
|------|----------|
| 10GB | 約 120円 |
| 20GB | 約 240円 |
| 30GB | 約 360円 |

---

### 構成別 合計

| 構成 | 月額目安 |
|---|---|
| t2.nano + 10GB | 約 **720〜820円** |
| t2.nano + 20GB | 約 **850〜950円** |
| t3.micro + 20GB | 約 **1,640円** |

※ 為替レート（約150円/USD）により多少前後する  
※ OS（Amazon Linux / AlmaLinux）による料金差はなし

---

## 🔐 IAM 権限の設定

本プロジェクトでは、EC2 が SSM Parameter Store から安全に設定を取得するため、以下の 2 つのポリシーを定義しています。  

### 1. EC2 用ロール（ec2-ssm-kondo）の作成
本プロジェクトでは、EC2 インスタンスが SSM Parameter Store から安全に設定情報を取得するために、専用の IAM ロール **`ec2-ssm-kondo`** を作成しています。  

このロールをインスタンスに付与することで、インスタンス自身が AWS サービス（SSM）に対して、許可された範囲内でのデータ取得リクエストを行えるようになります。  

### 2. 作業ユーザーへの譲渡権限付与（iam:PassRole）
Terraform 等で EC2 を作成し、そこに上記ロールを紐付ける（パスする）ための権限です。実行ユーザー側にこの許可がないと、作成した `ec2-ssm-kondo` ロールをインスタンスへ付与することができないため、セキュリティ上の重要な設定となります。

<details>
<summary>JSONポリシーの中身を確認する</summary>

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": "iam:PassRole",
			"Resource": "arn:aws:iam::276164042029:role/ec2-ssm-kondo"
		}
	]
}
```
</details>

### 3. ロールへの許可ポリシー設定（ssm:GetParameter）
作成した `ec2-ssm-kondo` ロールに対し、以下のポリシーをアタッチします。これにより、ロールを付与された EC2 は、SSM 内の `/redirect/` パス以下にあるデータのみをピンポイントで取得できるようになります（最小権限の原則）。

<details>
<summary>JSONポリシーの中身を確認する</summary>

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": [
				"ssm:GetParameter",
				"ssm:GetParameters"
			],
			"Resource": "arn:aws:ssm:ap-northeast-1:276164042029:parameter/redirect/*"
		}
	]
}
```

</details>

<details> 
<summary>📄 セットアップスクリプト（apache_redirect.sh.tmpl）を表示</summary>
```json
#!/bin/bash
set -eux

# ---------------------------------------------------------
# 1. 依存パッケージの解決
# ---------------------------------------------------------
yum update -y
yum install -y awscli httpd

# ---------------------------------------------------------
# 2. 設定の動的取得（SSM Parameter Store 連携）
# ---------------------------------------------------------
INSTANCE_NAME="${redirect_domain}"
REGION="${region}"

# SSM からリダイレクト先 URL を取得（失敗時はドメイン名をフォールバックとして使用）
SSM_VALUE=$(aws ssm get-parameter --name "/redirect/$INSTANCE_NAME/url" --query "Parameter.Value" --output text --region $REGION || echo "")

TARGET_URL=${SSM_VALUE:-"${redirect_domain}"}

# ---------------------------------------------------------
# 3. Apache 設定の自動生成（80/8080ポート対応）
# ---------------------------------------------------------
if ! grep -q "^Listen 8080" /etc/httpd/conf/httpd.conf; then
  echo "Listen 8080" >> /etc/httpd/conf/httpd.conf
fi

cat > /etc/httpd/conf.d/redirect.conf << EOL
<VirtualHost *:80>
    Redirect permanent / http://$TARGET_URL/
</VirtualHost>

<VirtualHost *:8080>
    Redirect permanent / http://$TARGET_URL/
</VirtualHost>
EOL

systemctl enable httpd
systemctl restart httpd

# ---------------------------------------------------------
# 4. 永続化設定（再起動時の自動同期）
# ---------------------------------------------------------
cp "$0" /var/lib/cloud/scripts/per-boot/redirect_sync.sh
chmod +x /var/lib/cloud/scripts/per-boot/redirect_sync.sh
```

</details>

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



