# Terraform 運用手順書（Apache Redirect EC2）

本ドキュメントは、Apache リダイレクト用 EC2 を  
Terraform を用いて作成・追加・削除するための  
**作業手順および運用ルール**をまとめたものです。

---

## 前提条件

- Terraform はローカル環境で実行する
- 本作業を行う担当者は **必ず 1 人のみ** とする
- 複数人で同時に `terraform apply` を実行してはならない

---

## 作業環境

- 作業サーバー：`2.56.0.205`
- 作業ディレクトリ：`/root/appache/terrafrom2`

---

## 作業手順

---

## ① AWS リージョンの設定

### 対象ファイル

```text
/root/appache/terrafrom2/main.tf


念のため：コードブロックを外した「そのまま版」

↓ これを README.md にそのまま貼ってOK です。

Terraform リダイレクトEC2 作成手順

本ドキュメントは、Terraform を用いてリダイレクト用 EC2 を作成・管理するための手順をまとめたものです。
この作業は必ず一人で実施してください（同時作業禁止）。

作業サーバー

サーバー名: 2.56.0.205

① AWSリージョンの設定

編集ファイル
/root/appache/terrafrom2/main.tf

provider "aws" {
  # region = "ap-northeast-1"
  region = "us-east-1"
  # region = "ap-northeast-2"

  profile = "aws180"
}

注意点

使用するリージョンのみコメントアウトを外す

profile = "aws180" は絶対に消さないこと

② 使用する号機（EC2）の指定

編集ファイル
/root/appache/terrafrom2/modules/redirect_ec2/main.tf

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

運用ルール

使用する号機のみコメントアウトを外す

一部リージョンでは一気に作成できないため 3〜6台ずつ作成

例

34・38を作成
→ コメントアウトを外して terraform apply

38を削除
→ 38をコメントアウトして terraform apply

34・38稼働中に39を追加
→ 39のコメントアウトを外して terraform apply

③ Terraform 初期化
terraform init

④ 実行計画の確認
terraform plan

⑤ 反映
terraform apply
