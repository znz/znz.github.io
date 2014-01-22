---
layout: post
title: "第 7 回 RubyMotion もくもく会 in Osaka に参加した"
date: 2014-01-22 23:00:00 +0900
comments: true
categories: event ruby rubymotion osx
---
第 1 回から第 6 回にも参加した RubyMotion もくもく会 in Osaka の
[第 7 回 RubyMotion もくもく会 in Osaka](http://connpass.com/event/4560/)
に参加してきました。

次回の
[第 8 回 RubyMotion もくもく会 in Osaka](http://connpass.com/event/4910/)
は 2/19(水) になりました。

<!--more-->

## もくもく

今回は初参加の人がいました。

途中は少し質問の話があったり、リリースしたものの話があったりした以外は
基本的にみんなもくもくしていました。

## 話に出たもの

話に出てきたサイトなどのメモです。

- https://parse.com/
- http://shin1x1.github.io/vagrantx/
- https://twitter.com/ametalk
- http://orekike.com/history/orekike7
- http://www.paintcodeapp.com/

## やっていたこと

- ちゃんと名前をつけて `motion create` からやり直しました。
- いらないメニュー項目がほとんどなので、 `buildMenu` をコメントアウトしてみたら、
  Command+q での終了も出来なくなって不便になったので、戻しました。
- `open` コマンドでエラーになったときに `NSAlert` でエラーメッセージを出すようにしました。

## 引き続き不明な点

細かいものはいろいろありますが、大きなものは以下の2点です。

### URL の受け取り方

起動中に `open http://localhost:3000` のように URL を開いた場合は

```ruby
    NSAppleEventManager.sharedAppleEventManager.setEventHandler(
      self,
      andSelector: :'handleGetURLEvent:withReplyEvent:',
      forEventClass: KInternetEventClass,
      andEventID: KAEGetURL)
```

で登録しておけば受け取れていますが、
起動していない時に開かれた URL を受け取る方法がわからないままです。

`open -a Hello --args http://localhost:3000` のように起動すれば
`NSProcessInfo.processInfo.arguments` で取り出せますが、
他のアプリケーションから URL を開いた時に受け取れないので
意味がないです。

### window が閉じられた時の処理

今のままだと Command+w などで閉じられてしまうと
URL を受け取っても見せる window がなくなっているので
何も出来ないです。

window を閉じられたら終了するか、
閉じられても URL を開いたらまた出てくるようにしたいです。

## まとめ

今回はみんなもくもくとそれぞれの作業が進んだという感じで、
成果発表もすぐに終わった感じでした。
