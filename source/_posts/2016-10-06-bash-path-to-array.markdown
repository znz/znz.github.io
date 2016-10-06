---
layout: post
title: "bash で PATH を配列に分解"
date: 2016-10-06 23:50:00 +0900
comments: true
categories: linux shell
---
zsh だと `PATH` と同期している配列変数として `path` があるのですが、
bash にはそういうものがなくて困ったので、分解する方法を考えてみました。

<!--more-->

## 結論

先に結論を書いておくと、最終的には `IFS=: read -r -a path <<<"$PATH"` という方法で分解できました。

```console
% bash -c 'IFS=: read -r -a path <<<"$PATH"; declare -p path'
```

のように動作確認できます。

declare で表示というのは [シェルスクリプトのデバッグは typeset または declare を使うと良いかも - よんちゅBlog](http://yonchu.hatenablog.com/entry/2013/07/09/230656 "シェルスクリプトのデバッグは typeset または declare を使うと良いかも - よんちゅBlog") を参考にしました。

## `-r` オプション

`-r` オプションは `read` コマンドを使うときの定石ですが、具体的には `\:` のような並びがあるときに影響がありました。

```console
% PATH="/bin:/tmp/foo\:/tmp/bar" bash -c 'IFS=: read -r -a path <<<"$PATH"; declare -p path'
declare -a path='([0]="/bin" [1]="/tmp/foo\\" [2]="/tmp/bar")'
% PATH="/bin:/tmp/foo\:/tmp/bar" bash -c 'IFS=: read -a path <<<"$PATH"; declare -p path'
declare -a path='([0]="/bin" [1]="/tmp/foo:/tmp/bar")'
```

実際のパスの挙動は `-r` がある場合と同じようでした。

```console
% mkdir /tmp/foo\\
% echo echo hoge > /tmp/foo\\/hoge
% chmod +x /tmp/foo\\/hoge
% PATH="/bin:/tmp/foo\:/tmp/bar" bash -c 'IFS=: read -a path <<<"$PATH"; for p in "${path[@]}"; do test -x "$p/hoge" && "$p/hoge"; done'
PATH="/bin:/tmp/foo\:/tmp/bar" bash -c 'IFS=: read -r -a path <<<"$PATH"; for p in "${path[@]}"; do test -x "$p/hoge" && "$p/hoge"; done'
hoge
% PATH="/bin:/tmp/foo\:/tmp/bar" hoge
hoge
```

## `-d` オプション

`-d` オプションというのもあったので試してみたのですが、そこで完全に読み込み終了になってしまって、期待した動作にはなりませんでした。

```console
% PATH="/bin:/tmp/foo\:/tmp/bar" bash -c 'read -d : -r -a path <<<"$PATH"; declare -p path'
declare -a path='([0]="/bin")'
```

## `-a` オプション

`-a` オプションはこのように複数の変数を指定する代わりにひとつの変数を指定して配列を代入してくれるオプションでした。

`-a` オプションがないと指定した変数のうち、最後に残り全て入ってしまうようです。

```console
PATH="/bin:/tmp/foo\:/tmp/bar:/tmp/baz" bash -c 'IFS=: read -r path1 path2 path3 <<<"$PATH"; declare -p path1 path2 path3'
declare -- path1="/bin"
declare -- path2="/tmp/foo\\"
declare -- path3="/tmp/bar:/tmp/baz"
```

## here string

`<<<word` は `echo word | ` のようなもので、標準入力に `word` を渡してくれる機能です。

## IFS

`read` などの単語区切りです。
デフォルトは空白、タブ、改行です。

ずっと変えてしまうと影響が大きすぎるので、 `read` の行だけ変更するようにしています。

また、このやり方を使うことで空白の入ったディレクトリを含むパスもうまく扱えます。

## カレントディレクトリを表す空のパスの扱い

頭や途中に入った空文字列 (カレントディレクトリを表す) は扱えたのですが、末尾にある場合はうまくいきませんでした。

セキュリティ上の問題もあるので、普通は設定しないと思うので、対応しなくても問題はないと思いますが、完全に変換したい場合は特別扱いを追加する必要がありそうでした。

```console
%  bash -c 'PATH=:/bin; IFS=: read -r -a path <<<"$PATH"; declare -p path'
declare -a path='([0]="" [1]="/bin")'
%  bash -c 'PATH=/bin::/bin; IFS=: read -r -a path <<<"$PATH"; declare -p path'
declare -a path='([0]="/bin" [1]="" [2]="/bin")'
%  bash -c 'PATH=/bin:; IFS=: read -r -a path <<<"$PATH"; declare -p path'
declare -a path='([0]="/bin")'
```

## まとめ

zsh には元から `path` があるし、 `/bin/sh` には配列がないので bash 限定ではありますが、
`IFS=: read -r -a path <<<"$PATH"` で実用上問題なく変換できるということがわかったので、必要な時には使うと良いのではないでしょうか。
