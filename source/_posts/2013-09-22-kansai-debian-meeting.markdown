---
layout: post
title: "第 76 回 関西 Debian 勉強会に参加した"
date: 2013-09-22 13:40
comments: true
categories: event debian kansaidebian
---
[第 76 回 関西 Debian 勉強会](https://wiki.debian.org/KansaiDebianMeeting/20130922)
に参加してきました。

「Linuxとサウンドシステム」
という主に ALSA の話と
「dgit でソースパッケージを触ってみる」
という話でした。

<!--more-->

## 事前課題

「libasound2-data」が wheezy には無いという無理難題だったので、
[debian-usersで質問してみた](http://mla.n-z.jp/?debian-users:56906)
ところ、
[wheezyではlibasound2がインストールされていれば大丈夫](http://mla.n-z.jp/?debian-users:56907)
という話でした。

## Linuxとサウンドシステム

主に ALSA の話で、
サウンドシステム周りはいろいろ関連するものが多くて大変そうな印象を受けました。

説明以外の実際にコマンドを試す部分では、

* `/dev/snd/`
* `/proc/asound/`
* `/proc/asound/cards`
  (ALSA が認識しているカードの一覧)
* `/usr/include/sound/asound.h`
  (`linux-libc-dev` パッケージに入っていた)
* `/proc/asound/card0/pcm0p/sub0/status`
  (再生出来ないときのトラブルシューティングに有用、入力側は pcm0c)
* `/sys/class/sound/`
* `/usr/share/alsa/`
  (`libasound2` の設定ファイルとか)

あたりの話がありました。

最終的には時間が足りなくて、続きは次回ということになりました。

## dgit でソースパッケージを触ってみる

いきなり現状は DD (Debian Developer) じゃないと使えないツールという話でした。
そのうち一般ユーザーでも使えるようになるかも、ということで、
主に調べた内容の発表でした。

`dgit` を使っているパッケージは
http://anonscm.debian.org/gitweb/
の `dgit-repos/` でわかるようです。

最後は
Debian Developer
の矢吹さんに実際に使えるところを
デモしてもらえて終わりました。
