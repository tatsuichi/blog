---
layout: post
title: "AWSサーバーレス鉄板構成（API Gateway + Lambda + DynamoDB）の設計上の注意点"
date: 2026-01-02
category: AWS
---

# はじめに

API Gateway + Lambda + DynamoDB は [AWS サーバーレスアーキテクチャ](https://aws.amazon.com/jp/serverless/patterns/serverless-pattern/)の鉄板構成です。<br>
[AWS Serverless Application Model（AWS SAM）](https://docs.aws.amazon.com/ja_jp/serverless-application-model/latest/developerguide/what-is-sam.html)を利用すると初期構築が早く Poc としては強力ですが、<br>
長期運用を見据えると事前に理解しておくべき制約があります。

# 基本構成

バックエンドは、AWS サーバーレスアーキテクチャの典型例である API Gateway + Lambda + DynamoDB で構成します。<br>
フロントエンドとバックエンドは、将来の変更を考慮し、REST API を介して疎結合にします。<br>
<br>

![AWSサーバーレスアーキテクチャ](/blog/assets/img/2026-01-02/AWSサーバーレスアーキテクチャ.drawio.png)
<br>
<br>

| AWS サービス            | 役割                                     |
| ----------------------- | ---------------------------------------- |
| API Gateway（REST API） | フロントエンドからのリクエストを受け取る |
| Lambda                  | 業務ロジック                             |
| DynamoDB                | データ保持                               |

# PoC に強い理由

[AWS Serverless Application Model（AWS SAM）](https://docs.aws.amazon.com/ja_jp/serverless-application-model/latest/developerguide/what-is-sam.html)を利用することで、初期構築から検証まで短期間で行えます。

- プロジェクトのひな型が用意されている
- デプロイ手順が単純
- 検証までのリードタイムが短い

そのため、「まず動かす」ことにおいて有効だと考えます。

# API Gateway（REST API）のメリット

[OpenAPI YAML をインポートできる](https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/api-gateway-import-api.html)ため、API 仕様を起点に設計できます。<br>
フロントエンドと分業できる点は、チーム開発においてメリットと考えます。


# Lambda の制約と設計判断

## 制約

- Lambda の起動時間が遅い（コールドスタート問題）
  - 回避策（SnapStart、Provisioned Concurrency）はコストが高い
    - 従量課金というサーバーレスのメリットを活かしづらい
- Lambda を同時に実行できる数に上限がある
  - 引き上げるには AWS に申請が必要

## 設計判断

- Lambda は「非同期でサクッと終わる処理」に適している
- [Lambda が失敗することを前提としたアーキテクチャとする](https://aws.amazon.com/jp/builders-library/timeouts-retries-and-backoff-with-jitter/)
  - ジッター付きリトライ
  - エンドユーザー操作による再実行

# DynamoDB の設計におけるトレードオフ

DynamoDB は、RDB とは異なり、アクセスパターンを基に設計します。<br>

``` text
アプリ UI（デザイン） -> アクセスパターン -> DynamoDB テーブル設計
```
UI 変更がそのままテーブル設計に影響するため、本来は疎結合であるべき UI と DB が、密結合になりやすい点に注意が必要です。<br>
ただし、KVS のためスキーマ設計が柔軟、コネクション管理が不要、といったメリットがあります。<br>
また、`limit` / `offset` に相当するクエリがないため、一般的なページネーションを前提とした一覧 UI とは相性が悪いです。

# まとめ

API Gateway + Lambda + DynamoDB は、Poc としては非常に有効な構成です。<br>
長期運用や将来の要件変更を見据えると、各サービスの制約を理解した上で、疎結合を意識して、代替可能な構成として設計することが重要です。

# 余談

CloudWatch Logs Insights は便利ですが、クエリ条件次第ではコストが急増するため注意が必要となります。
