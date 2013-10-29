---
layout: post
title: "rbenvのプラグインを簡単に追加出来るようにするrbenv-plugを作った"
date: 2013-10-29 23:55
comments: true
categories: ruby rbenv github
---
[rbenv](https://github.com/sstephenson/rbenv) のプラグインをインストールするのに
`git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build`
のように `ruby-build` を二回指定しないといけなかったり、
git の URL をコピペしないといけなかったりして面倒なので、
簡単にインストールできるようにするプラグイン
[rbenv-plug](https://github.com/znz/rbenv-plug)
を作りました。

[ruby-build](https://github.com/sstephenson/ruby-build)
のように `share` の下に一覧を持つようにしたので、
インストールできるプラグインは簡単に増やせます。

<!--more-->

## 使い方

[README](https://github.com/znz/rbenv-plug) に書いた通りです。

[plugins](https://github.com/sstephenson/rbenv/wiki/Plugins)
に載っているプラグインなら以下のように簡単にインストールできます。

    rbenv plug <plugin-name>

例えば
[ruby-build](https://github.com/sstephenson/ruby-build)
をインストールするなら以下のようになります。

    rbenv plug ruby-build

例えば
[rbenv-update](https://github.com/rkh/rbenv-update)
をインストールするなら、

    rbenv plug rbenv-update

のようにするか、
`rbenv-`
を省略して以下のようにします。

    rbenv plug update

git の URL を指定して任意のプラグインをインストールすることもできます。

    rbenv plug https://github.com/sstephenson/ruby-build.git

アンインストールも簡単にできます。
例えば
`rbenv-update`
をアンインストールするなら、

    rbenv unplug rbenv-update

のようにするか、
`rbenv-`
を省略して以下のようにします。

    rbenv unplug update

## インストール方法

以下のようにプラグインの一般的なインストール方法そのままで、
`$RBENV_ROOT/plugins` に `git clone` するだけです。

    mkdir -p ~/.rbenv/plugins
    git clone https://github.com/znz/rbenv-plug.git ~/.rbenv/plugins/rbenv-plug

## 仕組み

URL が指定された時は上の一般的なインストール方法と同様のことを実行するだけです。

プラグイン名を指定された時は
`share/rbenv-plug`
の中のスクリプトを実行します。

名前が `rbenv-` で始まるプラグインがほとんどなので、
`rbenv plug` や `rbenv unplug` の引数では
`rbenv-` を省略できるようにしました。

`share/rbenv-plug` のスクリプトで
[rbenv-aliases](https://github.com/tpope/rbenv-aliases)
なら `rbenv alias --auto` を追加で実行したり、
[rbenv-use](https://github.com/rkh/rbenv-use)
なら依存している `rbenv-whatis` もインストールしたりしています。

## 余談

`share/rbenv-plug` のファイルを追加している時に
[rbenv-plugin](https://github.com/taqtiqa/rbenv-plugin)
というのがあって、
今の `rbenv-plug` (と `rbenv-unplug`) という名前に変えました。
最初は `rbenv-plugin-install` (と `rbenv-plugin-uninstall`)
という名前で作りかけていたので、
`rbenv-plugin` のサブコマンドと思いっきり名前がかぶっていました。
`rbenv-plugins-install` という複数形の名前も使われてしまっていたので、
思い切って短い名前に変更しました。
