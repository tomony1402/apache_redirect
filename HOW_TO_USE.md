# Terraform リダイレクト EC2 作成手順

Terraform を使用して、Apache リダイレクト用 EC2 を作成・管理するための手順書です。

⚠️ **重要**  
この作業は **必ず一人で実施してください（同時作業禁止）**。

---

## 作業サーバー

- 2.56.0.205
-対象ディレクトリ
-/root/appache/terrafrom
-/root/appache/terrafrom2
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

```

### 注意点

- 使用するリージョンのコメントアウトを外す
- `profile = "aws180"` は削除しないこと
- この作業は必ず一人で実施すること

---

## ② 作成する EC2（号機）の指定

### 編集するファイル
`/root/appache/terraform2/modules/redirect_ec2/main.tf`

### 設定内容

```hcl
locals {
  redirect_domains = {
    web-34 = "tune-snowboarding.com"
    web-38 = "wc4h16cy93xvaj.net"
    web-39 = "awhmdoqexf.com"
    web-40 = "agent-miogaginger.com"
    web-43 = "zpkwtstcucghuy.com"
    web-48 = "xhykcndqlfsnsk.com"
    # web-51 = "27pckzcv8pccn.com"
    # web-52 = "0udnenw27gp.com"
    # web-53 = "attendance-proper.com"
    # web-54 = "charmingagrarian.com"
    # web-55 = "backboneimpinge.com"
    # web-56 = "abattamzwr-gbjr.com"
    # web-57 = "fdiaksbdibct-hsa.com"
    # web-58 = "lzyqqkjtrjnwqoni-myhj.com"
    # web-62 = "gaqgarcwmoylyxgi-iyzd.com"
    # web-63 = "oonp.alive-netksee.com"
    # web-64 = "madjievaness.com"
    # web-65 = "fbnizkgcn.com"
  }
}

```

### 運用ルール

- 作成したい号機のコメントアウトを外す
- 一部リージョンでは一括作成できないため **3〜6台ずつ** 作成する

- 既存号機を削除したい場合  
  → 該当行をコメントアウトして `terraform apply`

- 既存号機を残したまま追加したい場合  
  → 新しい号機のコメントアウトを外して `terraform apply`


---

## ③ Terraform 初期化

```bash
terraform init
```
---

## ④ 実行計画の確認

```bash
terraform plan
```

---

## ⑤ リソース作成（反映）

```bash
terraform apply
```

---

## ⑥ 不要リソースの削除（前回作成分）
```bash
terraform destroy
```

### ルール

-前回作成した方を必ず削除
