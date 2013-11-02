---
layout: post
title: "Land of Lisp 読書会 第3回(兵庫県)に参加してきた"
date: 2013-11-02 23:00
comments: true
categories: event amagasakirb land-of-lisp
---
[amagasakirb](http://kokucheese.com/main/tag/amagasakirb)
の
[11月2日 Ｌａｎｄ　ｏｆ　Ｌｉｓｐ 読書会　第3回(兵庫県)](http://kokucheese.com/event/index/105390/)
に参加してきました。
今回は第III部でした。

次回は会場の予約が取れなかったということで、12月は無くて、
1月第2週以降の予定で第一候補は 2014年1月11日ということでした。

<!--more-->

今回は他のイベントとも日程が重なっていたらしく、
それが原因かどうかはわかりませんが、
Lisper の人がいなかったので、
Lisp についての疑問は割と解決出来ない感じで進みました。
そういうこともあってか、結構雑談とか Ruby の話が多かったです。

以下はメモです。

* p.188 の挿絵の意味がよくわからない?
* `loop` の `for` が複数あるときに同時に動く (Ruby の `Array#zip` のように) のが気持ち悪い
  * 別々に動くのは `loop` のネストで代用できるからこういう仕様なのでは。
* 網膜に焼き付ける話から Google Glass の話
* `(setf アクセサ 値)` というのが気持ち悪いという話
  * p.197 の `(setf (gethash ...) t)` とか
  * 左辺値と右辺値の見た目が同じ言語は多い
  * シェルスクリプトは x=値 と $x で違う
* 話には出しませんでしたが `*plants*` の値は `t` の代わりに `plant-energy` にするのもありかもと思いました。
* 変数名とかの名前の付け方がよくわからない。
  * p.202 `xnu` とか。
* ジャングルは add-plants で必ず植物が発生するので
  植物が発生する確率が高い領域というだけで
  最初から植物がたくさんあるわけではない。
* loop はミニ言語
  * Perl の正規表現
  * Python のリスト内包表記
  * scala のリスト内包表記
* 独自言語の話
  * `printf` の書式文字列
  * C++ のマニピュレータは難しすぎるという話
  * Ruby の `#{}` は独自言語というほどのものではない
  * pack / unpack のテンプレート (これも Perl 由来なので、 Ruby 独自ではない)
* p.216 チルド文字
  * tilde なので変というわけではないがチルダの方がよく見かける
  * チルドの方が元の英語の発音に近い?
* `@` は `printf` の `%-10d` のような `-` とフラグの有無での左右が逆
* Ruby で
  * `%d` とか `%f` 以外はあまり使わない?
  * ljust とか rjust とかあるから `%s` と数字の組み合わせはあまり使わない?
  * `"%7.2f"` は他の方法では難しそう
* `'` の必要性の話
  * `'a` とかを指定出来るようにするため?
  * `'` で埋めたい場合は `''` とか
* `format` の方が `loop` にあった周期表のような表が欲しい
* `printf` の書式指定文字列に別の文字で埋める指定はできるか?
  * ljust や rjust ならできる (例: `"hoge".rjust(10, "!")`)
  - `printf` は不明 (できなさそう?)
  - `%{}d` とか `%()` とかの話
* `~$` は日本円だとあまり使わなさそう
  * アメリカはクオーターとかダイムとか (日本人には) わかりにくい
* `(fresh-line)` が便利なときの例は?
  * Ruby は `puts` が便利だから `fresh-line` のような機能の必要性が低い?
* p.222 `~t` 相当は Ruby にはなさそう
  * pack テンプレートなら `@` で似たことはできそう
* pack テンプレートの代わりに C の構造体をパースするものがあればわかりやすい?
* 「33文字分出力したら改行してくれ」の 33 とか数えたくない
* p.228 のゲームは実際に実行しても `-` の繰り返しの上下の枠線の行はない
* 名前がよくない?
  * prin1 とか
* 他の言語でも printf, strstr, stdio
* cout の読み方とか
* 13.5章は表紙ページがない
* p.233 挿絵はストリームが邪悪な感じというのを表している?
* p.239 「残念ながら、ソケットの標準化は ANSI Common Lisp の仕様化に間に合わなかったので、ソケットを扱う標準の方法というのはない。」
  * [Common Lisp - Wikipedia](http://ja.wikipedia.org/wiki/Common_Lisp) によると「1984年、1994年にANSIにより標準化」
* p.248 REPL にエラープロンプトを表示するのは REPL から実行した時だけ
* repl に戻る話から Ruby の話
  * `better_errors`
  * `binding_of_caller`
  * `ppp`
* Web 関連の雑談
  * Active Scaffold が IE10 で変?
  * CoffeeScript
  * ClojureScript
  * ブラウザで Ruby
  * ActiveScriptRuby
  * NougakuDo
  * Google の bot は JavaScript や CSS も解釈する
* p.250 q値
* car cadr caddr cdddr nth nthcdr
