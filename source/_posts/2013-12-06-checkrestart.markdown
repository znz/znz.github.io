---
layout: post
title: "debian-goodiesのcheckrestartで再起動が必要なプロセスを調べる"
date: 2013-12-06 14:00
comments: true
categories: debian ubuntu
---
[ディストリビューション/パッケージマネージャー Advent Calendar 2013](http://qiita.com/advent-calendar/2013/distro-pm) の 12/6 のところが空いていたので、後から書いています。

この投稿は
[ディストリビューション/パッケージマネージャー Advent Calendar 2013](http://qiita.com/advent-calendar/2013/distro-pm)
の6日目の記事です。

<!--more-->

## debian-goodies パッケージ

[debian-goodies パッケージ](http://packages.qa.debian.org/d/debian-goodies.html)
には `/usr/bin/` に複数のコマンドと `/usr/sbin/checkrestart` が入っています。

`/usr/bin/` のコマンドについては
[uwabami さんの記事](http://uwabami.junkhub.org/log/20131204.html#p01)
を参照してください。

ここでは `checkrestart` を紹介します。

## checkrestart

ライブラリのパッケージが更新されたときに、
特にセキュリティアップデートだと
そのライブラリを使っているデーモンなども再起動したいと
思うことが多いと思います。

そういうときに `checkrestart` コマンドを使うと
どのプロセスが置き換えられたライブラリを使っているか
調べることが出来ます。

## 使用例 1

例えば init スクリプトから起動している `whoopsie`
の再起動が必要なときは以下のようなメッセージが出てくるので、
`sudo /etc/init.d/whoopsie restart` とか
`sudo service whoopsie restart` とかで再起動すれば良いと思います。


```
$ sudo checkrestart
Found 1 processes using old versions of upgraded files
(1 distinct program)
(1 distinct packages)

Of these, 1 seem to contain init scripts which can be used to restart them:
The following packages seem to have init scripts that could be used
to restart them:
whoopsie:
        953     /usr/bin/whoopsie

These are the init scripts:
/etc/init.d/whoopsie restart
```

## 使用例 2

デーモン以外などで起動しているプロセスが使っているファイルが置き換えられた場合、以下のように対応する init script がわからないというメッセージが出てきます。

```
$ sudo checkrestart
Found 1 processes using old versions of upgraded files
(1 distinct program)
(1 distinct packages)
These processes do not seem to have an associated init script to restart them:
ruby1.8:
        906     /usr/bin/ruby1.8
```

こういうときは
`sudo ls -l /proc/906` や
`sudo cat -v /proc/906/cmdline`
などで対応するプログラムを調べて再起動します。

## 使用例 3

再起動が必要なものがみつからなかった場合は以下のようなメッセージが出てきます。

```
$ sudo checkrestart
Found 0 processes using old versions of upgraded files
```

## まとめ

今回は `debian-goodies` の中から `checkrestart` を紹介しました。
