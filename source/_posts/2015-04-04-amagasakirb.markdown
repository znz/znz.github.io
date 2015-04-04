---
layout: post
title: "4月4日 アンダースタンディング コンピュテーション読書会 第2回 (大阪府)に参加しました"
date: 2015-04-04 12:50:16 +0900
comments: true
categories: event amagasakirb ruby
---
[4月4日 アンダースタンディング コンピュテーション読書会　第2回 (大阪府)](http://kokucheese.com/event/index/276001/ "4月4日 アンダースタンディング コンピュテーション読書会　第2回 (大阪府)")
に参加しました。
今回はいつもと違う会場でした。

<!--more-->

## メモ

<div style="float:right">
<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&amp;bc1=000000&amp;IS2=1&amp;bg1=FFFFFF&amp;fc1=000000&amp;lc1=0000FF&amp;t=znz-22&amp;o=9&amp;p=8&amp;l=as4&amp;m=amazon&amp;f=ifr&amp;ref=ss_til&amp;asins=487311697X" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</div>

以下、今回のメモです。
個人的なメモと話に出ていたことのメモが混ざっています。

- [2015.04.04 アンダースタンディングコンピュテーション 読書会 第2回 · cuzic/amagasakirb Wiki](https://github.com/cuzic/amagasakirb/wiki/2015.04.04-%E3%82%A2%E3%83%B3%E3%83%80%E3%83%BC%E3%82%B9%E3%82%BF%E3%83%B3%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0%E3%82%B3%E3%83%B3%E3%83%94%E3%83%A5%E3%83%86%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3-%E8%AA%AD%E6%9B%B8%E4%BC%9A-%E7%AC%AC2%E5%9B%9E "2015.04.04 アンダースタンディングコンピュテーション 読書会 第2回 · cuzic/amagasakirb Wiki")
- [#amagasakirbに関するツイート](https://twitter.com/hashtag/amagasakirb "#amagasakirbに関するツイート")
- https://github.com/tomstuart/computationbook にサンプルコードがある
- p.41 「いろいろな意味で、これはスモールステップというアプローチよりも自然に感じられますが、」の自然に感じられる理由は? という話。
- 軽く自己紹介
- p.46 「この違いは、それぞれのアプローチの目的を際立たせます。」の目的とは? という話。
- Google マップで会場の検索をすると位置がずれているのは申請で修正できるという話。
- p.69 `string.chars.each` は一度配列を作る `chars` よりも `each_char` の方が良いのでは。
- p.73 `flat_map` をなぜ使っているのかという話。 `follow_rules_for` が配列を返すので、入れ子の階層を一段階減らすため。
- 有限オートマトンの有限とは? という話。
  とりうる状態が有限。
  状態が無限のものは無限オートマトンとか次の章で出てくるプッシュダウンオートマトンとか。
- p.82 `precedence` とは何かという話。 p.81 の `bracket` で使われている結合の優先順位。
- 3 章のサンプルコードは `the_simplest_computers` にあって最終状態のクラスのコードが入っている。実行結果の例もついているものは `irb.txt` に書いてある。
- p.90 脚注の「ここでは新しい状態を追加する代わりに、元の開始状態を受理状態にするだけで済ませることもできましたが、そのやり方だと複雑な場合に(たとえば、`(a*b*)*`)、空文字列以外の望まない文字列まで受理する機械ができてしまします。」で受理されてしまう例は?
  - `Repeat#to_nfa_design` の `start_state = Object.new` を `start_state = pattern_nfa_design.start_state` に変更して `pattern = Repeat.new(Concatenate.new(Repeat.new(Literal.new('a')), Literal.new('b')))` (`/(a*b)*/`) で試してみたら `'a'` も受理された。(`pattern.matches?('a')` が true になった。)
- p.92 http://patshaughnessy.net/2012/4/3/exploring-rubys-regular-expression-algorithm の URL が長い。
- わからないところは飛ばしつつ 3,4,5 章を読んで 3 章に戻った方がわかりやすいかもしれないという話。
- 具体例が正規表現しかないのがわかりにくいかもしれないという話。
- マクドナルドなどで注文した結果、一番安いセットが自動で選ばれるようになっているらしいという話。その実装を有限オートマトンでやっているだろうという話。
- オートマトン研究は昔の話なので、テープなど言葉が違う (古い) のは仕方がないという話。
- p.107, p.108 で `\g` で括弧の対応関係などの難しい話が載っているという話。
- `Struct.new` を継承する必要はあるのかと思っていたが p.116 の `current_configuration` の `super` のようなに上書きができるという利点があるとわかった。
- `tap` メソッドがよく出てくるという話。
- ブロック引数と外側の変数の名前が重なっていたときにどうなるかが 1.9 で変わった話。
- `Kernel#returning` というのが昔の ActiveSupport にあったという話。
`returning {} do |hash| ... end` は `{}.tap do |hash| ... end` と同じ意味。
- C++ の右辺値参照の話。
  [C++11 - シンプルな配列クラスを使って「右辺値参照」と「ムーブセマンティクス」を知る - Qiita](http://qiita.com/go_astrayer/items/5d85565e992487daa618 "C++11 - シンプルな配列クラスを使って「右辺値参照」と「ムーブセマンティクス」を知る - Qiita")
- C++11 で記号だらけという話。
- C++ のラムダ式 (`[&]{...}` とか) の話。
- C# も記号を使った記法が増えているらしいという話。
- `Object.new` は普通に使われているのかという話。
- p.127 `LexicalAnalyzer` は文字列の先頭や末尾に空白があるとうまく動かなさそうと思って試してみたら、末尾は `lstrip` で消えるので大丈夫だった。
  先頭は ```NoMethodError: undefined method `post_match' for nil:NilClass``` になった。
- p.128 `/(true|false)(?![a-z])/` は `/(true|false)\b/` でも良さそう。
- 計算機の歴史の話。
- オクテットの話。
- 等価性と非等価性の話。
- DFA=NFA (正規表現), DPDA (括弧の対応), NPDA (回文), チューリングマシン (`aaabbbccc` など)
