---
layout: post
title: "www.ruby-lang.org への pull request の出し方"
date: 2013-09-30 22:43
comments: true
categories: ruby community github
---
[Ruby 2.1.0-preview1 is released](https://www.ruby-lang.org/en/news/2013/09/23/ruby-2-1-0-preview1-is-released/)
というアナウンスが一週間経ってもまだ日本語訳が出来ていなかったので、
ちょっとの暇を見つけて pull request を出してみたので、
その手順をまとめておきます。

<!--more-->

まず github のアカウントは持っているというのは前提として話を進めます。

初回は
https://github.com/ruby/www.ruby-lang.org
の右上から自分のアカウントに Fork します。

すでに Fork していて二回目以降は upstream の変更を merge する必要があります。
[Syncing a fork](https://help.github.com/articles/syncing-a-fork)
を参考にして merge します。

実行したコマンドだけまとめておきます。
詳細は github のヘルプを参照してください。

```console
% git clone git@github.com:znz/www.ruby-lang.org.git
% cd www.ruby-lang.org
% git remote add upstream https://github.com/ruby/www.ruby-lang.org.git
% git remote -v
% git fetch upstream
% git branch -va
% git checkout master
% git merge upstream/master
% git push
```

英語版を元にして日本語版を作成します。

```console
% cp {en,ja}/news/_posts/2013-09-23-ruby-2-1-0-preview1-is-released.md
% git add ja/news/_posts/2013-09-23-ruby-2-1-0-preview1-is-released.md
% vi ja/news/_posts/2013-09-23-ruby-2-1-0-preview1-is-released.md
```

プレビュー用に `bundle install` します。

```console
% bundle install
% rake -T
% rake preview
```

`rake preview`
でサーバーが起動しているので、
ブラウザで
`http://localhost:4000/`
を開いて表示を確認します。

github に push して、
ブラウザから pull request を出します。

ブランチを切り忘れていたので、
ここで作りました。

```console
% git checkout -b ruby210preview1
% git commit -av
% git push --set-upstream origin ruby210preview1
```

最後に
`git checkout master`
で master ブランチに戻るなり、
消してしまって、
また作業する時に clone し直すなりします。
