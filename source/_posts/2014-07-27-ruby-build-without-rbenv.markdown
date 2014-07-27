---
layout: post
title: "ruby-buildをrbenvと組み合わせずに使う"
date: 2014-07-27 17:08:00 +0900
comments: true
categories: ruby
---
[ruby-build](https://github.com/sstephenson/ruby-build "ruby-build")
は
[rbenv](https://github.com/sstephenson/rbenv "rbenv")
と組み合わせて使われることが多いですが、
単独でも使えて、その情報が少ないので少し書いておきます。

<!--more-->

## `/usr/local` にインストールする方法

`ruby-build` 自体もインストールしてしまう場合は
`install.sh` を使ってインストールします。

```sh
git clone --depth 1 https://github.com/sstephenson/ruby-build
ruby-build/install.sh
rm -rf ruby-build
ruby-build 2.1.2 /usr/local
```

もっと詳しい使い方は
[[ReVIEW Tips] DockerでRe:VIEW](http://qiita.com/takahashim/items/406421d515ef1d4f1189 "[ReVIEW Tips] DockerでRe:VIEW")
が参考になると思います。

## ruby だけインストールする方法

`bin/ruby-build` を直接実行すれば `ruby-build` をインストールせずに
`ruby` だけインストールすることもできます。

```sh
git clone --depth 1 https://github.com/sstephenson/ruby-build
ruby-build/bin/ruby-build 2.1.2 /usr/local
rm -rf ruby-build
```

ドキュメントの生成を止めたり、
インストール中のメッセージを出したりするために
以下のように実行するのも良いと思います。

```sh
git clone --depth 1 https://github.com/sstephenson/ruby-build
export CONFIGURE_OPTS="--disable-install-doc"
ruby-build/bin/ruby-build --verbose 2.1.2 /usr/local
rm -rf ruby-build
```
