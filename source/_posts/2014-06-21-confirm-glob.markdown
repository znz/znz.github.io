---
layout: post
title: "シェルで glob 結果を事前に確認する方法"
date: 2014-06-21 13:16:56 +0900
comments: true
categories: linux shell zsh
---
`rm *~` のつもりで `rm * ~` (半角スペースが混ざっている) のように実行してしまうような間違いをすると危険です。

その対策として `tcsh` には `rmstar` という設定があったり
`zsh` には `RM_STAR_SILENT` や `RM_STAR_WAIT` という設定があるのですが、
ちゃんと展開結果を確認してからコマンドを実行する方が安全です。

また `rm` 以外でも展開結果を事前に確認できると便利なことが多いです。

<!--more-->

## 展開結果の確認方法

`bash` や `zsh` の一般的なキー割り当てだと `C-x g` (Control を押しながら x を押して Control を離して g) で展開結果を確認できます。

Tab キーだとコマンドライン中に展開されてしまいますが、
`C-x g` だと確認だけ出来ます。

### bash の場合

たとえば `bash` なら以下のように展開結果が出て、
プロンプトの行が出てきます。

```console bash
    $ echo /etc/host*<C-x>g
    host.conf    hostname     hosts        hosts.allow  hosts.deny
    $ echo /etc/host*
```

### zsh の場合

`zsh` ならプロンプトの行の下に展開結果が出てきます。

```console zsh
    % echo /etc/host*
    /etc/host.conf    /etc/hostname     /etc/hosts        /etc/hosts.allow  /etc/hosts.deny
```

## Tab キーの動作

比較のため Tab キーでの動作例も載せておきます。

### bash の場合

実際には同じ行でしたが、最初の展開結果に置き換わりました。

```console bash
    $ echo /etc/host*<tab>
    $ echo /etc/host.conf
```

### zsh の場合

こちらも実際には同じ行ですが、すべての展開結果に置き換わりました。

```console zsh
    % echo /etc/host*<tab>
    % echo /etc/host.conf /etc/hostname /etc/hosts /etc/hosts.allow /etc/hosts.deny
```

glob で指定しにくい一部のファイルだけ除外するなど、展開された方が便利な時は
Tab キーで展開してからコマンドラインを編集して実行することもあります。

## まとめ

一般的な bash 環境ならどこでも使えて、
複雑な glob の展開結果の確認にも便利なので、
`C-x g` は非常にオススメです。
