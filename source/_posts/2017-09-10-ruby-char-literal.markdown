---
layout: post
title: "Ruby の文字リテラルについて"
date: 2017-09-10 22:00:00 +0900
comments: true
categories: ruby
---
Ruby には `?a` で 1 文字のみの文字列を返す文字リテラルというものがあります。

Ruby を 1.9 以降から使い始めた人には `'a'` などの文字列リテラルとの使い分けや `String#chr` の存在意義などがわからないと思ったので、知っている範囲で歴史的経緯を説明してみたいと思います。

<!--more-->

## Ruby 1.8 以前と Ruby 1.9 以降の違い

マルチエンコーディング対応が入る前の文字リテラルは1バイト文字用のリテラルで、多バイト文字は使えませんでした。
コントロールやメタは昔から使えました。

以降の実行例も含めて、実行例は 2.0 以降も同じなので、省略しています。
1.8.6 以前は未確認ですが、1.8.7 とほぼ同じはずです。

```
% rbenv each ruby -ve 'p ?a'
ruby 1.8.7 (2013-12-22 patchlevel 375) [x86_64-linux]
97
ruby 1.9.3p551 (2014-11-13 revision 48407) [x86_64-linux]
"a"
% rbenv each ruby -ve 'p ?あ'
ruby 1.8.7 (2013-12-22 patchlevel 375) [x86_64-linux]
-e:1: Invalid char `\201' in expression
-e:1: Invalid char `\202' in expression
ruby 1.9.3p551 (2014-11-13 revision 48407) [x86_64-linux]
"あ"
% rbenv each ruby -ve 'p ?\C-a'
ruby 1.8.7 (2013-12-22 patchlevel 375) [x86_64-linux]
1
ruby 1.9.3p551 (2014-11-13 revision 48407) [x86_64-linux]
"\u0001"
% rbenv each ruby -ve 'p ?\M-a'
ruby 1.8.7 (2013-12-22 patchlevel 375) [x86_64-linux]
225
ruby 1.9.3p551 (2014-11-13 revision 48407) [x86_64-linux]
"\xE1"
```

## 文字とは?

文字を表すのに専用のクラスを導入するという案もあったようですが、Ruby は大クラス主義だという点や、必要とされる機能が文字列とほとんど変わらない、文字というのを文字列とは別に定義するのは難しいなどの理由から、最終的には1文字だけの String で文字を表すことになったと記憶しています。
詳細はメーリングリストや redmine の issue の議論などを探してみてください。

## getc と文字リテラル

文字リテラルの用途として CUI アプリなどで `getc` で入力した文字との比較という使われ方があったようで、 `getc` などを使っているプログラムが、文字リテラルと比較する書き方をしていれば壊れないようになっています。

```
% rbenv each ruby -ve 'p ARGF.getc' /dev/zero
ruby 1.8.7 (2013-12-22 patchlevel 375) [x86_64-linux]
0
ruby 1.9.3p551 (2014-11-13 revision 48407) [x86_64-linux]
"\u0000"
% rbenv each ruby -ve 'p ARGF.getc == ?\C-@' /dev/zero
ruby 1.8.7 (2013-12-22 patchlevel 375) [x86_64-linux]
true
ruby 1.9.3p551 (2014-11-13 revision 48407) [x86_64-linux]
true
```

## `String#chr`

[String#chr](https://docs.ruby-lang.org/ja/latest/method/String/i/chr.html) は単独で見ると `str[0]` などで代用できるので不要そうなメソッドですが、
文字リテラルや `getc` の返り値を文字列にするのに `chr` が使われていたので、
`String#chr` も互換性のために存在します。

```
% rbenv each ruby -ve 'p ?a.chr'
ruby 1.8.7 (2013-12-22 patchlevel 375) [x86_64-linux]
"a"
ruby 1.9.3p551 (2014-11-13 revision 48407) [x86_64-linux]
"a"
% rbenv each ruby -ve 'p ARGF.read(1) == ?\C-@.chr' /dev/zero
ruby 1.8.7 (2013-12-22 patchlevel 375) [x86_64-linux]
true
ruby 1.9.3p551 (2014-11-13 revision 48407) [x86_64-linux]
true
```

## ord

逆に `ord` は `str[0]` の返り値が文字列に変わってしまったので、常に数値が欲しい時にも使っていました。

```
% rbenv each ruby -ve 'p ?\C-@.ord'
ruby 1.8.7 (2013-12-22 patchlevel 375) [x86_64-linux]
0
ruby 1.9.3p551 (2014-11-13 revision 48407) [x86_64-linux]
0
% rbenv each ruby -ve 'p "a"[0]'
ruby 1.8.7 (2013-12-22 patchlevel 375) [x86_64-linux]
97
ruby 1.9.3p551 (2014-11-13 revision 48407) [x86_64-linux]
"a"
% rbenv each ruby -ve 'p "a"[0].ord'
ruby 1.8.7 (2013-12-22 patchlevel 375) [x86_64-linux]
97
ruby 1.9.3p551 (2014-11-13 revision 48407) [x86_64-linux]
97
```

## 結論

新規に書くプログラムで積極的に文字リテラル ( `?a` ) を使う必要性はほとんどないので、普通は文字列リテラルだけ使っておけば良いと思います。

`getc` などとの組み合わせのときに文字リテラルを使えば意味を明確にできますが、今更 1.8 以前との互換性を気にすることもないと思うので、古いプログラムで使われていたときに読めればいいだけで、書くときに使う必要はないと思います。
