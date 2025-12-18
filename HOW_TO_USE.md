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

