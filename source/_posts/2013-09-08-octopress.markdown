---
layout: post
title: "octopress"
date: 2013-09-08 22:58
comments: true
categories: octopress
---
octopress で github pages を作ってみました。
インストール記事などは複数あるので気になった点を列挙するだけにします。

<!--more-->

* ruby のバージョン
  * 2.0.0 ではなく 1.9.3 になっていたが、 github-pages という gem でわかるように github pages で使われているのが 1.9.3 のようなので、 1.9.3 を使うことにした。
  * 後でわかったんですが、 octopress の場合はローカルで生成するようなので、 2.0.0 でも問題なく使えそうでした。
  * `.ruby-version` に `1.9.3` とだけあって、 rbenv のパッチレベル付きのバージョンにマッチしなかったので、最初は `rbenv shell 1.9.3-p448` で作業していました。 (後で `rbenv local 1.9.3-p448` にしました。)
* 使い始め
  * `git clone git://github.com/imathis/octopress.git octopress`
  * `cd octopress`
  * `rake setup_github_pages`
    * `Repository url: git@github.com:znz/znz.github.io`
  * `_config.yml` を設定
  * `rake install`
  * `_config.yml` を設定
  * `rake deploy`
  * `rake new_post`
    * `Enter a title for your post:` ときいてくるので `rake new_post[title]` よりも楽
  * `rake preview` で確認しながら書く
  * `rake gen_deploy`
    * `rake generate` 相当がないと変更が反映されない (`_config.yml` を変更したときなど)
* `CNAME`
  * octopress 側では `source/CNAME` に置く必要がありました。
* Excerpts
  * http://octopress.org/docs/blogging/plugins/ にあるように `<!--more-->` と書いておくとトップに表示される内容を最初の方だけに出来ました。
* ブランチ
  * 手元では source ブランチで作業するようになっていて、 github pages へ deploy される master ブランチは `_deploy` ディレクトリにある別レポジトリにありました。
