---
layout: post
title: "第 5 回 RubyMotion もくもく会 in Osaka に参加した"
date: 2013-11-20 21:54
comments: true
categories: event ruby rubymotion osx
---
第 1 回から第 4 回にも参加した RubyMotion もくもく会 in Osaka の
[第 5 回 RubyMotion もくもく会 in Osaka](http://connpass.com/event/3871/)
に参加してきました。

今回も東京と同時開催でしたが、
接続などはせずに終わってしまいました。

次回の
[第 6 回 RubyMotion もくもく会 in Osaka](http://connpass.com/event/4211/)
は 12/26(木) になりました。

<!--more-->

## 話に出たもの

話に出てきたサイトなどのメモです。

- http://shin1x1.github.io/VagrantX/
  のサイトのデザインを
  https://wrapbootstrap.com/
  から選ぼうとしたが、結局 github のテンプレートのまま使うことにしたという話
- 昔は nib というバイナリだったけど今は xib という XML ファイルになっているという話
- [Rainy Mood](http://www.rainymood.com/)
  と
  [多田屋のBGM](http://tadaya.net/blog/2012/12/01)
  の組み合わせが良いという話
- アドベントカレンダーの話
  - [Adventar](http://www.adventar.org/)
  - [RubyMotion Advent Calendar 2013](http://qiita.com/advent-calendar/2013/rubymotion)
- [Img2icns](http://www.img2icnsapp.com/) の無料の方で OSX アプリのアイコンへの変換が出来そうという話
- クラウドソーシングでアイコンを募集するのはどうかという話
  - [designclue（デザインクルー） - デザインクラウドソーシング](http://www.designclue.co/)

## やっていたこと

### www.ruby-lang.org

[www.ruby-lang.org のサイトデザイン募集](https://www.ruby-lang.org/ja/news/2013/09/28/design-contest/)
の投票を webmaster でやっていて、締め切りが今日だったということで、
投票しようかと思っていたら、
締め切りが 9:00 20 Nov(JST) だったのに気付いて諦めました。
まだ集計前だったようなので、間に合いそうでしたが、
無理はしないということにしました。

ちなみに
[応募は7件](https://github.com/ruby/www.ruby-lang.org/issues?labels=contest)
でした。

### notification

最初は
https://github.com/Watson1978/notification
という OSX のサンプルをいじってみていました。

`open ./build/MacOSX-10.8-Development/notification.app`
で開くと右上の通知はそもそも出てきているのかどうかわからない状態で終了してしまって、
通知センターを開くとちゃんと送れていることが確認できたり、
引数は `-psn_なんとか` という
Emacs.app のカレントディレクトリ問題を調べていた時に見かけたものが渡ってきていたりしたのがわかりました。

### Calc

[RubyMotionSamples](https://github.com/HipByte/RubyMotionSamples)
の osx に Calc というのが追加されていたので、
それを試していました。

## デフォルトブラウザ設定

デフォルトブラウザはどこで設定しているんだろうと思って、
`defaults read`
の出力を調べたりしていたら、
`LSSetDefaultHandlerForURLScheme`
と
`LSSetDefaultRoleHandlerForContentType`
で設定できるとわかったので、
RubyMotion の中から呼び出そうとしたのですが、
`NoMethodError`
になるだけで、
結局呼び出し方がわかりませんでした。

試したこととしては `Calc` のボタンと同じように
ボタンが押された時に呼ばれるメソッドの中で、

```ruby
    bundle_id = NSBundle.mainBundle.bundleIdentifier
    LSSetDefaultHandlerForURLScheme("http", bundle_id) #~> NoMethodError
```

のように呼び出そうとしたところ、
`LSSetDefaultHandlerForURLScheme`
で
`NoMethodError`
になりました。

それから `Rakefile` の
`Motion::Project::App.setup do |app|`
のブロックで
`app.frameworks << 'ApplicationServices'`
のように
`ApplicationServices`
フレームワークを追加というのも試してみたのですが、
変化はありませんでした。

そんな感じで手詰まっていたら時間が来て終了ということになりました。

確認したバージョンは以下の通りです。

- Mac OS X 10.8.5
- Xcode 5.0.2
- RubyMotion 2.14

### 解決

`CoreServices` が必要と教えてもらったので、
`app.frameworks << 'CoreServices'`
も足してみたのですが、
`NoMethodError`
のままで動かず、
結局
RubyMotion を 2.15 に上げると直っていました。
