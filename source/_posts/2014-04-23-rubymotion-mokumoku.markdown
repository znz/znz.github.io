---
layout: post
title: "第 10 回 RubyMotion もくもく会 in Osaka に参加した"
date: 2014-04-23 19:16:51 +0900
comments: true
categories: event ruby rubymotion osx
---
第 1 回から第 9 回にも参加した RubyMotion もくもく会 in Osaka の
[第 10 回 RubyMotion もくもく会 in Osaka](http://connpass.com/event/5674/)
に参加してきました。

次回の
[第 11 回 RubyMotion もくもく会 in Osaka - connpass](http://connpass.com/event/6116/)
は 05/28(水) になりました。

<!--more-->

## メモ

- [ボコスカ！モグランド](https://itunes.apple.com/jp/app/bokosuka!mogurando/id845835380?mt=8)
- <a href="http://www.amazon.co.jp/gp/product/4873116678/ref=as_li_ss_tl?ie=UTF8&amp;camp=247&amp;creative=7399&amp;creativeASIN=4873116678&amp;linkCode=as2&amp;tag=znz-22">AngularJSアプリケーション開発ガイド</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=znz-22&amp;l=as2&amp;o=9&amp;a=4873116678" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" /> が発売されているという話
- [Pixate](http://www.pixate.com/)
- MacFace
- [osx/Tweets app crashes on launch](https://github.com/HipByte/RubyMotionSamples/issues/31)
- http://www.impress-plus.co.jp/

## 調査続き

引き続き初回起動時の URL の受け取り方を調べていました。

- [「ログイン時に起動」を実装する](http://questbeat.hatenablog.jp/entry/2014/04/19/123207) はあんまり関係なかった。
- GitHub で `LSSetDefaultHandlerForURLScheme` を検索
  - https://github.com/s-faychatelard/Helm/blob/2c85b303781fa430432c17ed9848a0b252c82eae/sources/others/mac/mac.mm : Qt で実装していて URL を受け取っている部分がわからず (Qt のイベントに変換されたものを受け取っている?)
  - https://github.com/bigbearlabs/MotionKit/blob/51f46c14b52819b709c57d6e1c954d84c852e48b/lib/motion-kit/misc/osx/osx_browsers.rb : ブラウザーの一覧の取得など
  - https://github.com/bigbearlabs/MotionKit/blob/51f46c14b52819b709c57d6e1c954d84c852e48b/lib/motion-kit/app/osx/app.rb : `application:openFile:` や `application:openFiles:` で受け取っているように見えるが試してみても呼ばれない。
  - https://github.com/gnyrd/WebEx/blob/0ea21241591e496d1d702c45768400fa14d75662/URLHandler.m : `kAEGetURL` の登録に `handleGetURLEvent:withReplyEvent:` (2引数) ではなく `getURL:` (1引数) を使っている。

前回までの試行錯誤の途中で `setEventHandler` の呼び出しを `applicationDidFinishLaunching:` から `initialize` に移動していたのと `getURL:` への変更の組み合わせで、やっと初回起動時にも URL が受け取れるようになって、
さらに `rake clean` したり一度デフォルトブラウザーを他のブラウザーに設定し直したりしていたら、
`handleGetURLEvent:withReplyEvent:` に戻しても初回起動時に受け取れるようになっていました。

原因がはっきりしないのが嫌な感じですが、何か設定が中途半端に残っているとうまく動かなかったようです。

## RubyMine

前回のもくもく会でインストールして、他では使っていなかったので、
ほぼ使っていないまま登録せず試用期限が切れてしまって、
30分ごとに終了するようになってしまいました。
