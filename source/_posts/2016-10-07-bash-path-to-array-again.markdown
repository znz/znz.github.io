---
layout: post
title: "bash で PATH を配列に分解の続き"
date: 2016-10-07 21:39:31 +0900
comments: true
categories: linux shell bash
---
zsh だと `PATH` と同期している配列変数として `path` があるのですが、
bash にはそういうものがなくて困ったので、分解する方法を考えてみた話の続きです。
末尾の空文字列や改行に対応しました。

<!--more-->

## 昨日の方法の問題点

[昨日の記事](/blog/2016-10-06-bash-path-to-array.html) に書きましたが、末尾の空文字列の処理に問題がありました。
また、書いていませんでしたが、改行に対応できていませんでした。

## 末尾対応

`PATH= bash -c 'ls'` を `/bin` やそれ以外で実行してみたら
空の `PATH` はカレントディレクトリのみと同じ意味のようなので、
「[末尾の空文字列に対応するために、PATHにの最後に:を追加した文字列をreadすればいい?](https://twitter.com/fixedpoint_jp/status/784229832930369536)」
という指摘のように末尾に `:` を追加する方法で良さそうでした。

つまり、以下のようになります。

```console
% bash -c 'PATH=/bin:; IFS=: read -r -a path <<<"$PATH:"; declare -p path'
declare -a path='([0]="/bin" [1]="")'
% bash -c 'PATH=/bin; IFS=: read -r -a path <<<"$PATH:"; declare -p path'
declare -a path='([0]="/bin")'
% bash -c 'PATH=; IFS=: read -r -a path <<<"$PATH:"; declare -p path'
declare -a path='([0]="")'
```

「PATH自体が空だった場合は振舞いが変わるけど。」という話もありましたが、
`PATH` 探索を再現できれば良いということを考えると空の場合も問題なさそうでした。

```console
% bash -c 'PATH=; ls'
bash: ls: No such file or directory
% cd /bin
% bash -c 'PATH=; ls /bin/bash'
/bin/bash
```

## 改行対応

いろいろ試していると改行を含む `PATH` を扱えないことに気づいたのですが、
`-d $'\0'` (以下の例ではコマンドラインのエスケープでひどいことになっていますが) を
指定して区切り文字を変えると here string の末尾に付く改行も入力の一部として
扱われてしまうのでうまくいかないようでしたが、末尾の要素を `unset` で削除することで
良い感じになりました。

`$'\0'` は NUL 文字で C 言語での終端文字なので、普通は `PATH` の途中に入らないことが期待できるので、ありえない文字として指定しています。

```console
% bash -c 'PATH=/bin; IFS=: read -d $'"'"'\0'"'"' -r -a path <<<"$PATH:"; declare -p path'
declare -a path='([0]="/bin" [1]="
")'
% bash -c 'PATH=/bin; IFS=: read -d $'"'"'\0'"'"' -r -a path <<<"$PATH:"; unset path[-1]; declare -p path'
declare -a path='([0]="/bin")'
% bash -c 'PATH=/foo$'"'"'\n'"'"'bar:/bin; IFS=: read -d $'"'"'\0'"'"' -r -a path <<<"$PATH:"; unset path[-1]; declare -p path'
declare -a path='([0]="/foo
bar" [1]="/bin")'
```

わかりやすいようにファイルにして実行すると以下のような感じです。

```console
% cat /tmp/p.bash
PATH=/foo$'\n'bar:/bin
IFS=: read -d $'\0' -r -a path <<<"$PATH:"
unset path[-1]
declare -p path
PATH=/foo$'\n'bar:/bin:
IFS=: read -d $'\0' -r -a path <<<"$PATH:"
unset path[-1]
declare -p path
% bash /tmp/p.bash
declare -a path='([0]="/foo
bar" [1]="/bin")'
declare -a path='([0]="/foo
bar" [1]="/bin" [2]="")'
```

## 他の処理例

`/etc/group` くらいのデータになると awk などを使った方が良いと思いますが、
`/etc/group` (末尾に空文字列が入ることがある) のパースも良い感じにできるようです。

設定されていない状態でも構わなかったり、
末尾に空文字列が入らないことがわかっている `/etc/passwd` などの場合は
bash の組み込みコマンドだけでいけそうです。

```console
% cat /tmp/t.bash
#!/bin/bash
sed 's/$/:/' /etc/group | while IFS=: read -r -a group; do
  declare -p group
done
% bash /tmp/t.bash
declare -a group='([0]="root" [1]="x" [2]="0" [3]="")'
declare -a group='([0]="daemon" [1]="x" [2]="1" [3]="")'
declare -a group='([0]="bin" [1]="x" [2]="2" [3]="")'
declare -a group='([0]="sys" [1]="x" [2]="3" [3]="")'
declare -a group='([0]="adm" [1]="x" [2]="4" [3]="syslog,vagrant")'
(略)
declare -a group='([0]="lpadmin" [1]="x" [2]="114" [3]="vagrant")'
declare -a group='([0]="sambashare" [1]="x" [2]="115" [3]="vagrant")'
declare -a group='([0]="vboxsf" [1]="x" [2]="999" [3]="")'
declare -a group='([0]="scanner" [1]="x" [2]="116" [3]="")'
declare -a group='([0]="colord" [1]="x" [2]="117" [3]="")'
```

## まとめ

最終的には

```bash
IFS=: read -d $'\0' -r -a path <<<"$PATH:"
unset path[-1]
```

で `PATH` を配列に変換できることがわかりました。

入力データに改行がないとわかっているなら `-d $'\0'` などは省略できるので、
入力データの性質に応じて適度に手を抜きつつ処理をするのが良いのではないでしょうか。
