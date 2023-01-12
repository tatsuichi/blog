---
layout: post
title: "Windows環境でGitHub Pages+Jekyllでブログを始める方法"
date: 2023-01-06
category: Jekyll
---
# はじめに
ブログをGitHub Pagesで始めたときのメモ書きです。
当初、技術メモを、VS Codeを使ってMarkdownで書いて、そのMarkdownを`Markdown PDF: Export (html)`でHtmlに変換して、MarkdownとHtmlの両方をGitHubにアップロードしてました。面倒だったので簡略化できないかと調べていたところ、Jekyllを使ってGitHub PagesでWebページを公開するやり方にたどり着きました。

# 環境
+ Windows 10 64bit

# 構成
![イメージ](/blog/assets/img/GithubPagesとJekyllの構成図.png)
+ [GitHub Pages](https://docs.github.com/ja/pages)
+ [Jekyll](http://jekyllrb-ja.github.io/)

GitHub Pagesとは、[公式サイト](https://docs.github.com/ja/pages/quickstart)より引用

> GitHub Pagesは、GitHubを通じてホストされ、公開されるパブリックなWebページです。 
> 立ち上げて実行するための最速の方法は、<span style="color: red">Jekyll</span> テーマ選択画面を使って事前作成されたテーマをロードすることです。 その後、GitHub Pagesのコンテンツやスタイルを変更できます。

Jekyllとは、[公式サイト](http://jekyllrb-ja.github.io/docs/)より引用
> Jekyllは静的サイトジェネレータです。
> 好きなマークアップ言語で書かれたテキストを用意すれば、Jekyllはレイアウトを合成して静的サイトを作成します。

# 前提
GitHubにブログ用のリポジトリ`blog`が作成されていること<br>
（リポジトリは、`<UserNmae>/<UserNmae>.github.io`ではなく、`<UserNmae>/<リポジトリ名>`です）

# 手順
## WindowsにJekyllをインストールする
+ ローカルPCで動作確認するためにインストールする
+ Jekyllは、Markdownから静的サイトを作成し、ローカルサーバー`localhost:4000`の立ち上げまで、やってくれる。
+ JekyllはRuby製でRubyの実行環境をインストールする必要がある

1. [Rubyのダウンロードサイト](https://rubyinstaller.org/downloads/)より、Rubyのインストーラーをダウンロードする
1. ダウンロードした`rubyinstaller-devkit-3.1.3-1-x64.exe`を実行しRubyをインストールする
1. `gem install jekyll bundler`でJekyllとBundlerをインストールする
1. `jekyll -v`でインストールされたか確認する

## ブログのテーマを選ぶ
+ 自分で1からWebサイトをデザインするのは大変なのでテーマサイトよりテーマを選択する（`jekyll new <フォルダ名>`で初期環境は構築できるがデザイン性は乏しい）
+ テーマサイトには、GitHub上でリポジトリが公開されているものが集められている
+ テーマによってリポジトリの構成が異なるため、後からテーマを変更するのは大変となる（簡単に移行できない）
+ 選択したテーマをテンプレートとして、ブログ記事のみを追加していくイメージ
+ 私は[Monos](http://jekyllthemes.org/themes/monos/)というテーマを選んだ。選定理由は、シンプルなのと、`jekyll new <フォルダ名>`で作成される初期環境とさほど乖離していなかったためである。

1. [Jekyllのテーマサイト](http://jekyllthemes.org/)よりブログのテーマを選ぶ
2. ローカルPCに選んだテーマのリポジトリをCloneする
3. Cloneしたフォルダで`bundle install`を実行し必要な環境をインストールする（エラーがでれば対応する）
4. Cloneしたフォルダで`bundle exec jekyll serve --baseurl ''`を実行しローカルサーバを立ち上げる
5. ブラウザから`http://localhost:4000`にアクセスする

## 選択したテーマをカスタマイズする
Monosテーマの場合で説明します。
1. `https://<UserName>.github.io/<リポジトリ名>`で公開できるようにhtml内のURLを修正<br>（`https://<UserName>.github.io/`を前提としているようだった）
2. デザイン（css）の修正
   + 表の枠線を描く
   + コードスパンの背景色を設定
   + フォントの変更
3. [jekyll-toc](https://github.com/allejo/jekyll-toc)を参考に目次を追加（2023/1/12 追記）

## ブログ記事を書く
Monosテーマの場合で説明します。
1. Markdownでブログ記事を書く
1. `_posts`配下にMarkdownファイルを置く
1. `bundle exec jekyll serve --baseurl ''`を実行し、ブラウザで記事の見栄えを確認する
1. 問題がなければ、ブログ用のリポジトリにPushする

## リポジトリをWebページとして公開する
1. GitHubにサインインする
2. ブログ用のリポジトリで「Settings > Pages」より公開するBranchを選択し保存する
3. ブラウザから`https://<UserName>.github.io/<リポジトリ名>`にアクセスする

![イメージ](/blog/assets/img/GitHubPagesの設定.png)

# 参考サイト
+ WindowsにJekyllをインストールする
  + [Jekyll on Windows](http://jekyllrb-ja.github.io/docs/installation/windows/)
+ Jelleyのテーマについて
  + [GitHub Pagesの使い方 – Jekyllのブログテーマを変更する方法｜Howpon[ハウポン]](https://howpon.com/10476) 
  + [Markdownで執筆できるブログをJekyll + Github Pagesで公開する - Fusic Tech Blog](https://tech.fusic.co.jp/posts/jekyll-githubpages/)
+ トラブルシューティング
  + [Jekyllを実行した時に bundler: failed to load command: jekyll, `require': cannot load such file -- webrick (LoadError) が出るときの対処法](https://tex2e.github.io/blog/ruby/jekyll-cannot-load-webrick)
  + [はじめてのJekyll on Github Pages - たけぞう瀕死ブログ](https://takezoe.hatenablog.com/entry/20140608/p1)
+ 目次（2023/1/12 追記）
  + [#jekyll GitHub Pagesでも目次(ToC)は作れる - Kotet's Personal Blog](https://blog.kotet.jp/2018/04/toc-on-github-pages/)
  + [Jekyll製ブログの記事に目次を追加する · Change Of Pace](https://changeofpace.site/posts/2019-11-10-jekyll-toc)