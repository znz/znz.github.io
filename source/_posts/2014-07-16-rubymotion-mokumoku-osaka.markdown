---
layout: post
title: "第 13 回 RubyMotion もくもく会 in Osaka に参加した"
date: 2014-07-16 19:33:35 +0900
comments: true
categories: event ruby rubymotion osx
---

第 1 回から第 12 回にも参加した RubyMotion もくもく会 in Osaka の
[第 13 回 RubyMotion もくもく会 in Osaka](http://rubymotionjp.connpass.com/event/7079/)
に参加してきました。
今日はほぼ途中の会話はなくて、みんなでもくもくしていました。
その代わり、最後の成果発表の時は盛り上がっていました。

次回の
[第 14 回 RubyMotion もくもく会 in Osaka](http://rubymotionjp.connpass.com/event/7583/ "第 14 回 RubyMotion もくもく会 in Osaka")
は 08/20(水) になりました。

<!--more-->

## メモ

- AppCode
- TapTheCircle
- motion-osx-cli
- [RubyMotionの気持ちいいところ // Speaker Deck](https://speakerdeck.com/iwazer/rubymotionfalseqi-chi-tiiitokoro "RubyMotionの気持ちいいところ // Speaker Deck")
- RubyMotion + IB で参考にしたもの
  - [RubyMotion(MacOS)でIBの作成方法 - たまたんのぶろぐ](http://tama.hatenablog.jp/entry/2014/05/02/231633 "RubyMotion(MacOS)でIBの作成方法 - たまたんのぶろぐ")
  - [RubyMotionアプリでStoryboardとIBのアウトレット+アクションを使う話 - laiso+iphone](http://d.hatena.ne.jp/laiso+iphone/20130510/1368201914 "RubyMotionアプリでStoryboardとIBのアウトレット+アクションを使う話 - laiso+iphone")
- https://github.com/omoon/rm-test-test
  - travis-ci で RubyMotion のテストも動かせるという話
  - `bundle exec rake spec osx=true` にすれば OSX も動く。
  - 参考: https://github.com/rubymotion/BubbleWrap/blob/master/.travis.yml
- https://github.com/mattsears/nyan-cat-formatter

## 今日の成果

情報を探してみたら `rake ib` で Interface Builder を起動すれば stub を自動生成してくれて、
ボタンクリックとアクションのひも付けができるとわかったので、
ボタンをクリックした時にメソッド呼び出しはできるようになりました。

しかし、 `ib_outlet` で宣言したプロパティに outlet でひも付けしたインスタンス変数が `nil` のままで悩んでいたところ、
最後に `outlet` に変えると動くようになりました。
`ib_outlet` のままでも Interface Builder で出てくるので使えるのかと勘違いしていました。

家に帰ってから
`$(find /System/Library/Frameworks -name lsregister) -kill -r -domain local -domain user`
で既存の起動の関連付けを削除して、
ib 版で `Set Default Browser` をやり直したところ、
まだ問題があったので、それも直して以前のものと同様に使えるところまでは出来ました。
