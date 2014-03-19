---
layout: post
title: "第 9 回 RubyMotion もくもく会 in Osaka に参加した"
date: 2014-03-19 19:32:08 +0900
comments: true
categories: event ruby rubymotion osx
---
第 1 回から第 8 回にも参加した RubyMotion もくもく会 in Osaka の
[第 9 回 RubyMotion もくもく会 in Osaka](http://connpass.com/event/5276/)
に参加してきました。

次回の
[第 10 回 RubyMotion もくもく会 in Osaka](http://connpass.com/event/5674/)
は 4/23(水) になりました。

<!--more-->

## メモ

- [VagrantX のロゴ](https://github.com/shin1x1/vagrantx/issues/1)
- [RubyMine](http://www.jetbrains.com/ruby/)
- [TestFlight](http://testflightapp.com/) の代わりに今は [DeployGate](https://deploygate.com/) が良いという話

## やったこと

RubyMine をインストールして使い始めてみました。
試用期間は次回のもくもく会には切れているので、
それまでに他でも使っておかないとあっという間に試用期間が終わってしまいそうです。

[イベントハンドラの設定を init でやれば良い](http://stackoverflow.com/questions/9289613/handlegeturlevent-doesnt-get-called)というような話があったので、
init メソッドを定義してみたところ、呼ばれていなかったので、
initialize にしてみたら、
呼ばれていることは確認できても、
やっぱり起動時の URL は受け取れないままでした。

内部的な変更としては
REPL で確認した時に
`KeyDirectObject` が存在しなかったので、
`keyDirectObject = '----'.unpack('L')[0]`
のように作っていたのを
`KeyDirectObject`
を使うように変えました。

以前のもくもく会の時に気付いたように
`rake`
でのコンパイル時に参照していない定数は REPL でも存在しないというだけでした。

他にはいろいろ調べている時に
[ウィンドウのクローズとアプリケーションの終了を同期させる](http://vivacocoa.jp/objective-c3e/chapter5c.html)
というのを知ったので、
以下のメソッド定義を追加して
window を閉じた時に終了するようにしました。
これを追加するまでは閉じたら開き直す手段がなくて何も出来なくなっていました。

```ruby app/app_delegate.rb
  def applicationShouldTerminateAfterLastWindowClosed(application)
    true
  end
```
