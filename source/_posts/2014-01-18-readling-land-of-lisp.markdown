---
layout: post
title: "Land of Lisp 読書会 第4回(兵庫県)に参加してきた"
date: 2014-01-18 23:45:00 +0900
comments: true
categories: event amagasakirb land-of-lisp
---
[amagasakirb](http://kokucheese.com/main/tag/amagasakirb)
の
[1月18日 Land of Lisp 読書会 第４回(兵庫県)](http://kokucheese.com/event/index/129358/)
に参加してきました。
今回は「第IV部 LISPは科学なり」の最初の方の14章と15章でした。

次回は2月22日ということでした。

<!--more-->

<div style="float:right">
<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&amp;bc1=000000&amp;IS2=1&amp;bg1=FFFFFF&amp;fc1=000000&amp;lc1=0000FF&amp;t=znz-22&amp;o=9&amp;p=8&amp;l=as4&amp;m=amazon&amp;f=ifr&amp;ref=ss_til&amp;asins=4873115876" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</div>

今回は15章の前半が難しくて、あまり進みませんでした。

以下はメモです。

* p.295 の `(when list ...)` から `(car nil)` と `(cdr nil)` がエラー (例外) ではなく `nil`
* データ構造がわかりにくい
* よくわからない挿絵よりもデータ構造の図がほしい
* `defstruct` を使えば良いのに
* C言語やC++で `.` に比べて `->` の入力コストが高い
* なぜ hex (六角形) なのか
* 由来はボードゲームなどで距離の計算がしやすいかららしい
* ダイヤモンドゲームは三角形だが、辺と頂点の見方を変えれば六角形
* [Geohash](http://ja.wikipedia.org/wiki/%E3%82%B8%E3%82%AA%E3%83%8F%E3%83%83%E3%82%B7%E3%83%A5) とか [GeoDelta](http://d.hatena.ne.jp/nayutaya/20110826/1314382637) とか
- loop や format がわかりにくい
- `~a` とか printf の `%` と全く違う
- 「、。」と「，．」(全角)と「,.」(半角) という感じで全角半角も意見が割れた
- yen sign (`¥`) と backslash (`\`) の話
- p.320 選択肢にある 2 -> 3 の 2 とか 3 とかの説明がない
- choose your move: が選択肢の前にあって入力するときは行頭なのが変
- p.320 announce-winner関数のアイコンなどは [正誤表](http://practical-scheme.net/wiliki/wiliki.cgi/Shiro:LandOfLisp) に載っていた
- PDF (第3刷) では直っていた
- Ruby の末尾最適化の話
  - `RubyVM::InstructionSequence.compile_option = { :tailcall_optimization => true }` で変更できる
  - 例: [CRubyで末尾最適化を使った再帰 - yarbの日記](http://d.hatena.ne.jp/yarb/20121029/p1)
- p.333 スタックトレースが問題になる

懇親会での話の一部のメモです。

- 最近は朝とか移動中にオーディオブックを聞いているという話
- [4倍速にしている](http://qiita.com/cuzic/items/86f012bba8c17cc98aea)話
- コンテナ物語という本の話 <div style="float:right"><iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&amp;bc1=000000&amp;IS2=1&amp;bg1=FFFFFF&amp;fc1=000000&amp;lc1=0000FF&amp;t=znz-22&amp;o=9&amp;p=8&amp;l=as4&amp;m=amazon&amp;f=ifr&amp;ref=ss_til&amp;asins=4822245640" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe></div>
- ソードアート・オンラインは1巻だけでも良いのでオススメという話 <div style="float:right"><iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&amp;bc1=000000&amp;IS2=1&amp;bg1=FFFFFF&amp;fc1=000000&amp;lc1=0000FF&amp;t=znz-22&amp;o=9&amp;p=8&amp;l=as4&amp;m=amazon&amp;f=ifr&amp;ref=ss_til&amp;asins=4048677608" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe></div>
- なれる!SE は1巻には1と付いていないという話 <div style="float:right"><iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&amp;bc1=000000&amp;IS2=1&amp;bg1=FFFFFF&amp;fc1=000000&amp;lc1=0000FF&amp;t=znz-22&amp;o=9&amp;p=8&amp;l=as4&amp;m=amazon&amp;f=ifr&amp;ref=ss_til&amp;asins=4048686054" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe></div>
- [ASCII.jp：なれる！ＳＥ 間違いだらけの？ＩＴ用語辞典（中二版）](http://ascii.jp/elem/000/000/760/760389/)
- [Ruby on Rails Advent Calendar 2013](http://qiita.com/advent-calendar/2013/ruby-on-rails) のときから qiita に書くようになって、最近は[いろいろ書いている](http://qiita.com/cuzic)という話
- Ruby の id3 のライブラリが SJIS を latin-1 と解釈して UTF-8 に変換してしまって化けていたという話
- [ID3タグの文字コードは latin-1 か Unicode](http://en.wikipedia.org/wiki/ID3#cite_ref-5) ということになっているのでライブラリとしては仕様通りの挙動だが日本語圏では latin-1 として実体は SJIS を入れていることが多いため問題が生じる
- [Unicode正規化](http://homepage1.nifty.com/nomenclator/unicode/normalization.htm) の話
- Unicode 絵文字の色情報はどこからきているのかわからないという話
- OpenType とかのフォントには色情報は入らないはずなので別途どこかに持っている?
