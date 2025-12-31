---
layout: post
title: "API Gateway + Cognito Authorizer : なぜ ID トークンで問題が起きなかったのか"
date: 2025-12-30
category: AWS
---

# はじめに

API Gateway（REST API）+ Lambda + DynamoDB でバックエンドを構成してます。<br>
API Gateway に Cognito Authorizer を設定して REST API の認証を実現してます。<br>
Cognito Authorizer には、ID トークンを指定してます。<br>
設計当時は、深く考えずに ID トークン を採用してましたが、<br>
「なぜそれで問題が起きなかったのか」「アクセストークンではどうなるのか」を改めて整理しました。<br>
偶然の選択でしたが、今振り返るとこの構成では ID トークンの方が自然でした。

## 前提

システム仕様

- 複数の権限が存在する
- 権限によって、操作できる機能が異なる
- アカウント毎に権限を適用する
- 権限は、リクエスト毎に最新の状態で評価する

## バックエンド構成

![バックエンド構成](/blog/assets/img/2025-12-30/API%20Gateway+Cognito%20Authorizer.drawio.png)

## 責務

アカウントは、Cognito と DynamoDB で 2 重管理となります。<br>
API Gateway + Cognito は、認証のみを行い、権限チェックはしません。<br>
Lambda は、渡ってきたユーザー情報を基に、DynamoDB から権限を取得して権限チェックします。

| AWS サービス | 責務                                                  |
| ------------ | ----------------------------------------------------- |
| Cognito      | アカウント管理 + 認証                                 |
| API Gateway  | トークンを Cognito に渡す、検証成功後は Lambda に渡す |
| Lambda       | 認可                                                  |
| DynamoDB     | 権限を含むアカウント情報の保持                        |

## API Gateway で認証する理由

- 認証処理を一元管理したいため
- なるべく早い段階で認証したいため
- Lambda に不正リクエストを到達させたくないため

## ID トークンを採用した理由

設計当時は知識がなかったため、調べたところ ID トークンを指定している例が多かったため、ID トークンを採用しました。<br>
改めて[公式サイト](https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/apigateway-integrate-with-cognito.html)を見ると、ID トークン、アクセストークン、どちらも指定できることが分かりました。

## アクセストークンを採用するとどうなるか

アクセストークンを指定したところ`401`になりました。<br>
これは、以下の設定がされていなかったためでした。

- Cognito
  - リソースサーバーを作成する
  - スコープを定義する
- API Gateway
  - 認可スコープを設定する

## まとめ

結果論ですが、本システムでは以下の理由から ID トークンの採用は自然でした。

- API Gateway では認証のみを行い、認可は Lambda 側に委ねている
- 認可スコープ設計が不要で、構成がシンプル
- 権限を DynamoDB で一元管理でき、リクエスト毎に最新状態を評価できる

トークンの選択は、システム全体を俯瞰した認証、認可の責務に依存するものだと分かりました。
