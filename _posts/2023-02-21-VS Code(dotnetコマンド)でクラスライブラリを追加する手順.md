---
layout: post
title: "VS Code(dotnetコマンド)でクラスライブラリを追加する手順"
date: 2023-02-21
categories: C#
---
# 環境
- VS Code
- WSL
- .NET 6
- コンソールアプリケーション

# 構成（ゴール）

```text
|--クラスライブラリ
|  |--クラスライブラリ.csproj
|--コンソールアプリケーション
|  |--src
|     |--コンソールアプリケーション.csproj
|  |--test
|     |--テスト.csproj
|--ソリューション.sln
```

# 手順

## クラスライブラリのプロジェクトを作成する
ソリューション（.sln）があるディレクトリで、
```bash
$ mkdir クラスライブラリのディレクトリ名
$ cd クラスライブラリのディレクトリ名
$ dotnet new classlib
```
または
```bash
$ dotnet new classlib -o クラスライブラリ名
```

## ソリューションにクラスライブラリのプロジェクトを追加する
ソリューション（.sln）があるディレクトリで、
```bash
dotnet sln add クラスライブラリ/クラスライブラリ.csproj
```

### コンソールアプリケーションのプロジェクトにクラスライブラリのプロジェクト参照を追加する

```bash
dotnet add コンソールアプリケーション.csproj reference クラスライブラリ.csproj
```
