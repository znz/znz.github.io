---
layout: post
title: ".zshrcの自動再コンパイル"
date: 2013-12-10 00:00
comments: true
categories: zsh
---
zsh のスクリプトは `zcompile` コマンドでコンパイルすることができます。
`.zshrc` も大きくなって読み込みに時間がかかるようになったらコンパイルすれば良さそうですが、
変更したときに手動でコンパイルし直すのは面倒なので、
自動で再コンパイルする設定を紹介します。

この投稿は
[zsh Advent Calendar 2013](http://qiita.com/advent-calendar/2013/zsh)
の10日目の記事です。

<!--more-->

## 設定方法

`.zshrc` の適当な場所に以下の設定を追加します。

```sh
if [ ~/.zshrc -nt ~/.zshrc.zwc ]; then
   zcompile ~/.zshrc
fi
```

これで `.zshrc.zwc` より `.zshrc` の方が新しい時に
`zcompile .zshrc` が自動で実行されます。

`.zshrc.zwc` がある時だけ実行されるので、
最初に `zcompile ~/.zshrc` を手動で実行しておきます。

## 読み込み順序

zsh 自体が `file.zwc` よりも `file` の方が新しい時に
`file` の方を読み込むようになっているので、
`.zshrc` の方が新しくて `zcompile` し直していないときに
`.zshrc.zwc` が読み込まれて
古い設定のままになるということはありません。

## 自動作成もする場合

ファイルを分割していて `.zshrc` はコンパイルする必要がない環境では
何もしない挙動になってほしいので、
こうしていますが、
最初から自動作成してほしいのなら、
存在チェックも付けて以下のようにすれば良いと思います。

```sh
if [ ! -f ~/.zshrc.zwc -o ~/.zshrc -nt ~/.zshrc.zwc ]; then
   zcompile ~/.zshrc
fi
```

## まとめ

ここでは `.zshrc` だけを対象にしましたが、
他にも自前の `fpath` のファイルなどで自動で再コンパイルしたいファイルがあれば
同様のことをするのがよいかもしれません。

あまり変更しないファイルなら更新チェックの無駄の方が大きいかもしれないので、
そのトレードオフは考えておく必要がありそうです。
