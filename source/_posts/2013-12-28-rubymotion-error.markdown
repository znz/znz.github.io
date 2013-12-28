---
layout: post
title: "RubyMotionのOSXアプリの起動エラー"
date: 2013-12-28 18:55:18 +0900
comments: true
categories: ruby rubymotion osx
---
最近 RubyMotion で作っている OSX アプリが
`LSOpenURLsWithRole() failed with error -10810`
などで起動しないことがあったので原因を調べてみました。

<!--more-->

## LSOpenURLsWithRole() failed with error -10810

### よくある原因

`Hello.app/Contents/MacOS/Hello`
のような実行ファイルの実体に実行属性が外れているというのが
よくある原因のようで、
[ownCloud](http://owncloud.org/)
経由で同期したアプリが実行できなかったときは
これが原因でした。

### 別の原因

実行属性も問題なくて悩んでいたときに
コンソール (Console.app) のログを見てみると

```
2013/12/28 10:39:17.963 open[85406]: spawn_via_launchd() failed, errno=22 label=com.yourcompany.Hello.44032 path=.../Hello/build/MacOSX-10.9-Development/Hello.app/Contents/MacOS/Hello flags=0 : LaunchApplicationClient.cp #1168 LaunchApplicationViaLaunchDJobLabel() q=com.apple.main-thread
```
のようなログが出ていたので、
`LaunchApplicationViaLaunchDJobLabel`
で検索してみたところ、
[Mavericks problem opening fresh wrapper](http://portingteam.com/topic/9723-mavericks-problem-opening-fresh-wrapper/)
という話が見つかって、
そこに書いてあった

```
launchctl remove $(launchctl list | grep wineskin | awk '{ print $3 }')
```

という手順を参考にして解決しました。

`Hello` が2個残っていて、
`$()` だとダメだったので個別に `launchctl remove` しました。

```
% launchctl list | grep Hello
-	2	com.yourcompany.Hello.44032
-	0	com.yourcompany.Hello.53008
%  launchctl remove $(launchctl list | awk '/Hello/{print $3}')
usage: launchctl remove <job label>
zsh: exit 1     launchctl remove $(launchctl list | awk '/Hello/{print $3}')
%  echo launchctl remove $(launchctl list | awk '/Hello/{print $3}')
launchctl remove com.yourcompany.Hello.44032 com.yourcompany.Hello.53008
%  launchctl remove com.yourcompany.Hello.44032
%  launchctl remove com.yourcompany.Hello.53008
%  open -a Hello
```

### その他

その他の対象方法も含めて
[Error -10810 Openng Applictions or Relaunching Finder](http://www.thexlab.com/faqs/error-10810.html)
にいろいろ書いてあるようです。
(英語で長かったのでちゃんと読んでません。)

## MacBookPro6,2 で EXC_BAD_INSTRUCTION (SIGILL) になって起動しない

未解決です。

[EXC_BAD_INSTRUCTION crash in App Store version...](https://github.com/MohawkApps/Hacker-Bar/issues/36)
に似た話があって、
RubyMotion のバグっぽいという話のようです。

`motion create --template=osx HelloOSX`
で作成しただけのものでも
[クラッシュ](https://gist.github.com/znz/8158061#file-helloosx-report-txt)
して起動しませんでした。

RubyMotion を入れている自分の MacBook Air の環境の問題かどうかを切り分けるために
[VagrantX](http://shin1x1.github.io/vagrantx/)
も試してみましたが、
[同じくクラッシュ](https://gist.github.com/znz/8158061#file-vagrantx-report-txt)
して起動しませんでした。
