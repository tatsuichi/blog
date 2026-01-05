---
layout: post
title: "poetry add @latest で Incompatible constraints エラーが出た原因"
date: 2026-01-05
category: Python
---

# はじめに

[Poetry](https://python-poetry.org/) を利用した Python プロジェクトにおいて、<br>
既存環境のパッケージ更新で少しハマった点があったため、備忘としてまとめます。<br>
**プロジェクト構築者とメンテナンス担当者（後任）が異なるケース**で発生しやすい内容です。<br>

# 環境

- WSL 2.6（Ubuntu）
- poetry 2.1

# 前提

- すでに Poetry でプロジェクトが作成されている
- プロジェクトの構築者と、現在のメンテナンス担当者が異なる

# 目的

- プロジェクトで使用しているすべてのパッケージを最新化すること
- 具体的には、`poetry show --outdated`で表示されるパッケージをなくすこと

# パッケージ更新

[poetryの日本語サイト](https://cocoatomo.github.io/poetry-ja/cli/)から、以下の使い分けと理解してます。

## 制約内のバージョンアップ

`pyproject.toml`に記載されているバージョン制約内でバージョンアップする

``` sh
poetry update
```

## 制約外のバージョンアップ

`pyproject.toml`に記載されているバージョン制約自体を更新する

``` sh
poetry add
```

最新バージョンにする場合は`@latest`を付ける

``` sh
poetry add パッケージ名@latest
```

# エラー発生手順

以下の手順で、パッケージを更新しました。

1. 現状のパッケージバージョンを確認

    ``` sh
    poetry show --outdated
    ```

1. 制約内のバージョンアップを実行

    ``` sh
    poetry update
    ```

1. 更新結果を確認（期待通り、パッケージ数が少なくなる）

    ``` sh
    poetry show --outdated
    ```

1. 制約外のバージョンアップを実行

    ``` sh
    poetry add パッケージ名@latest
    ```

1. 以下のエラーが発生

    ``` text
    Incompatible constraints in requirements of プロジェクト名
    ```

# 原因と対処

依存関係が以下のようにグループに分かれておりました。

- プロダクション用
- 開発用（dev）

`poetry update`はグループを意識せずに更新してくれますが、<br>
`poetry add`は、どのグループに属するパッケージかを明示的に指定する必要がありました。<br>

そのため、正しくは次のように指定します。

``` sh
poetry add パッケージ名@latest --group グループ名
```

# まとめ

Poetry プロジェクトを後任としてメンテナンスする場合は、以下のことを理解しておくとトラブルに強くなります。

- パッケージがどのグループ（dev など）に属しているか
- `poetry update`と`poetry add`の挙動の違い

パッケージ更新前に、`pyproject.toml`の`tool.poetry.group`を事前に確認することを推奨します。

## 更新方法の整理

- 制約内更新 → `poetry update`
- 制約外更新 → `poetry add パッケージ名@latest --group グループ名`