---
layout: post
title: "heroku本を読んだ"
date: 2015-01-03 16:58:56 +0900
comments: true
categories: book heroku
---
<a href="http://www.amazon.co.jp/gp/product/4048915134/ref=as_li_ss_tl?ie=UTF8&amp;camp=247&amp;creative=7399&amp;creativeASIN=4048915134&amp;linkCode=as2&amp;tag=znz-22">プロフェッショナルのための 実践Heroku入門 プラットフォーム・クラウドを活用したアプリケーション開発と運用 (書籍)</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=znz-22&amp;l=as2&amp;o=9&amp;a=4048915134" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
を読んだので、そのメモです。

<!--more-->

内容としては heroku をまだ使ったことない人には特におすすめで、使ったことがある人にもどういう思想で作られているのかなど参考になるのでおすすめだと思いました。
特に最後の The Twelve Factor App の翻訳は heroku に限らず参考になると思いました。

## メモ

<div style="float:right">
<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&amp;bc1=000000&amp;IS2=1&amp;bg1=FFFFFF&amp;fc1=000000&amp;lc1=0000FF&amp;t=znz-22&amp;o=9&amp;p=8&amp;l=as4&amp;m=amazon&amp;f=ifr&amp;ref=ss_til&amp;asins=4048915134" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</div>

- ceder → cedar (全般的に)
- p.47 rubyforge はもう終了しているのでダウンロードできなさそう。
- p.49 `brew --prefix readline` と `brew --prefix openssl` の周りに `$( )` が抜けている?
- p.55 gem install するのは bundle よりも bundler の方が良さそう。
- p.60 下から4行目 Frameowk → Framework
- p.69 下から2行目 PostgresSQL → PostgreSQL
- p.70 `(後述)` とあるが既に説明済み?
- p.79 webtype → web type
- p.79 `jobs:worK` → `jobs:work` ? (2カ所)
- p.91 newrelick → newrelic
- p.92 MG → MB ? (3カ所)
- p.101 sandobox → sandbox ?
- p.135 ctext 型をの → citext 型を
- p.159 注7 の URL がパスしかない
