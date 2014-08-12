---
layout: post
title: "zshでfpathからwhichのように検索する話など"
date: 2014-08-12 23:08:30 +0900
comments: true
categories: zsh
---
zsh で `PATH` から `which` コマンドや `type` コマンドで実行ファイルを検索するように `fpath` から `autoload` される関数の実体を探したいことがあります。
そういうときは `${^fpath}/cdr(N)` のように書けば検索できます。

<!--more-->

## 解説

まず
`setopt` の `RC_EXPAND_PARAM` が設定されていないときには `$fpath/cdr` で配列の最後だけに `/cdr` がつくので、
`${^spec}` を使ってすべてにつけるようにしています。

最後に `(N)` で存在するものだけ残すようにしています。

```console 実行例
% echo ${^fpath}/cdr(N)
/usr/share/zsh/5.0.2/functions/cdr
```

## PATH で後ろに隠れているコマンドも探す

`rbenv` を使っていると `PATH` の後ろに `/usr/bin/ruby` が隠されるなど、
同じコマンドが `PATH` に複数存在することがあります。

そういうときに zsh なら `${^path}/ruby(N)` で隠れているコマンドも含めて展開できます。

```console 実行例
% echo ${^path}/ruby(N)
/Users/kazu/.rbenv/shims/ruby /usr/bin/ruby
```

## zsh スクリプトでコマンドの存在を調べる

シェルスクリプトの中で、
外部コマンドが実行できるかどうか調べるのに

    type peco >/dev/null 2>&1

のように `type` コマンドで調べることがありますが、
zsh なら

    (( ${+commands[peco]} ))

のように `commands` という連想配列を調べるという方法が使えます。
