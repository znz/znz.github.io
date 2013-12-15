---
layout: post
title: "RubyMotionとRubyのバージョン"
date: 2013-12-15 14:49
comments: true
categories: ruby rubymotion
---
今のところ RubyMotion は毎月のもくもく会以外ではほとんど触れていないので、
RubyMotion と Ruby のバージョンの関係と言う小ネタです。

この投稿は
[RubyMotion Advent Calendar 2013](http://qiita.com/advent-calendar/2013/rubymotion)
の12日目の記事です。

<!--more-->

## RUBY_DESCRIPTION 確認

まず `Rakefile` の先頭で `RUBY_DESCRIPTION` を入れて、
普通の `ruby` コマンドのバージョンを表示して、
次に開発環境のプロンプトでも確認してみました。

```console
% head -n2 Rakefile
# -*- coding: utf-8 -*-
p RUBY_DESCRIPTION
% rake
"ruby 2.0.0p353 (2013-11-22 revision 43784) [x86_64-darwin12.5.0]"
     Build ./build/MacOSX-10.8-Development
   Compile ./app/app_delegate.rb
   Compile ./app/menu.rb
    Create ./build/MacOSX-10.8-Development/Hello.app/Contents
    Create ./build/MacOSX-10.8-Development/Hello.app/Contents/MacOS
      Link ./build/MacOSX-10.8-Development/Hello.app/Contents/MacOS/Hello
    Create ./build/MacOSX-10.8-Development/Hello.app/Contents/PkgInfo
    Create ./build/MacOSX-10.8-Development/Hello.app/Contents/Info.plist
      Copy ./resources/Credits.rtf
    Create ./build/MacOSX-10.8-Development/Hello.dSYM
       Run ./build/MacOSX-10.8-Development/Hello.app/Contents/MacOS/Hello
(main)> RUBY_DESCRIPTION
=> "RubyMotion (ruby 1.9.2) [universal-darwin13.0, x86_64]"
(main)> exit
```

外側は 2.0.0 なのに RubyMotion の方は 1.9.2 でした。

## まとめ

調べたきっかけはデバッグ用に `File.write`
([IO.write](http://docs.ruby-lang.org/ja/2.0.0/class/IO.html#S_WRITE))
が使えなかったからでした。
それは結局普通に `File.open` して書き込むことにしましたが、
開発環境側の Ruby のバージョンを変えても生成されるアプリ側の
Ruby のバージョンには影響しないという話でした。
