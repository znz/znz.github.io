---
layout: post
title: "第 6 回 RubyMotion もくもく会 in Osaka に参加した"
date: 2013-12-26 23:00:00 +0900
comments: true
categories: event ruby rubymotion osx
---
第 1 回から第 5 回にも参加した RubyMotion もくもく会 in Osaka の
[第 6 回 RubyMotion もくもく会 in Osaka](http://connpass.com/event/4211/)
に参加してきました。

次回の
[第 7 回 RubyMotion もくもく会 in Osaka](http://connpass.com/event/4560/)
は 1/22(水) になりました。

<!--more-->

## 話に出たもの

話に出てきたサイトなどのメモです。

- http://shin1x1.github.io/vagrantx/
  - URL が小文字に変わっていました。
- [Cocoa Controls](https://www.cocoacontrols.com/)
- [Pocket](http://getpocket.com/)
- [Particle Designer](http://71squared.com/ja/particledesigner)
- UIMotionEffect で視差効果が簡単に実装できる
- All Cast という Android アプリで Apple TV に AirPlay できる

## やっていたこと

今回は前回作り始めていたデフォルトブラウザを乗っ取って、
Firefox とか Chrome とか Safari とかに起動し分けられる
OSX アプリの作成の続きをやっていました。

今回の最後ではスクリーンショットのようになりました。

{% img /images/2013-12-26-hello.png "screenshot" "作成途中のアプリのスクリーンショット" %}

適当に作った Hello.app に動作確認用のメソッドを付け足していただけなので、
まだタイトルは Hello のままになっています。

### URL 受け取り問題

LimeChat で URL をクリックしたときなどに受け取るのは
前回調べていた `LSSetDefaultHandlerForURLScheme` での
デフォルトブラウザ設定とあわせて、

```ruby
    NSAppleEventManager.sharedAppleEventManager.setEventHandler(
      self,
      andSelector: :'handleGetURLEvent:withReplyEvent:',
      forEventClass: KInternetEventClass,
      andEventID: KAEGetURL)
```

でイベントを受け取ると Hello.app 起動中は受け取れたのですが、
起動していない時の URL クリックのイベントを受け取れない問題は
結局解決できませんでした。

`open -a Hello --args URL`
というなら
`NSProcessInfo.processInfo.arguments`
で受け取れるのですが、
LimeChat などでの URL クリックで起動された時や
`open -a Hello URL`
という起動方法だと受け取り方がわかりませんでした。

他のブラウザの起動も
`open -a 'Google Chrome' http://localhost:4000/ --args -incognito`
のように URL は args ではない方法で渡さないと開けませんでした。

### `keyDirectObject` がない問題と前面に出てこない問題

URL の受け取り部分は今のところ、以下のようにしています。

```ruby
  def handleGetURLEvent(event, withReplyEvent: replyEvent)
    keyDirectObject = '----'.unpack('L')[0]
    urlStr = event.paramDescriptorForKeyword(keyDirectObject).stringValue
    @text.stringValue = urlStr

    Process.spawn("osascript", "-e", <<-SCRIPT)
tell Application "#{NSBundle.mainBundle.infoDictionary['CFBundleName']}"
  activate
end tell
    SCRIPT
  end
```

最初の問題として、
`keyDirectObject` が RubyMotion にはなさそうだったので、
値を調べて `unpack` で同等の値を生成するようにしました。
この部分は前回から今回までの間に調査して作成済みだったので、
詳しい調査方法は忘れてしまいましたが、
引数のオブジェクトを調べて見つけたような気がします。

前面に出てこない問題は
今回のもくもく会の間に対処しました。

前面に持ってくる方法がどういう API を呼べばよいのかわからず、
AppleScript を使った方法は情報があったので、
とりあえず `osascript` を呼び出してしまうことにしました。

最初は `spawn` ではなく `system` を使ってしまったら、
デッドロックしてしまって強制終了するしかなくなってしまったので、
`Process.spawn` を使いました。

## まとめ

今回は作りたいものが決まっていて、しっかりもくもく出来ました。

起動時に URL が受け取れない問題は
自分が使うだけなら
URL を 2 回クリックするという方法で
回避できるのですが、なんとかしたいところです。

`keyDirectObject` と `osascript` を使っているところは
他に方法がなければ
今のままでも実用上は困らないと感じました。
