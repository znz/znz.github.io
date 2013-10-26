---
layout: post
title: "第8回 関西Emacs勉強会に参加しました"
date: 2013-10-26 14:19
comments: true
categories: event emacs
---
[(kansai-emacs #x08) #関西Emacs : ATND](http://atnd.org/events/44155)
というイベントに参加してきました。

<!--more-->

## ポジションペーパー

以前作りかけで一部は
[関西Ruby会議05](http://regional.rubykaigi.org/kansai05)
の時に使った rabbit での自己紹介のスライドに
Emacs 関係の情報を追加して作った PDF だったので、
1ページにできずに複数ページになってしまいました。

会場への移動中に作成したのですが、
作成前に既にアップロードされているものを確認したら
複数ページのものがあるので、
まあいいかと思ったのも理由でした。

## 会場到着から開始まで

到着時点では知っている人がいなくて、
どうすればいいのかよくわからなかったので、
入り口近くで座って twitter とか見ながら待っていました。
13時を少しすぎたところで準備ができたらしく案内があったので、
席に移動しました。

## 発表

`package-install` だと
環境を作り直した時に magit とかがバージョンアップで挙動が変わって
困ることがあるのでどうするのが良いのかという相談とか、
`ac-mozc` の紹介とか、ちゃんとメモをとっていなかったのですが、
いろいろありました。

`ac-mozc` の質疑応答では

  * 英単語に続けて日本語を入力する時はスペースを入れて入力して、確定するとスペースが消える、スペースが必要な時はその直後に undo すればいい
  * 英単語を入力していてローマ字ともみなせて不要なのに ac-mozc の候補が出てきたときは ac のキャンセルと同じように C-g で消せばいい
  * コメントの中とか文字列の中とかで一部だけ変換したい時とかは変換用のコマンドを直接呼び出せばいい

のような話があったと記憶しています。


## やっていたこと

### package.el

[package.el - Emacs JP](http://emacs-jp.github.io/packages/package-management/package-el.html)
を参考にして `package.el` を使う設定をしました。

### shell-pop

上のメモには書いていませんが、発表にあった `shell-pop` を
`package.el` を使って入れてみました。
昔ちょっと試したことがあって、その設定が悪さをしていて、
最初は
`(void-function shell-pop-set-internal-mode)`
というエラーになってしまっていましたが、
古い設定を削除すれば直りました。

[EmacsWiki の Shell Pop](http://www.emacswiki.org/emacs/ShellPop)
には上の方に小さく書いてあるだけですが、
github の方が新しくて互換性がなくなっているのが理由でした。

### ansi-term

`shell-pop` の設定例にある `ansi-term` を使ってみると
最下行で文字を入力すると一瞬上にスクロールして戻るという現象が出て、
`ansi-term` 単体でもおきたので、
`ansi-term` の問題かと思ってクリーンな環境でも試してみると発生しませんでした。

scroll 関係の設定があやしいと思って、
`scroll-margin` を `2` にしていたのを `0` にしたらなおったのですが、
クリーンな環境で `scroll-margin` だけ変更しても現象が出なかったので、
結局何が再現条件なのかはわかりませんでした。
`scroll-step` を `1` にしているのも関係しているかと思ったのですが、
関係ありませんでした。

### elscreen

`auto-install` でインストールしていた `elscreen.el` を
`package.el` でインストールしたものに変更しました。

`(elscreen-start)` が必要になっているというのを知らなかったので、
うまく動かなくてかなり悩んでいましたが、教えてもらって解決しました。

### anything.el から helm への変更

`anything.el` は一時期はいろいろ設定を頑張っていたのですが、
結局デフォルトの `anything` 以外はほとんど使っていなかったので、
`helm-mini` に置き換えるだけで問題なさそうな感じでした。

`anything.el` だとメニューに登録されていたり
`(anything-execute-anything-command)` とか
`(anything-call-source)` のようなメタ機能があったりして、
存在を知らない機能でも探しやすいのですが、
`helm` にはそういうものがあるのかどうかわかりませんでした。

ちなみに `M-x` で `helm-` で始まる `interactive` な関数を探すのは
`helm` 関係のバッファでのキーバインドようの関数が混ざっているので
この用途には使えません。
