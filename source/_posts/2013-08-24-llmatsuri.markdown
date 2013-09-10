---
layout: post
title: "LLまつりでLTしました"
date: 2013-08-24
comments: true
categories: event shell LL
---
[LLまつり](http://ll.jus.or.jp/2013/) で「シェルスクリプトで簡易ping監視」というタイトルでライトニングトークをしてきました。

<!--more-->

スライドは http://slide.rabbit-shocker.org/authors/znz/simple-ping-summary/ や slideshare で公開しています。
スライドのソースも https://github.com/znz/simple-ping-summary-slide で公開しています。

監視プログラム自体は https://github.com/znz/simple-ping-summary で公開しています。

## バージョンソート

[GNU coreutils の sort](http://linuxjm.sourceforge.jp/html/GNU_coreutils/man1/sort.1.html)
や
[ls](http://linuxjm.sourceforge.jp/html/GNU_coreutils/man1/ls.1.html)
にバージョンソートという機能があるのですが、
それが IPv4 アドレスのソートにも使えるということが
一番言いたかったことでした。

## 画像生成

画像を作成する時はテキストエディタで作成出来る XBM や XPM を使うことが多いのですが、
後で twitter の反応を確認してみると SVG という案もあったようなので、
機会があれば使ってみたいと思いました。

## 会場での実行結果

最後にデモとしてちょっと見せようとして、説明は間に合わなかったLLまつりの会場でのデータは [llmatsuri ブランチ](https://github.com/znz/simple-ping-summary/tree/llmatsuri) にあるので `git clone https://github.com/znz/simple-ping-summary` して `git checkout llmatsuri` で `llmatsuri` ブランチに切り替えて `_img/summary.html` を開くと見えます。
発表後のデータもあるので、ちらっと見せたものよりちょっと増えています。

会場では一般の PC などが対象なので、監視対象になることが前提で PING 応答が許可されているルーターとは違って、 DHCP で IP アドレスが割り当てられていても応答がないことが多いので、結構歯抜けになっています。

DHCP の割り当て範囲がどうなっていたのかわからなかったので、対象は適当な範囲だったのですが、 [192.168.4.1から254の範囲の結果](https://github.com/znz/simple-ping-summary/blob/llmatsuri/_img/192.168.4/20130824.png) をみると、徐々にこの範囲の割り当てが増えていっていると推測出来そうです。

## slideshare

<iframe src="http://www.slideshare.net/slideshow/embed_code/25545146" width="427" height="356" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC;border-width:1px 1px 0;margin-bottom:5px" allowfullscreen webkitallowfullscreen mozallowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="https://www.slideshare.net/znzjp/simple-ping-summary" title="シェルスクリプトで簡易ping監視" target="_blank">シェルスクリプトで簡易ping監視</a> </strong> from <strong><a href="http://www.slideshare.net/znzjp" target="_blank">Kazuhiro Nishiyama</a></strong> </div>

## Rabbit SlideShow

<iframe src="http://slide.rabbit-shocker.org/authors/znz/simple-ping-summary/viewer.html"
        width="640" height="524"
        frameborder="0"
        marginwidth="0"
        marginheight="0"
        scrolling="no"
        style="border: 1px solid #ccc; border-width: 1px 1px 0; margin-bottom: 5px"
        allowfullscreen> </iframe>
<div style="margin-bottom: 5px">
  <a href="http://slide.rabbit-shocker.org/authors/znz/simple-ping-summary/" title="シェルスクリプトで簡易ping監視">シェルスクリプトで簡易ping監視</a>
</div>

