---
layout: post
title: "第 12 回 RubyMotion もくもく会 in Osaka に参加した"
date: 2014-06-20 19:27:13 +0900
comments: true
categories: event ruby rubymotion osx
---
第 1 回から第 11 回にも参加した RubyMotion もくもく会 in Osaka の
[第 12 回 RubyMotion もくもく会 in Osaka](http://rubymotionjp.connpass.com/event/6666/)
に参加してきました。

次回の
[第 13 回 RubyMotion もくもく会 in Osaka](http://rubymotionjp.connpass.com/event/7079/)
は 07/16(水) になりました。

<!--more-->

## メモ

- 一周年記念
- RubyMotion は Android にも対応予定
- RubyMotion のライセンスが expired して更新しても `rake` のときの `Your maintenance plan is expired. Run 'motion account' to renew it.` というメッセージはキャッシュの影響で 24 時間ぐらい待たないと消えないらしいという話
- バージョンは 2.29 が最新
- Swift が話題
- [割り勘電卓](http://tools.msng.info/)
- https://github.com/shin1x1/vagrantx のソース公開
- [MacFace v2](https://github.com/MacFace/MacFace/tree/v2.0) が Swift で作り直し中

## やったこと

Interface Builder と `gem 'ib'` で GUI を作り直そうとしていました。

GUI 作成は細かいサイズを気にしなくて良い Tcl/Tk の pack 方式や
Web での bootstrap のようなやり方の方が慣れていて、
GUI ツールで作成するのは GTK+2 の glade ぐらいしかまともに使ったことがなかったので、
使い方や良い感じに配置する方法で結構悩んでいました。
悩んでいた部分は最後の成果発表の時にいろいろ教えてもらったりしました。

コードとの組み合わせは、検索で見つけた

    @mainWindowController = MainWindow.alloc.initWithWindowNibName('MainWindow')
    @mainWindowController.window.makeKeyAndOrderFront(self)

という方法で window は表示できたものの、
`@mainWindowController.window` が `nil` になっていて、
`makeKeyAndOrderFront` の呼び出しは失敗していました。

ひとまず現状は [ib ブランチ](https://github.com/znz/urltrapper/tree/ib) に公開しています。
