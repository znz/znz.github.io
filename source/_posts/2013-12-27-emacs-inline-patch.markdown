---
layout: post
title: "Cocoa Emacsのinline patch修正版を使ってみた"
date: 2013-12-27 23:00:00 +0900
comments: true
categories: emacs mac
---
[launchdでdistnotedを定期的に終了させる](/blog/2013-11-13-killall-distnoted-periodically.html)
話のコメントで
[Emacs.appでインラインパッチを当てた時にdistnotedが暴走しなくなる](https://gist.github.com/anonymous/8142555)
修正を教えてもらったので、
試してみました。

<!--more-->

## homebrew で適用するパッチの変更

適用法としては
`brew edit emacs`
で以下のように変更して、
`brew reinstall emacs`
で再インストールしました。

```diff
diff --git a/Library/Formula/emacs.rb b/Library/Formula/emacs.rb
index 712c3d1..5ce4ce2 100644
--- a/Library/Formula/emacs.rb
+++ b/Library/Formula/emacs.rb
@@ -47,7 +47,7 @@ class Emacs < Formula
     # "--japanese" option:
     # to apply a patch from MacEmacsJP for Japanese input methods
     if build.include? "cocoa" and build.include? "japanese"
-      p[:p0].push("http://sourceforge.jp/projects/macemacsjp/svn/view/inline_patch/trunk/emacs-inline.patch?view=co&revision=583&root=macemacsjp&pathrev=583")
+      p[:p0].push("https://gist.github.com/anonymous/8142555/raw/d67ad1dc814579d125afbd18de3a62ba69895601/emacs-inline.patch")
     end
     p
   end unless build.head?
```

## 元のパッチからの変更点

元の sourceforge.jp のパッチとの差分をとってみると、
以下のメソッド呼び出しが変わっているだけでした。

```diff
diff --git a/emacs-inline.patch.sfjp b/emacs-inline.patch.gist
index 52f2052..d67ad1d 100644
--- a/emacs-inline.patch.sfjp
+++ b/emacs-inline.patch.gist
@@ -1015,7 +1015,7 @@ diff -r -N -p ../emacs-24.3.org/src/nsterm.m src/nsterm.m
                                                name: nil object: nil]; */
 +   [[NSDistributedNotificationCenter defaultCenter] addObserver: NSApp
 + 					selector: @selector (changeInputMethod:)
-+ 						   name: @"AppleSelectedInputSourcesChangedNotification" object: nil];
++ 						   name: @"AppleSelectedInputSourcesChangedNotification" object: nil suspensionBehavior:NSNotificationSuspensionBehaviorDeliverImmediately];
   
     dpyinfo = xzalloc (sizeof *dpyinfo);
   
```

## まとめ

使ってみて問題がなければ sourceforge.jp の方に取り込んでもらうのが良さそうです。

しばらく使ってみた感じだと distnoted のメモリ使用量が増えていっても、
一度 Emacs.app を終了すると distnoted のメモリ使用量が一気に減るようになりました。

