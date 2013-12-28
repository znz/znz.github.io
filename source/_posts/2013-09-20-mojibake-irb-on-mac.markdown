---
layout: post
title: "Macでirbなどでの文字化けを直した"
date: 2013-09-20 18:00
comments: true
categories: osx irb rails mojibake
---
基本的には
[rails cで日本語が通らないときの直し方](http://qiita.com/irohiroki/items/c82657b5cb4bdb2aaac4)
のやり方そのままで、
one liner としてまとめただけです。

先に結論を書いておくと
`install_name_tool -change /usr/lib/libedit.3.dylib $(brew list readline | grep libreadline.dylib) $(ruby -r readline -e 'puts $".grep /readline/')`
になりました。

<!--more-->

## `readline.bundle` の探し方

まず `readline.bundle` の探し方です。

実際に `require 'readline'` した時の
`$LOADED_FEATURES` から
探すのが確実ということで、
`ruby -r readline -e 'puts $".grep /readline/'`
としました。
試したところ、
`Array#grep`
で見つかるのが1個だけだったので、
そのまま `puts` しています。
複数見つかるようならもう少し調整が必要だと思います。

```
% ruby -r readline -e 'puts $".grep /readline/'
/Users/kazu/.rbenv/versions/2.0.0-p247/lib/ruby/2.0.0/x86_64-darwin12.4.1/readline.bundle
% otool -L $(ruby -r readline -e 'puts $".grep /readline/')
```

## readline のインストール

homebrew を使っているので、
`brew install readline`
でインストールしました。

インストールされた
`libreadline`
の場所は
`brew list readline | grep libreadline.dylib`
で調べました。

`libedit`
が
`/usr/lib/libedit.3.dylib`
という名前でリンクされているので、
`libreadline.6.dylib`
という名前を使った方が良いのかもしれませんが、
homebrew でインストールされているパスに
バージョンが含まれていて、
ファイル名にバージョンを含めても意味がないと思って、
バージョンなしにしました。

```
% brew list readline | grep libreadline.dylib
/usr/local/Cellar/readline/6.2.4/lib/libreadline.dylib
```

## `install_name_tool`

Qiita の記事にあったように
`install_name_tool -change /usr/lib/libedit.3.dylib $(brew list readline | grep libreadline.dylib) $(ruby -r readline -e 'puts $".grep /readline/')`
でリンクを変更しました。
念のため、実行前に
`ruby -rpp -e 'pp ARGV' --`
で実行内容を確認してから実行しました。

```
% ruby -rpp -e 'pp ARGV' -- install_name_tool -change /usr/lib/libedit.3.dylib $(brew list readline | grep libreadline.dylib) $(ruby -r readline -e 'puts $".grep /readline/')
["install_name_tool",
 "-change",
 "/usr/lib/libedit.3.dylib",
 "/usr/local/Cellar/readline/6.2.4/lib/libreadline.dylib",
 "/Users/kazu/.rbenv/versions/2.0.0-p247/lib/ruby/2.0.0/x86_64-darwin12.4.1/readline.bundle"]
% install_name_tool -change /usr/lib/libedit.3.dylib $(brew list readline | grep libreadline.dylib) $(ruby -r readline -e 'puts $".grep /readline/')
```
