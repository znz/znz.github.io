---
layout: post
title: "aptitude検索パターンの紹介"
date: 2013-12-04 00:00
comments: true
categories: debian ubuntu
---
Debian や Ubuntu のパッケージのインストールなどで
コマンドライン操作では `apt-get` や `aptitude`
を使いますが、
ここでは
`apt-get` では出来ない `aptitude`
の便利な使い方を紹介します。

この投稿は
[ディストリビューション/パッケージマネージャー Advent Calendar 2013](http://qiita.com/advent-calendar/2013/distro-pm)
の4日目の記事です。

<!--more-->

## 残ってしまった設定ファイルの削除

deb パッケージのアンインストールは `remove` と `purge` の二種類があって、
`purge` すれば設定ファイルまで消えるのですが、
`remove` だと設定ファイルは残ってしまいます。

普段は
`apt-get purge hoge`
や
`aptitude purge hoge`
で削除していたとしても、
依存関係で自動インストールされたものが
自動削除される時は `remove` になってしまって
設定ファイルが残ってしまいます。

そういうときに `aptitude` だと

```
aptitude purge '~c'
```

で設定ファイルだけ残ったパッケージを一気に `purge` できます。

この `~c` というのが `aptitude` の search term の一種で、
削除されていて `purge` (完全削除) されていない、
つまり設定ファイルがシステム上に残っているパッケージという意味になります。

## クオートの必要性

`''` でクオートしているのはシェルの展開を抑制するためで、
必須ではないのですが、
環境によって意図しない指定になることを避けるために、
常に `''` でくくっておくことをお勧めします。
ちなみに、この例だと `c` というユーザーが存在した場合に
そのホームディレクトリに展開されてしまいます。

## 長い形式と短い形式

ほとんどの search term は短い形式 (short form) と
長い形式 (long form) があり、
`~c` が短い形式で対応する長い形式は `?config-files`
になります。
つまり

```
aptitude purge '?config-files'
```

でも同じ意味になります。

## 検索パターンの確認

`aptitude search` でも `aptitude purge` でも `aptitude install` でも
同じように使えるので、
`aptitude search` で確認してから `aptitude purge` するとか、
`aptitude install` するという使い方も出来ます。

## Search Term reference

search term の一覧は `aptitude-doc-ja` パッケージを入れて参照するか、
[Search term reference](http://aptitude.alioth.debian.org/doc/ja/ch02s04s05.html)
などを参照してください。

## 他によく使っている search term

他によく使っているものとしては、

```
aptitude search '~U'
```

でアップデート対象のパッケージ一覧を見たり、

```
aptitude purge '~i~n 3.2.0-2[1-3]'
```

のような指定で古いカーネル関連のパッケージを削除したりするのをよく使います。
`~i` がインストール済みのものという意味で、
`~n` はパッケージ名の検索で引数は正規表現なのですが、
ここでは他に間違ってマッチしそうなものはないため、
`.` のエスケープは省略してしまうことが多いです。
`linux-.*` の部分も指定しなくても充分絞り込めるので
省略してしまっています。

`~i` は引数をとらなくて、デフォルトはパッケージ名の検索なので、
`~i 3.2.0-2[1-3]` と省略することも出来ます。

## 応用例

### 別 apt-line で入れたパッケージの検索など

たとえば Ubuntu なら

```
aptitude search '~i!~Oubuntu'
```

のように Origin が Ubuntu 以外のパッケージという検索で
Ubuntu の apt-line 以外から入れたパッケージが検索できます。

昔から使い続けている Debian なら

```
aptitude search '~i!~Odebian'
```

で他の apt-line からインストールしたものに加えて、
昔の Debian にパッケージが存在して、
今の Debian にはもう収録されていないパッケージが残っているものも
探すことも出来ます。

### exim から postfix への入れ替え

`aptitude install` や `aptitude purge` で
パッケージを指定する時に末尾に `_` を付けると `purge` できたり、
`+` を付けるとインストールできたりします。

これらを組み合わせると `postfix+` でインストールしつつ、
名前に `exim` を含むパッケージを `purge` することで
`default-mta` や `mail-transport-agent` に依存しているパッケージの
依存関係が満たせないと言われずにパッケージの入れ替えができます。

```
sudo aptitude purge '~i~nexim' postfix+
```

### バグがあったパッケージのインストールを止めたい

Debian の unstable や testing を使っていて
`apt-listbugs` を入れていると
インストール前にこのパッケージの
このバージョンは入れない方が
良さそうということがありますが、
同じソースパッケージで複数のパッケージに分かれていると
指定が面倒なことがあります。

たとえば `sysv-rc` でバグがあった時に
`apt-get source sysv-rc` でソースパッケージ名を調べて、
対象パッケージを確認した上で
以下のように
`aptitude forbid-version`
でそのバージョンはインストールしない、
ということが出来ます。

```
aptitude search '?installed ?source-package(sysvinit)'
sudo aptitude forbid-version '?installed ?source-package(sysvinit)'
```

### その他いろいろ

最後にその他のいろいろな例を列挙しておきます。
詳しい説明は `aptitude` のリファレンスなどを参照してください。

- `aptitude search '?maintainer(uwabami)` : メンテナで探す
- `aptitude search '~t minimal'` : タスクでインストールされるパッケージ
- `aptitude search '~n^lsb'` : 名前が lsb で始まるパッケージ
- `aptitude search '?section(metapackages)'` : メタパッケージ
- `aptitude search '?priority(important)'` : 優先度が重要のパッケージ
- `aptitude search '?provides(mail-transport-agent)'` : `mail-transport-agent` を提供しているパッケージ

## まとめ

今回は `aptitude` の検索パターンの便利な使い方を紹介しました。

今まで `apt-get` や `apt-cache` しか使っていなかったとか、
`aptitude` を使っていても単純にパッケージ名を指定しかしたことがなかったとか
いう人は `aptitude purge '~c'` だけでも試してみると良いのではないでしょうか。
