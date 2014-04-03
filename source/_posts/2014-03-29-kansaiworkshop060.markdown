---
layout: post
title: "第60回 Ruby/Rails勉強会＠関西で正規表現の先読みについてという発表をしました"
date: 2014-03-29 22:00:00 +0900
comments: true
categories: event ruby rubykansai
---
[第50回 Ruby/Rails勉強会＠関西](https://github.com/rubykansai/workshops/wiki/Kansaiworkshop060) に参加して、「正規表現の先読みについて」という発表をしてきました。

<!--more-->

## 発表資料

- https://github.com/znz/regexp-201403
- http://slide.rabbit-shocker.org/authors/znz/regexp-201403/
- https://speakerdeck.com/znz/zheng-gui-biao-xian-falsexian-du-minituite
- http://www.slideshare.net/znzjp/regexp-201403

## デモアプリ

デモに使った Web アプリのソースは
https://gist.github.com/znz/9835956#file-regexp-201403-rb
で公開しています。

heroku にも deploy したので、
http://regexp-201403.herokuapp.com/
で試せます。

### 制限事項

発表で見せる範囲しかちゃんと作っていないので、
変なことをすると変になるのはそういうものです。

対応している正規表現も最初に例として並んでいるもので使っているようなものしか対応していません。

## 発表内容

デモを先に作って後で発表資料を作ったので、デモの方がメインで資料は後付けという感じです。

## KPT

- Keep
  - ちゃんと発表できた。
  - 直前になってしまったけれど、発表資料もちゃんと作成できた。
- Problem
  - 準備の時間があまり取れなかった。
  - 会場の後ろの方でちゃんと見えていたのかとか、聞こえていたのかとかが確認できなかった。
  - 発表資料の公開が遅くなった。(4月3日)
- Try
  - 中継内容の事前確認をしておきたい。録画ありかどうかとか。
  - 今度こそ rabbiter を使いたい。

## 他の発表の感想とか

RSpec 3 対応のライブコーディングは事前にかなりちゃんと準備をしてきた感じがして、
内容も参考になって非常に良かったです。

他はこういうことをやっています、というサービスの紹介っぽい感じが多くて、
プログラミングの話は少なかったので、内容は面白いけど
Ruby/Rails勉強会＠関西向きではないかも、と感じるものもありました。
