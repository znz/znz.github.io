---
layout: post
title: "Mac で rabbit が動かなくなったので brew reinstall したら直った"
date: 2013-09-09 23:00
comments: true
categories: rabbit mac homebrew
---
いつの間にか動かなくなっていた rabbit (2.1.1) が `brew reinstall` で動くようになったという話です。

<!--more-->

現象としては rabbit のウィンドウが出てくるのに無反応で、スライドの内容も表示されないし、クリックにも反応せず、 `Ctrl+C` での停止もきかないので `kill -KILL` で強制終了するしかないという状態でした。

現象から考えて、リンクしているライブラリとの不整合が原因だろうと予想してインストール手順をやり直すことにしました。

[Mac OS XでHomebrewを使ったインストール方法](http://rabbit-shocker.org/ja/install/homebrew.html) の「brew install と brew link の実行」の手順を以下のようにちょっと変更してやり直したら動くようになりました。

```
brew reinstall cairo
brew unlink cairo
brew link cairo
brew unlink pixman
brew link --overwrite pixman
brew reinstall pango
brew reinstall gtk+
brew reinstall poppler --with-glib
```

これでも動かなければ gem も uninstall して install し直す予定でした。
