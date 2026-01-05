---
layout: post
title: "yarn upgrade-interactive --latest を実行する前に yarn.lock を削除してはいけない理由"
date: 2026-01-05
category: TypeScript
---

# はじめに

yarn を利用した TypeScript プロジェクトにおいて、<br>
既存環境のパッケージ更新で少しハマった点があったため、備忘としてまとめます。<br>
**プロジェクト構築者とメンテナンス担当者（後任）が異なるケース**で発生しやすい内容です。<br>
<br>
※ 本記事は、[poetry add @latest で Incompatible constraints エラーが出た原因](https://tatsuichi.github.io/blog/python/2026/01/05/poetry-add-@latest-%E3%81%A7-Incompatible-constraints-%E3%82%A8%E3%83%A9%E3%83%BC%E3%81%8C%E5%87%BA%E3%81%9F%E5%8E%9F%E5%9B%A0.html)の yarn 版です。

# 環境

- WSL 2.6（Ubuntu）
- yarn 1.22

# 前提

- すでに yarn でプロジェクトが作成されている
- プロジェクトの構築者と、現在のメンテナンス担当者が異なる

# 目的

- プロジェクトで使用しているすべてのパッケージを最新化すること
- 具体的には、`yarn upgrade-interactive --latest`で表示されるパッケージをなくすこと

# 想定していたパッケージ更新作業

以下のコマンドで、対話形式でパッケージを更新して、

``` sh
yarn upgrade-interactive --latest
```

更新された`package.json`、`yarn.lock`を Git に push する予定でした

# 実際に行ったパッケージ更新作業

`yarn upgrade-interactive`を実行する前に、<br>
「一度依存関係をきれいにしてから更新したい」と考え、<br>
以下の手順で再インストールを行いました。

``` sh
rm -rf node_modules
rm yarn.lock
yarn install
```

その後に、以下のコマンドを実行したところ、

``` sh
yarn upgrade-interactive --latest
```

すでに多くのパッケージが最新になっており、更新対象がほとんど表示されないという状態になりました。

# 原因

原因は、`yarn.lock`を削除した状態で`yarn install`を実行したことでした。

`yarn.lock` が存在しない状態で `yarn install` を実行すると、<br>
`package.json` に記載されたバージョン制約を基に依存関係が再解決されます。<br>
その結果、以前の `yarn.lock` で固定されていたバージョンより<br>
新しいバージョンがインストールされることがあります。

# まとめ

`yarn`を使ったパッケージ更新では、既存の`yarn.lock`を基準に実行することが重要です。

- `yarn upgrade-interactive --latest`でパッケージ更新し、`package.json`、`yarn.lock`を Git に push する
- `node_modules`、`yarn.lock`を削除した状態での`yarn install`は、挙動を理解した上で最後の手段とする