---
layout: post
title: "Land of Lisp 読書会 最終回（第5回）(兵庫県)に参加してきた"
date: 2014-02-22 22:45:00 +0900
comments: true
categories: event amagasakirb land-of-lisp
---
[2月22日 Land of Lisp 読書会 最終回（第5回）(兵庫県)](http://kokucheese.com/event/index/144998/)
に参加してきました。
今回は第16章から最後まででした。

<div style="float:right">
<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&amp;bc1=000000&amp;IS2=1&amp;bg1=FFFFFF&amp;fc1=000000&amp;lc1=0000FF&amp;t=znz-22&amp;o=9&amp;p=8&amp;l=as4&amp;m=amazon&amp;f=ifr&amp;ref=ss_til&amp;asins=4873116155" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</div>

次回は4月5日で<a href="http://www.amazon.co.jp/gp/product/4873116155/ref=as_li_ss_tl?ie=UTF8&amp;camp=247&amp;creative=7399&amp;creativeASIN=4873116155&amp;linkCode=as2&amp;tag=znz-22">RとRubyによるデータ解析入門</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=znz-22&amp;l=as2&amp;o=9&amp;a=4873116155" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />の読書会に決まりました。

<!--more-->

## 会場

今回は今までと違って、普通の会議室がとれなかったということで音楽室でした。
静かだったり、外から見えるようになっていたりしたのが印象的でした。

## メモ

以下今回のメモです。

- `,` が気持ち悪い。
- clojure だと `~`
- <code>`</code> の話とマクロの話は別
- Lisp のマクロの話と Ruby に似たようなものがあるかという話
- Ruby の `eval` と `class_eval` と `instance_eval` の話
- SVG の話
- imagemagick とか minimagick とかでも画像生成できる話 (例: http://jarp.does.notwork.org/diary/201308a.html )
- c14n (XML Canonicalization) してインデントを付けると読みやすくなる話
- middleman : rails の asset pipe line などの技術を使って静的サイトが作れる
- WMI : SNMP の Windows 専用版みたいなもの
- PowerShell の話
  - パイプをオブジェクトが流れるのがモナドとか jQuery とかっぽい
  - 他の言語の printf などと全く違う。エスケープがバッククオートとか
  - WSHは32ビットと64ビットの問題がある?
- 単なる便利なライブラリと DSL の違いは?
  - 外部 DSL だとわかりやすいけど、内部 DSL だと曖昧
  - その他いろいろな意見あり
- Ruby だと内部 DSL は括弧を省略するのが普通
- Gemfile とか Rakefile とか
- rspec の話
- 18章「Haskell や Clojure では、遅延評価が言語のコアでサポートされている。」
- p.378 value が nil になることもあるから forced は必要そう
- lazy-car や lazy-cdr は lazy を作るのではなく force で取り出すので lazy-cons や lazy-nil と意味が違ってわかりにくい
- マンガはコマ割がわかりにくいし p.432 には矢印まであるのは矢印を付けるよりもコマ割を工夫すべきでは

## 次回の予定

- エリック・エヴァンスのドメイン駆動設計
- クラウドデザインパターン
- SQLアンチパターン
- RとRubyによるデータ解析入門

などの候補が出ましたが R と Ruby の本に決定して、
amagasakirb としては久しぶりの Ruby の本ということになりました。
