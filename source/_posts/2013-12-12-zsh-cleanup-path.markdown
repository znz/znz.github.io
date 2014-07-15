---
layout: post
title: "zshのPATHの自動重複削除や余計なPATHの削除"
date: 2013-12-12 00:00
comments: true
categories: zsh
---
シェルの中から `exec zsh` をしたり、
GNU screen や tmux を経由して間接的にシェルの中でシェルを開いたりするときに
何も考えずに `PATH` を追加していくと
どんどん長くなっていってしまうと思います。

`bash` などでも使えるように汎用的にしようとすると自前で頑張らないといけないのですが、
`zsh` では `zsh` 自体の機能で簡単に重複を防げます。

また、パスに望ましくないものが入っていた時に削除する方法も紹介します。

この投稿は
[zsh Advent Calendar 2013](http://qiita.com/advent-calendar/2013/zsh)
の12日目の記事です。

<!--more-->

## 重複削除

重複を削除するには

```
typeset -U path PATH
```

で `path` と `PATH` に unique という属性を付けて、
重複した値の最初だけ残すようにします。

ちゃんと設定できていれば、パラメーター展開の `t`
で調べた時に `unique` が付いています。

```
% echo ${(t)path}
array-unique-special
% echo ${(t)PATH}
scalar-export-unique-special
```

## 望ましくないパスの削除

ここでは 3 種類のパスを望ましくないものとして
除外するようにしました。

1. `NULL_GLOB` の `N` を使って存在しないディレクトリを除外
2. `-/` を使ってシンボリックリンクの場合のリンク先もチェックした上でディレクトリのみ残す
3. `^W` で world-writable という実行ファイルのパスとしては危険なパーミッションになっているディレクトリを除外
4. `${^spec}` で `RC_EXPAND_PARAM` を有効にしてパスすべてに適用

Glob Qualifiers なので `PATH` ではなく `path` を使ってフィルタリングしています。

```
path=(
    # allow directories only (-/)
    # reject world-writable directories (^W)
    ${^path}(N-/^W)
)
```

## `RC_EXPAND_PARAM` (2014-07-16 追記)

`$path(N)` のような書き方だと配列の最後の要素だけ `(N)` が適用されていて、
意図した挙動になっていませんでした。

`${^path}(N)` のように `RC_EXPAND_PARAM` を使うとすべての要素に Glob Qualifiers が付きます。

これを応用して `${^fpath}/cdr(N)` で `fpath` から `cdr` コマンドを探したり、
`${^path}/ruby(N)` で `path` 全部から `ruby` コマンドを `type -a ruby` のように全部探したりできます。

## まとめ

ここでは `zsh` の機能を活用してコマンド探索パス `PATH` をきれいにする方法を紹介しました。

これを応用して `fpath` などにも適用してみると良いかもしれません。
