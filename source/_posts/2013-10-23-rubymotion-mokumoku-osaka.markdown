---
layout: post
title: "第 4 回 RubyMotion もくもく会 in Osaka に参加した"
date: 2013-10-23 19:30
comments: true
categories: event ruby rubymotion
---
第 1 回から第 3 回にも参加した RubyMotion もくもく会 in Osaka の
[第 4 回 RubyMotion もくもく会 in Osaka](http://connpass.com/event/3557/)
に参加してきました。

今回も前回と同じく東京と同時開催でした。
前回はハングアウトで接続していたのですが、
今回は接続せずに終わってしまいました。

<!--more-->

## 話に出たもの

話に出てきたアプリやサイトのメモです。

* [Reflector - AirPlay mirror your iPhone or iPad to any Mac or PC](http://www.airsquirrels.com/reflectorc/)
  * Mac OS X が Apple TV の代わりに iOS の画面を受け取れるようになる (有償 $12.99)。
  * 最後の成果発表の時に、複数の画面を同時に受け取れるという発見がありました。
  * それを使えば複数の画面をまとめて同時に Apple TV に出せました。
  * 同じ会社が作っている
    [AirParrot - AirPlay your Mac or PC's screen to Apple TV](http://www.airsquirrels.com/airparrot/)
    を使えば Windows PC からでも Apple TV に画面を飛ばせるという話もありました。
* [POP - Prototyping on Paper | iPhone App Prototyping Made Easy](https://popapp.in/)
  ペーパープロトタイピングで UI の設計 (無償)
* [Flinto – iPhone and iPad Prototyping](https://www.flinto.com/)
  こちらもペーパープロトタイピングでブラウザ上でも見えます (有償)。
  画像のアップロードが面倒なので POP の方を使っているという話でした。
* [ASCIIwwdc](http://asciiwwdc.com/)
  WWDC をテキストにおこしたサイト
* [Estimote Beacons — real world context for your apps](http://estimote.com/)
  iBeacon 関連のハードウェア
* [Rainy Mood](http://www.rainymood.com/)
  落ち着く BGM
* [第 5 回 RubyMotion もくもく会 in Osaka - connpass](http://connpass.com/event/3871/)
  次回は 2013/11/20(水) 19:30 〜 21:30
  (ハッシュタグが設定されていない?)
* [第15回 RubyMotion もくもく会 in Tokyo - connpass](http://connpass.com/event/3870/)
  東京の次回
  (ハッシュタグが設定されているから参加しますツイートにハッシュタグがついている?)
* [Sketch | The designer’s toolbox](http://www.bohemiancoding.com/sketch/)
  iOS アプリでアイコンとか作るのに使っているという話
* [Peatix | 簡単イベント管理、チケット販売サイト（ピーティックス） | Peatix](http://peatix.com/)
  手数料が安いという話

## やっていたこと

前回の続きで
https://github.com/HipByte/RubyMotionSamples
のサンプルを試していたのですが、
https://github.com/HipByte/RubyMotionSamples/tree/master/osx/Tweets
が Twitter API 1.1 への変更の影響で動かなくなっていました。

具体的には

```ruby
        begin
          json = JSONParser.parse_from_url(url)
        rescue RuntimeError => e
          presentError e.message
        end
```

という部分で

```
Terminating app due to uncaught exception 'TypeError', reason: 'data_parser.rb:6:in `parse:': exception class/object expected (TypeError)
	from json_parser.rb:3:in `parse_from_url:'
	from app_delegate.rb:71:in `block in search:'
```

というエラーになっていました。

ブラウザで
`http://search.twitter.com/search.json?q=xcode%20crash`
にアクセスしてみると
`{"errors": [{"message": "The Twitter REST API v1 is no longer active. Please migrate to API v1.1. https://dev.twitter.com/docs/api/1.1/overview.", "code": 68}]}`
というエラーが
`HTTP/1.1 410 Gone`
で返ってきていました。

API 1.1 に対応するには OAuth 対応が必須ということで、
簡単には対処できなさそうでしたが、
https://github.com/HipByte/RubyMotionSamples/issues/31
に報告がすでにあったのでコメントをつけておきました。

それとは別に、エラー処理も `RuntimeError` しか `rescue` していないので、
`TypeError` は `rescue` できていなかったり、
`rescue` できても

```
Terminating app due to uncaught exception 'NoMethodError', reason: 'app_delegate.rb:73:in `block in search:': undefined method `presentError' for #<AppDelegate:0x7fe1eb107910 ...> (NoMethodError)
```

でエラー処理がちゃんと動いていなかったりしました。

これは `NSApplication.sharedApplication.presentError error_ptr` にすれば良いのかも、
と思って試してみたけどうまくいかない、というところで時間が来て終了になりました。
