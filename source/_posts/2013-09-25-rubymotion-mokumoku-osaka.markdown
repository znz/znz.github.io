---
layout: post
title: "第 3 回 RubyMotion もくもく会 in Osaka に参加した"
date: 2013-09-25 20:43
comments: true
categories: event ruby rubymotion
---
第 1 回と第 2 回にも参加した RubyMotion もくもく会 in Osaka の
[第 3 回 RubyMotion もくもく会 in Osaka](http://connpass.com/event/3450/)
に参加してきました。

<!--more-->

OS X アプリの勉強をしようと思って、
とりあえず
`motion create --template=osx Hello`
で生成されたファイルを読んでいたら、
`app/menu.rb`
の
`Italic`
の
`keyEquivalent:`
が
`Bold`
と同じ
`'b'`
になっていたのを見つけて、
他もテキストエディットと見比べていたら
違う部分を見つけたので、
`motion support`
で報告してみました。

```diff
diff --git a/app/menu.rb b/app/menu.rb
index 58b48f8..0da85e1 100644
--- a/app/menu.rb
+++ b/app/menu.rb
@@ -47,7 +47,7 @@ class AppDelegate
     fontMenu = createMenu('Font') do
       addItemWithTitle('Show Fonts', action: 'orderFrontFontPanel:', keyEquivalent: 't')
       addItemWithTitle('Bold', action: 'addFontTrait:', keyEquivalent: 'b')
-      addItemWithTitle('Italic', action: 'addFontTrait:', keyEquivalent: 'b')
+      addItemWithTitle('Italic', action: 'addFontTrait:', keyEquivalent: 'i')
       addItemWithTitle('Underline', action: 'underline:', keyEquivalent: 'u')
       addItem(NSMenuItem.separatorItem)
       addItemWithTitle('Bigger', action: 'modifyFont:', keyEquivalent: '+')
@@ -60,9 +60,11 @@ class AppDelegate
       addItemWithTitle('Justify', action: 'alignJustified:', keyEquivalent: '')
       addItemWithTitle('Align Right', action: 'alignRight:', keyEquivalent: '}')
       addItem(NSMenuItem.separatorItem)
-      addItemWithTitle('Show Ruler', action: 'toggleRuler:', keyEquivalent: '')
-      addItemWithTitle('Copy Ruler', action: 'copyRuler:', keyEquivalent: 'c')
-      addItemWithTitle('Paste Ruler', action: 'pasteRuler:', keyEquivalent: 'v')
+      addItemWithTitle('Show Ruler', action: 'toggleRuler:', keyEquivalent: 'r')
+      item = addItemWithTitle('Copy Ruler', action: 'copyRuler:', keyEquivalent: 'c')
+      item.keyEquivalentModifierMask = NSCommandKeyMask|NSControlKeyMask
+      item = addItemWithTitle('Paste Ruler', action: 'pasteRuler:', keyEquivalent: 'v')
+      item.keyEquivalentModifierMask = NSCommandKeyMask|NSControlKeyMask
     end
 
     addMenu('Format') do
```

他にはサンプルとして
https://github.com/satococoa/rubymotion-osx-browser
とか
https://github.com/HipByte/RubyMotionSamples
とかをちょっと動かして見たりしていました。


他の人の話としては、
`UICreateCGImageFromIOSurface`
というシンボルで問題が起きて、
最初は
[ZBar](https://github.com/ZBar/ZBar)
というライブラリが疑われていたけど違ったとかいう話とか、
[Reveal.framework が原因で Release ビルドで外せば良い](http://support.revealapp.com/discussions/suggestions/8-reference-to-uicreatecgimagefromiosurface-should-be-removed-for-release-scheme)
という話とか、
東京と Google ハングアウトでつないでみてたとか、
[(1/2) - 第二回 RubyMotion でアプリケーションを作ろう (1) - 実践！RubyMotion - Mobile Touch - モバイル/タブレット開発者およびデザイナー向け情報ポータル](http://mobiletou.ch/2013/09/002-create-application)
で開発中のアプリをみせてもらったりとかがありました。

東京とのハングアウトが終わった後は
[Vagrant入門ガイド](https://gihyo.jp/dp/ebook/2013/978-4-7741-6024-5)
の話とか、
[JAWS FESTA Kansai 2013](http://jfk2013.jaws-ug.jp/)
の話とかもしていました。

最後に次回も東京と日程を合わせるということで
[第 4 回 RubyMotion もくもく会 in Osaka](http://connpass.com/event/3557/)
は 10/23 (水) 19:30 からということになりました。
