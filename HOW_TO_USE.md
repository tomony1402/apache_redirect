# Terraform リダイレクト EC2 作成手順

Terraform を使用して、Apache リダイレクト用 EC2 を作成・管理するための手順書です。

⚠️ **重要**  
この作業は **必ず一人で実施してください（同時作業禁止）**。

---

## 作業サーバー

- 2.56.0.205

---

## ① AWS リージョンの設定

### 編集するファイル
`/root/appache/terraform2/main.tf`

### 設定内容

```hcl
provider "aws" {
  # region = "ap-northeast-1"
  region = "us-east-1"
  # region = "ap-northeast-2"
  # region = "ap-northeast-2"

  profile = "aws180"
}

### 注意点

- 使用するリージョンのコメントアウトを外す
- `profile = "aws180"` は削除しないこと
- この作業は必ず一人で実施すること

---

## ② 作成する EC2（号機）の指定
