---
layout: post
title: "LILO ＆ 東海道らぐのイベントに参加しました"
date: 2014-05-03 23:00:00 +0900
comments: true
categories: lilo event linux
---
[LILO ＆ 東海道らぐ・オフラインミーティング（2014/05/03）](http://lilo.doorkeeper.jp/events/10790) に参加しました。

アンカンファレンス形式で発表したい人が順番に前に出て話をするという形式でした。

<!--more-->

## 内容

大部分は資料を用意しての発表という形式ではなくて、
発表のタイトルもちゃんとあるわけではなかったので、
大雑把にこういう話でしたというメモだけです。

- 自己紹介1
- しまださんの東海道らぐの紹介
- 自分の `libpam-google-authenticator` の話
- 自己紹介2 (後から来た人の自己紹介)
- えのきさんの LibreOffice の QA などの相談 (ディスカッション)
- さとうさんの Raspberry Pi の話
- 矢吹さんの LXC などの話
- 自己紹介3 (さらに後から来た人の自己紹介)
  - pandoc がオススメ
- としひささんの Raspberry Pi で NTP Stratum 1 Server の話
  - [Raspberry Pi で NTP Stratum-1 Server を作る． | tosihisa's public notebook](http://tosihisa.postach.io/raspberry-pi-de-ntp-stratum-1-server-wozuo-ru)
  - http://www.pool.ntp.org/scores/60.56.214.78
  - `ntpdate -quv ntp.netfort.gr.jp` で確認できました。
- 守屋さんの Linux の昔話とかの話
- こんどうさんの Flightgear の話
- 大浦さんの 文字書き起こしツール [SMART-GS](http://sourceforge.jp/projects/smart-gs/) の話
- さかもとさんのカーネルモジュールの話
  - http://netcat.bandcamp.com/
  - https://github.com/usrbinnc/netcat-cpi-kernel-module

## 感想

いろいろな内容の話が聞けて面白かったです。

自分が話をするのは、 LILO のサーバー移行の話という案もあったのですが、
それはちゃんとまとめる時間をとってからの方が良さそうと思って、
事前に一度試していたことのあった `libpam-google-authenticator` を前の晩に試し直しながらデモの順番を考えて、
その話をしました。
