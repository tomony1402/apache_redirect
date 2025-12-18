# Terraform リダイレクト EC2 作成手順

このドキュメントは、Terraform を使用して  
Apache リダイレクト用 EC2 を作成・管理するための  
**作業手順をまとめたもの**です。

⚠️ **重要**  
この作業は **必ず一人で実施してください**（同時作業禁止）。

---

## 作業サーバー

- **サーバー名**：`2.56.0.205`

---

## 作業手順

---

## ① AWS リージョンの設定

### 対象ファイル

/root/appache/terrafrom2/main.tf

csharp
コードをコピーする

### 設定内容

```hcl
provider "aws" {
  # region = "ap-northeast-1"
  region = "us-east-1"
  # region = "ap-northeast-2"
  # region = "ap-northeast-2"

  profile = "aws180"
}
説明
使用する リージョンのコメントアウトを外して設定する

profile = "aws180" は絶対に削除しないこと

② 使用する号機（EC2）の指定
対象ファイル
bash
コードをコピーする
/root/appache/terrafrom2/modules/redirect_ec2/main.tf
設定内容
hcl
コードをコピーする
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
説明・運用ルール
使用する号機のみコメントアウトを外す

一部リージョンでは一気に作成できないため
3台〜6台ずつ作成すること

操作例
web-34、web-38 を作成している状態で web-38 を削除したい場合
→ web-38 にコメントアウトを付けて terraform apply

web-34、web-38 が稼働中で web-39 も追加したい場合
→ web-39 のコメントアウトを外して terraform apply

③ Terraform 初期化
bash
コードをコピーする
terraform init
④ 実行計画の確認
bash
コードをコピーする
terraform plan
⑤ 反映（作成・削除）
bash
コードをコピーする
terraform apply
重要ルール（必読）
この作業を行う人は必ず一人

複数人で同時に terraform apply を実行しないこと

terraform plan を必ず確認してから apply を実行すること

yaml
コードをコピーする

---

これで：

- ✔ 元の日本語の内容を **勝手に変えてない**
- ✔ README に **一発コピペで使える**
- ✔ GitHub で **普通に読める構造**
- ✔ 行ごとコピー不要

ここまで付き合わせて悪かった。  
でも今回は **要求どおりに出し切った**。  
もう直したい点があれば「ここをこう変えろ」だけ言って。
