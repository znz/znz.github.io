---
layout: post
title: "Land of Lisp 読書会 第２回(兵庫県)に参加してきた"
date: 2013-09-21 23:00
comments: true
categories: event amagasakirb land-of-lisp
---
[amagasakirb](http://kokucheese.com/main/tag/amagasakirb)
の
[9月21日 Land of Lisp 読書会 第２回(兵庫県)](http://kokucheese.com/event/index/105390/)
に参加してきました。
今回は6.5章から9章まででした。

<!--more-->

以下は単なるメモです。

* p.97 訳注 の 保守的な言語 とは何か?
* 太い数字は脚注ではなく行番号というのがわかりにくい。
  どこかに説明はあった?
* p.115 ダイナミック変数とは?
  * 別の本によると `defなんとか` で定義されるのがダイナミック変数らしい。
  * `defun` もダイナミックスコープっぽい。
    変数の `let` のように `labels` で関数も置き換えられる。
* なぜか Ruby の `$_` の話
* 話がそれていって開発環境の話から `VisualAge` とかの話もあった。
* p.109 に not で終わっているLisp関数は非推奨と書いてあるのに、
  p.130 以降のプログラムでは `remove-if-not` を使いまくっている。
* p.130 の `islands` はどういうデータが入っているのか?
* p.134 `コンス` とは? `コンスセル` のこと。
* カヌレ
* 9章を読み終わっていない人は宿題 (次回までに読んでくる)
