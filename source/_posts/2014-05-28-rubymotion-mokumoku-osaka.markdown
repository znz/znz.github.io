---
layout: post
title: "第 11 回 RubyMotion もくもく会 in Osaka に参加した"
date: 2014-05-28 19:38:39 +0900
comments: true
categories: event ruby rubymotion osx
---
第 1 回から第 10 回にも参加した RubyMotion もくもく会 in Osaka の
[第 11 回 RubyMotion もくもく会 in Osaka](http://connpass.com/event/5674/)
に参加してきました。

次回の
[第 12 回 RubyMotion もくもく会 in Osaka](http://connpass.com/event/6666/)
は 06/20(金) になりました。

<!--more-->

## メモ

- `gem 'ib'` と Auto Layout
- [アプリで飯を食う: AdMobメディエーションで広告(Nend,iAd,AdMob)を切り替える方法](http://kojisatoapp.blogspot.jp/2014/03/admobnendiadadmob.html)
- `usingSpringWithDamping`
- [Vagrant Manager](http://vagrantmanager.com/)
- [Hash のソート](http://stackoverflow.com/questions/13216092/how-to-sort-a-hash-by-value-in-descending-order-and-output-a-hash-in-ruby)
- [URLTrapper](http://urltrapper.n-z.jp/) v0.1
- [ここを送る](https://itunes.apple.com/jp/app/kokowo-songru/id473519853?mt=8)
- [VagrantX](http://shin1x1.github.io/vagrantx/) v0.2.0
- [MacFace](http://macface.github.io/)

## URLTrapper 公開

今日は URLTrapper の公開作業をしていました。

最終的には
[URLTrapper](http://urltrapper.n-z.jp/)
の[ソースコードを公開](https://github.com/znz/urltrapper)して、
[v0.1 をリリース](https://github.com/znz/urltrapper/releases/tag/v0.1)しました。

## Config 設定

まず
[RubyMotion Project Management Guide](http://rubymotion.jp/RubyMotionDocumentation/guides/project-management/index.html)
の 2. Configuration をみて identifier などを設定していきました。

## リリースファイル作成

`rake build` で `build/MacOSX-10.9-Release/*.app` が出来るので、
それを Finder で zip にして、
バージョン付きのファイル名にしました。

## リリース

github の Release からリリースを作成して、
先ほどの zip ファイルをアップロードしてリリースしました。

途中、リリースノートにスクリーンショットをアップロードしました。
このスクリーンショットは後で GitHub Pages を作る時にも使いました。

## GitHub Pages 作成

github の Settings から GitHub Pages にある Automatic page generator を使って作成しました。

自動作成されたものから、ダウンロードのリンクのところは VagrantX を参考にして、
ソースコードのダウンロードからバイナリの zip のダウンロードに書き換えました。

## ドメイン問題

`znz.github.io` は、この blog 用に使っていて、
`znz.github.io/urltrapper` を開くと
`blog.n-z.jp/urltrapper` に飛ばされてしまって
困ってしまったので、
https://github.com/znz/urltrapper/blob/gh-pages/CNAME
として個別のサブドメインを使うように設定したのですが、
すぐには反映されなくて、しばらく悩んでいました。

最後の成果発表のタイミングでは反映されていて見えるようになっていました。
