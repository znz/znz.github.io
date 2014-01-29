---
layout: post
title: "emacsやdistnotedを安定させるパッチをhomebrewで適用した"
date: 2014-01-29 15:59:28 +0900
comments: true
categories: emacs osx mavericks
---
Emacs で distnoted が大変なことになる件で、
さらにコメントで追加の情報があったので、
homebrew でのインストール時に
追加のパッチをあてるようにしました。

<!--more-->

## パッチ適用

適用法としては
`brew edit emacs`
で以下のように変更して、
`brew reinstall emacs`
で再インストールしました。
その後の `brew update` で引っかかる原因になるので、
reinstall したら `brew edit emacs` で戻しました。

適用されればどこでも良いのですが、最後に追加するようにしてみました。

```diff
diff --git a/Library/Formula/emacs.rb b/Library/Formula/emacs.rb
index db7c46c..26ec334 100644
--- a/Library/Formula/emacs.rb
+++ b/Library/Formula/emacs.rb
@@ -49,6 +49,7 @@ class Emacs < Formula
     if build.include? "cocoa" and build.include? "japanese"
       p[:p0].push("http://sourceforge.jp/projects/macemacsjp/svn/view/inline_patch/trunk/emacs-inline.patch?view=co&revision=583&root=macemacsjp&pathrev=583")
     end
+    p[:p1].push "https://gist.github.com/anonymous/8553178/raw/c0ddb67b6e92da35a815d3465c633e036df1a105/emacs.memory.leak.aka.distnoted.patch.diff"
     p
   end unless build.head?
 
```

## しばらく使ってみて

inline patch をあてたり外したりして試していた時は時々突然 Emacs 自体が落ちていたのですが、このパッチにしてからは起きていないようです。

## japanese パッチ問題

別の話として、ことえりの日本語入力の確定直後になぜか「英字」入力状態になってしまうことがあって、カーソルを動かさないと直らないという現象が以前から起きています。
これは inline patch と mavericks の組み合わせで発生しているようなのですが、原因は調べられていません。

文章を入力しにくくなるので、今は

```
brew uninstall emacs
brew install emacs --cocoa --srgb --with-gnutls
brew linkapps
```

という感じで `--japanese` は外した状態でインストールしています。

homebrew でオプションの追加は `brew reinstall emacs --japanese` のように `reinstall` でも出来るのに、
外す方は `reinstall` だと出来ないようなので、
`uninstall` してから `install` し直しています。
