---
layout: post
title: "独自ビルドした ruby を rbenv で使う"
date: 2013-09-14 23:22
comments: true
categories: ruby rbenv
---
[rbenv](https://github.com/sstephenson/rbenv)
を使っているなら、
[ruby-build](https://github.com/sstephenson/ruby-build)
でインストールしたもの以外にも
自分でビルドした ruby も rbenv で切り替えたくなることがありますが、
これは簡単に出来ます。

<!--more-->

元々 `rbenv` に `ruby-build` が必須というわけではないので、
`configure` の `--prefix` に
`~/.rbenv/versions/some-name`
を指定してインストールすれば良いだけです。

名前にはシェルで特別な意味を持つ文字を避ければ
何でも良さそうですが、
`rbenv install` で上書きされてしまう危険があるのと
単純に紛らわしいので、
`ruby-build` でインストール出来る名前は避けた方が無難だと思います。

例えば以下のように `configure` してインストールすれば `rbenv shell trunk` などで `ruby-build` でインストールしたものと同様に使えます。

* `./configure --prefix=$HOME/.rbenv/versions/trunk --enable-shared --enable-debug-env CPPFLAGS=-DRUBY_DEBUG_ENV`
* `./configure --prefix=$HOME/.rbenv/versions/git --enable-shared --enable-debug-env CPPFLAGS='-DRUBY_DEBUG_ENV -DARRAY_DEBUG'`
* `./configure --prefix=$HOME/.rbenv/versions/git-debug --enable-shared --enable-debug-env CPPFLAGS='-DRUBY_DEBUG_ENV -DARRAY_DEBUG -DBIGDECIMAL_DEBUG'`

`$HOME` を使っている理由は
`--prefix=~/path/to/somewhere`
だと `configure` の実行前には展開されず、
`autoconf` の `configure` ではなかったと思いますが、
`./~/path/to/somewhere`
にインストールされてしまうという問題が起きたことがあったので、
それ以来
`$HOME`
を使って目の前でフルパスに展開されるようにしています。

`-DRUBY_DEBUG_ENV` などを渡すのに `CPPFLAGS` を使うのは
[chkbuild](https://github.com/akr/chkbuild)
のやり方を
[Ruby CI](http://rubyci.org/)
のログをみて参考にしました。

安定したビルドを使いたいのなら、
Ruby CI
で使われているのと同じような引数を使うのが良いと思います。

普段私が使っているのは
`CPPFLAGS='-DRUBY_DEBUG_ENV -DARRAY_DEBUG`
です。

`-DBIGDECIMAL_DEBUG`
まで付けると、デバッグ出力が多すぎたり、
`make test-all`
が途中で止まってしまったりして
問題が起きそうなので、
おすすめしません。
