---
layout: post
title: "bundled gem と default gem の違い"
date: 2016-09-10 18:55:55 +0900
comments: true
categories: ruby
---
RubyKaigi 2016 の後の移動中に hsbt さんに bundled gem と default gem との違いについて聞いてみた話をまとめてみました。

<!--more-->

## 違い

- bundled gem は単なる gem で gem uninstall もできる普通の gem
- default gem は
  - uninstall できない
  - bundler の `clean_env` 環境でも見える
  - bundler で別のバージョンを指定してインストールしていれば、通常の bundler の load path の挙動に従って、そちらが使われる
  - たとえば ruby 2.3.1 だと `lib/ruby/gems/2.3.0/gems/rdoc-4.2.1` に `bin/rdoc` と `bin/ri` しかなくて他は `lib/ruby/2.3.0/rdoc*` などの標準添付のところに入っている

というような違いだと聞きました。

調べてみたところ、他には

- `$(gem env gemdir)/specifications/default` に `*.gemspec` ファイルが入っている
- 新しい rubygems だと `gem list` で `json (default: 2.0.2)` のように `default:` がつく

という違いがあるようでした。

## 続く

続くかも。
