---
layout: post
title: "8月9日 アンダースタンディングコンピュテーション読書会 第4回(兵庫県)に参加しました"
date: 2015-08-09 13:02:58 +0900
comments: true
categories: event amagasakirb ruby
---
[8月9日 アンダースタンディングコンピュテーション読書会 第4回(兵庫県)](http://kokucheese.com/event/index/322444/ "8月9日 アンダースタンディングコンピュテーション読書会 第4回(兵庫県)")
に参加しました。
今回は8章から9章でした。

<!--more-->

## メモ

<div style="float:right">
<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&amp;bc1=000000&amp;IS2=1&amp;bg1=FFFFFF&amp;fc1=000000&amp;lc1=0000FF&amp;t=znz-22&amp;o=9&amp;p=8&amp;l=as4&amp;m=amazon&amp;f=ifr&amp;ref=ss_til&amp;asins=487311697X" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</div>

以下、今回のメモです。

- https://github.com/tomstuart/computationbook
- https://github.com/ko1/uc_ja
- p.259 `to_s(2).scan(/.+?(?=.{8}*\z)/)` の書き換えの話
- `reserve` して `each_slice(8)` はどうかという話
- `Kernel#Integer` を使うかどうかという話
- 正規表現の先読みの話 `gsub(/(?=(.{3})+\z)/, ',')` など
- `does_it_say_no.rb` の話
- p.264 監訳注の `"no"` と言って停止というのは環境 (スタックの深さ) に依存して `"yes"` になることもある https://github.com/ko1/uc_ja/issues/7
- Quine から `HQ9+` の話 https://ja.wikipedia.org/wiki/HQ9%2B
- p.276 `Prime.each(n - 1)` の話
- p.296 `Array#product` の話 http://docs.ruby-lang.org/ja/2.2.0/method/Array/i/product.html
- p.299 `T``Type::BOOLEAN` の `T` が余分? https://github.com/ko1/uc_ja/issues/8
- 8章は時間がかかったが9章はすんなり読めた話
- 多重代入と `to_ary` と `to_a` などの話
- SlideShare と SpeakerDeck の違いの話
- `-Float::INFINITY` の話

## 次回の本の候補

- <a href="http://www.amazon.co.jp/gp/product/4274050653/ref=as_li_ss_tl?ie=UTF8&amp;camp=247&amp;creative=7399&amp;creativeASIN=4274050653&amp;linkCode=as2&amp;tag=znz-22">Rubyのしくみ -Ruby Under a Microscope-</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=znz-22&amp;l=as2&amp;o=9&amp;a=4274050653" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
- <a href="http://www.amazon.co.jp/gp/product/4798139823/ref=as_li_ss_tl?ie=UTF8&amp;camp=247&amp;creative=7399&amp;creativeASIN=4798139823&amp;linkCode=as2&amp;tag=znz-22">Effective Ruby</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=znz-22&amp;l=as2&amp;o=9&amp;a=4798139823" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
- <a href="http://www.amazon.co.jp/gp/product/4797376279/ref=as_li_ss_tl?ie=UTF8&amp;camp=247&amp;creative=7399&amp;creativeASIN=4797376279&amp;linkCode=as2&amp;tag=znz-22">10年戦えるデータ分析入門 SQLを武器にデータ活用時代を生き抜く (Informatics &amp;IDEA)</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=znz-22&amp;l=as2&amp;o=9&amp;a=4797376279" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
- <a href="http://www.amazon.co.jp/gp/product/4873117321/ref=as_li_ss_tl?ie=UTF8&amp;camp=247&amp;creative=7399&amp;creativeASIN=4873117321&amp;linkCode=as2&amp;tag=znz-22">ユーザーストーリーマッピング</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=znz-22&amp;l=as2&amp;o=9&amp;a=4873117321" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
- <a href="http://www.amazon.co.jp/gp/product/4627817711/ref=as_li_ss_tl?ie=UTF8&amp;camp=247&amp;creative=7399&amp;creativeASIN=4627817711&amp;linkCode=as2&amp;tag=znz-22">データ解析の実務プロセス入門</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=znz-22&amp;l=as2&amp;o=9&amp;a=4627817711" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
- [Pro Git 日本語版電子書籍公開サイト](https://progit-ja.github.io/ "Pro Git 日本語版電子書籍公開サイト")
- [Ruby on Rails ガイド (4.2 対応)](http://railsguides.jp/index.html "Ruby on Rails ガイド (4.2 対応)")
- <a href="http://www.amazon.co.jp/gp/product/4873116988/ref=as_li_ss_tl?ie=UTF8&amp;camp=247&amp;creative=7399&amp;creativeASIN=4873116988&amp;linkCode=as2&amp;tag=znz-22">実践 機械学習システム</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=znz-22&amp;l=as2&amp;o=9&amp;a=4873116988" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
- <a href="http://www.amazon.co.jp/gp/product/4797376724/ref=as_li_ss_tl?ie=UTF8&amp;camp=247&amp;creative=7399&amp;creativeASIN=4797376724&amp;linkCode=as2&amp;tag=znz-22">ソフトウェアシステムアーキテクチャ構築の原理 第2版 ITアーキテクトの決断を支えるアーキテクチャ思考法</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=znz-22&amp;l=as2&amp;o=9&amp;a=4797376724" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
- <a href="http://www.amazon.co.jp/gp/product/4798139211/ref=as_li_ss_tl?ie=UTF8&amp;camp=247&amp;creative=7399&amp;creativeASIN=4798139211&amp;linkCode=as2&amp;tag=znz-22">システムテスト自動化 標準ガイド (CodeZine BOOKS)</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=znz-22&amp;l=as2&amp;o=9&amp;a=4798139211" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
- <a href="http://www.amazon.co.jp/gp/product/4274069117/ref=as_li_ss_tl?ie=UTF8&amp;camp=247&amp;creative=7399&amp;creativeASIN=4274069117&amp;linkCode=as2&amp;tag=znz-22">型システム入門 −プログラミング言語と型の理論−</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=znz-22&amp;l=as2&amp;o=9&amp;a=4274069117" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />

## 次回予定

<div style="float:right">
<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&amp;bc1=000000&amp;IS2=1&amp;bg1=FFFFFF&amp;fc1=000000&amp;lc1=0000FF&amp;t=znz-22&amp;o=9&amp;p=8&amp;l=as4&amp;m=amazon&amp;f=ifr&amp;ref=ss_til&amp;asins=4274069117" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</div>

次回の本は<a href="http://www.amazon.co.jp/gp/product/4274069117/ref=as_li_ss_tl?ie=UTF8&amp;camp=247&amp;creative=7399&amp;creativeASIN=4274069117&amp;linkCode=as2&amp;tag=znz-22">型システム入門 −プログラミング言語と型の理論−</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=znz-22&amp;l=as2&amp;o=9&amp;a=4274069117" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />に決定しました。
次回は10月の予定で詳細は未定です。
