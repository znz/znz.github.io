---
layout: post
title: "homebrew の reinstall で使われるオプションの保存場所"
date: 2013-09-13 21:58
comments: true
categories: homebrew osx
---

[rabbiter の記事](/blog/2013-09-08-rabbiter-on-mac.html)
で、
「一度 `--with-curl-ca-bundle` 付きでインストールした後だと、以下のようにオプションなしの reinstall でもオプションが付くようです。」
と書きましたが、そのオプションがどこに保存されているのかを調べました。

<!--more-->

実際に調べるのはソースをたどったりして結構大変だったのですが、
最終的に
`/usr/local/Cellar/glib-networking/2.36.2/INSTALL_RECEIPT.json`
に `used_options` というキーで保存されているということがわかりました。

`Cellar` はインストールしたファイルの実体が入るところなので、
あらかじめダミーの `INSTALL_RECEIPT.json` を用意しておいて
オプションなしの `brew reinstall` の時に使われるオプションを
埋め込んでおくという用途に使うのには向いていないということが
わかりました。
