---
layout: post
title: "Rubyによるクローラー開発技法を読んだ"
date: 2014-12-30 18:21:00 +0900
comments: true
categories: ruby book
---
次回の
[1月10日 Rubyによるクローラー開発技法　読書会　第3回(兵庫県)](http://kokucheese.com/event/index/247692/ "1月10日 Rubyによるクローラー開発技法　読書会　第3回(兵庫県)")
は
[LILO ＆ 東海道らぐ・オフラインミーティング（2015/01/10） - LILO | Doorkeeper](http://lilo.doorkeeper.jp/events/18987 "LILO ＆ 東海道らぐ・オフラインミーティング（2015/01/10） - LILO | Doorkeeper")
と重なっていて参加できないということで、Rubyによるクローラー開発技法を読んでしまったので、そのメモです。

<!--more-->

## メモ

<div style="float:right">
<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&amp;bc1=000000&amp;IS2=1&amp;bg1=FFFFFF&amp;fc1=000000&amp;lc1=0000FF&amp;t=znz-22&amp;o=9&amp;p=8&amp;l=as4&amp;m=amazon&amp;f=ifr&amp;ref=ss_til&amp;asins=4797380357" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</div>

- p.62 `robotex.rb` の 4 行目は 3 行目の引数にならずに意味がないのでは。
- p.67 `anemone-skip_links_like.rb` で「adminを含むURLは除外」と書いてあるのに `/\/r\//` になっている。
- p.69 `http://example.com` と `http://example.com/` は同じ意味で 301 リダイレクトは発生しないのでは。
- p.143 「スクリーンショットはページごとに上書きされていって、」とあるが、ここは上書きされないのでは。
- p.173 `Nokogiri#parse` などは `Nokogiri.parse` (インスタンスメソッドではなくクラスメソッド) なのでは。
- p.215 `initialize` の `#{}` は不要なのでは。
- p.239 `page.code = 500` は `page.code == 500` の間違い?
- p.258 `base_url` への代入が連続しているのが不要
- p.359 プスクリプト → スクリプト
- p.420 varidation → validation
- p.420 gsub の結果は Integer にならないのでは?
