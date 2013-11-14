---
layout: post
title: "Mac OS XでのIME制御についての考察"
date: 2013-11-14 02:24
comments: true
categories: mac emacs tty ime
---
[Cocoa Emacs のインラインパッチ関連の設定](/blog/2013-11-12-cocoa-emacs-ime.html)
の時や
[iTerm2 で IME 制御シーケンスがきかないのを調べた](/blog/2013-11-13-tty-ime.html)
時にも気になったのですが、
Mac の IME に open/close という概念はあるのかという話です。

<!--more-->

## IME の開閉状態?

iBus 1.5 関連の記事の
[iBusがクソになった理由 — KaoriYa](http://www.kaoriya.net/blog/2013/10/18/)
で知ったのですが、
Mac OS X は IME のオン・オフのような状態の切り替えではなく、
「ことえりのひらがな」とか
「ことえりのカタカナ」とか
「ことえりの英字」とか
のようなIMEの種類を切り替えて入力するようになっています。

この考え方と開閉状態しか考慮していない二値的な制御シーケンスは相性が悪いのではないかと思いました。

## Cocoa Emacs のインラインパッチでは?

Cocoa Emacs のインラインパッチをよく見ると
`mac-toggle-input-source`
という関数があって、
`(mac-toggle-input-source nil)`
で IME オフ相当に、
`(mac-toggle-input-source t)`
で IME オン相当にできるようなので、
対応は不可能ではないのかもしれません。

## 端末の制御シーケンスでは?

[TeraTerm Pro の対応制御シーケンス](http://ttssh2.sourceforge.jp/manual/ja/about/ctrlseq.html)
には以下の3種類の IME 関連の制御シーケンスがあります。

```
  CSI < r    TTIMERSIME の開閉状態を復元する。
  CSI < s    TTIMESVIME の開閉状態を保存する。
  CSI < Ps t TTIMESTIME の開閉状態を設定する。省略時の Ps の値は 0。
               Ps = 0      IME を閉じる。
                  = 1      IME を開く。
```

この中の `TTIMESTIME` の `Ps` を拡張して、
2 以上も受け付けるようにして、
(別の状態が不可能なら 1 と同じ挙動で)
可能ならカタカナ入力などの別の IME の状態を設定できるようにするという案を思いつきました。
