---
layout: post
title: "Mac で rabbiter が動いた"
date: 2013-09-08 21:30
comments: true
categories: rabbit osx homebrew
---
homebrew を使った Mac OS X 10.8.4 環境で rabbiter (2.0.1) が動くようになりました。

<!--more-->

rabbiter の issues に報告していた
[show_uri の問題](https://github.com/rabbit-shocker/rabbiter/issues/1)
と
[glib-networking のルート証明書の問題](https://github.com/rabbit-shocker/rabbiter/issues/2)
が解決したので、 `glib-networking` を `brew reinstall glib-networking --with-curl-ca-bundle` でインストールすれば使えるようになりました。
ちなみに、初回のインストールでも `brew install` ではなく `brew reinstall` で大丈夫のようです。

以下のように `configure` に `--with-ca-certificates` が付いていれば使えます。

```
% brew reinstall glib-networking --with-curl-ca-bundle
==> Reinstalling glib-networking --with-curl-ca-bundle
==> Downloading http://ftp.gnome.org/pub/GNOME/sources/glib-networking/2.36/glib-networkin
Already downloaded: /Library/Caches/Homebrew/glib-networking-2.36.2.tar.xz
==> ./configure --with-ca-certificates=/usr/local/opt/curl-ca-bundle/share/ca-bundle.crt -
==> make install
🍺  /usr/local/Cellar/glib-networking/2.36.2: 68 files, 352K, built in 18 seconds
```

以下のように `configure` に `--without-ca-certificates` と付いているときは rabbiter が使えません。

```
% brew reinstall glib-networking
==> Reinstalling glib-networking
==> Downloading http://ftp.gnome.org/pub/GNOME/sources/glib-networking/2.36/glib-networkin
Already downloaded: /Library/Caches/Homebrew/glib-networking-2.36.2.tar.xz
==> ./configure --without-ca-certificates --prefix=/usr/local/Cellar/glib-networking/2.36.
==> make install
🍺  /usr/local/Cellar/glib-networking/2.36.2: 68 files, 352K, built in 21 seconds
```

一度 `--with-curl-ca-bundle` 付きでインストールした後だと、以下のようにオプションなしの reinstall でもオプションが付くようです。

```
% brew reinstall glib-networking
==> Reinstalling glib-networking --with-curl-ca-bundle
==> Downloading http://ftp.gnome.org/pub/GNOME/sources/glib-networking/2.36/glib-networkin
Already downloaded: /Library/Caches/Homebrew/glib-networking-2.36.2.tar.xz
==> ./configure --with-ca-certificates=/usr/local/opt/curl-ca-bundle/share/ca-bundle.crt -
==> make install
🍺  /usr/local/Cellar/glib-networking/2.36.2: 68 files, 352K, built in 18 seconds
```
