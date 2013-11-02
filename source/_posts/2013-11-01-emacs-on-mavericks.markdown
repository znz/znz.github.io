---
layout: post
title: "MavericksでEmacs.appが起動時にホームディレクトリにならない"
date: 2013-11-01 18:00
comments: true
categories: emacs mavericks mac homebrew
---
Mac OS X を Mavericks にあげたら homebrew で入れた
Emacs.app を起動した時に `default-directory`
がホームディレクトリから `/` に変わってしまっていたので、
原因を調べてみました。

<!--more-->

## 調べたきっかけ

きっかけは
[Mavericksにアップデートして遭遇した不具合　まとめ](http://qiita.com/ksato9700/items/1ec373895b9693529f82)
を見て自分の環境だけで起きている現象ではないと知ったからです。

## 原因

Emacs のソースの以下の部分で `-psn` で始まる引数の有無で
ホームディレクトリに移動するかどうかを判定しているのに、
Mavericks だと引数なしで起動されるようになったからのようです。

```c emacs-24.3/src/emacs.c
 #ifdef HAVE_NS
   ns_pool = ns_alloc_autorelease_pool ();
   if (!noninteractive)
     {
 #ifdef NS_IMPL_COCOA
       if (skip_args < argc)
         {
 	  /* FIXME: Do the right thing if getenv returns NULL, or if
 	     chdir fails.  */
           if (!strncmp (argv[skip_args], "-psn", 4))
             {
               skip_args += 1;
               chdir (getenv ("HOME"));
             }
           else if (skip_args+1 < argc && !strncmp (argv[skip_args+1], "-psn", 4))
             {
               skip_args += 2;
               chdir (getenv ("HOME"));
             }
         }
 #endif  /* COCOA */
     }
 #endif /* HAVE_NS */
```

## 状況

ちなみに Mac OS X 10.8.5 では、たとえば
`/usr/local/Cellar/emacs/24.3/Emacs.app/Contents/MacOS/Emacs -psn_0_29637698`
のように起動されていました。

Qiita の記事では Dock から起動した時のことを書いていますが、
`open -a Emacs.app` のように起動しても同じでした。

RubyMotion で最小限のカレントディレクトリを表示するだけのアプリを作って試してみたところ、
起動時にカレントディレクトリが `/` になっているのは
Cocoa アプリでは普通の動作のようでした。

## 対応状況

[ML での情報](http://osdir.com/ml/general/2013-10/msg61593.html)
によると trunk には対応がチェックインされているということで、
調べてみると
revision 114730 <!-- http://bzr.savannah.gnu.org/lh/emacs/trunk/revision/114730 -->
と
revision 114882 <!-- http://bzr.savannah.gnu.org/lh/emacs/trunk/revision/114882 -->
のパッチをあわせて以下のように対応すれば良さそうに見えました。

```diff emacs.c.diff
--- src/emacs.c.orig	2013-02-06 13:33:36.000000000 +0900
+++ src/emacs.c	2013-11-02 22:38:45.000000000 +0900
@@ -1158,10 +1158,13 @@
   if (!noninteractive)
     {
 #ifdef NS_IMPL_COCOA
+      /* Started from GUI? */
+      /* FIXME: Do the right thing if getenv returns NULL, or if
+         chdir fails.  */
+      if (! inhibit_window_system && ! isatty (0))
+        chdir (getenv ("HOME"));
       if (skip_args < argc)
         {
-	  /* FIXME: Do the right thing if getenv returns NULL, or if
-	     chdir fails.  */
           if (!strncmp (argv[skip_args], "-psn", 4))
             {
               skip_args += 1;
```

というわけで homebrew の方に
[fix default-directory on Cocoa and Mavericks](https://github.com/mxcl/homebrew/pull/23897)
として pull request を出してみました。
