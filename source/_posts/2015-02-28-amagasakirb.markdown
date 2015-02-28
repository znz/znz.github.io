---
layout: post
title: "アンダースタンディング コンピュテーション読書会　第1回(兵庫県)に参加しました"
date: 2015-02-28 13:00:40 +0900
comments: true
categories: event amagasakirb ruby
---
[2月28日 アンダースタンディング コンピュテーション読書会　第1回(兵庫県)](http://kokucheese.com/event/index/258232/ "2月28日 アンダースタンディング コンピュテーション読書会　第1回(兵庫県)")
に参加しました。

<!--more-->

## メモ

<div style="float:right">
<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&amp;bc1=000000&amp;IS2=1&amp;bg1=FFFFFF&amp;fc1=000000&amp;lc1=0000FF&amp;t=znz-22&amp;o=9&amp;p=8&amp;l=as4&amp;m=amazon&amp;f=ifr&amp;ref=ss_til&amp;asins=487311697X" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</div>

以下、今回のメモです。

- [2015.02.28 アンダースタンディングコンピュテーション 読書会 第1回 · cuzic/amagasakirb Wiki](https://github.com/cuzic/amagasakirb/wiki/2015.02.28-%E3%82%A2%E3%83%B3%E3%83%80%E3%83%BC%E3%82%B9%E3%82%BF%E3%83%B3%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0%E3%82%B3%E3%83%B3%E3%83%94%E3%83%A5%E3%83%86%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3-%E8%AA%AD%E6%9B%B8%E4%BC%9A-%E7%AC%AC1%E5%9B%9E "2015.02.28 アンダースタンディングコンピュテーション 読書会 第1回 · cuzic/amagasakirb Wiki")
- `->` が多い
- `->` の後に括弧を書くかどうか (書く方が多数派だった)
- `.call()` の `call` の省略を使うかどうか (省略しない方が多数派だった)
- 多重代入の右辺の `[]` でくくって配列にしているのはあってもなくてもこの例だと同じ
- `"#{obj}"` は `#to_s` が呼ばれる
- `"abc" + obj` は `#to_str` が呼ばれる
- `String(obj)` も `#to_str` が呼ばれる
- p.9 `*演算子` の演算子という表現が引っかかる
- `Symbol#to_proc` と `Object#tap` は Rails (ActiveSupport 由来)
- `class Point < Struct.new(:x, :y)` の話
- 名前が決まらなくて入らないメソッドがある話
- 名前が決まって最近入った例としては `itself` (ruby 2.2 から)
- p.19 「Ruby 1.8.7 には文書としての仕様書が存在しており、ISO標準として受理されています (ISO/IEC 30170)」とあるが更新されることはないのかという話
- p.21 の図が難しい
- p.23 二重山括弧
- Option + Shift + K で入力できる Apple 記号の話
- p.51 の `DoNothing` がほぼ `itself`
- 簡約可能という言葉の意味が分かっていないとつらいのではないかという話 (簡約可能とはまだ処理することが残っていることで簡約不可能とはもう処理することが残っていない状態という話)
- Treetop, Parslet (オススメ), ANTLR
- Node.js, io.js, JXcore
- p.37 `DoNothing.new` の `.new` はあってもなくても良いのではないかという話
- [『アンダースタンディング コンピュテーション』のサポートリポジトリ](https://github.com/ko1/uc_ja "『アンダースタンディング コンピュテーション』のサポートリポジトリ")
- yacc やコンパイラコンパイラの話
- ブログの引っ越し先の話
- markdown の方言の話
- 次回は 4/4(土) に小田公民館の会議室が確保できなかったという理由で別の場所で。
