---
layout: post
title: "gitolite から gitolite3 への移行"
date: 2018-01-23 21:16:34 +0900
comments: true
categories: ubuntu linux
---
Ubuntu 14.04.5 (trusty) から 16.04.3 (xenial) に更新したら、 gitolite パッケージが消えてしまって、 gitolite3 で設定し直す必要がありました。

<!--more-->

## 環境

- Ubuntu 14.04.5 (trusty) から 16.04.3 (xenial)
- gitolite 2.3-1 から gitolite3 3.6.4-1

## gitolite3 パッケージのインストール

「gitolite のアクセス設定を管理するユーザの鍵を指定」では「gitolite version 2.x から移行する場合は空白にしてください。」と書いてあったので空欄のままにしました。

```
% sudo apt install gitolite3
[sudo] password for adminuser@ns6:
パッケージリストを読み込んでいます... 完了
依存関係ツリーを作成しています
状態情報を読み取っています... 完了
以下の追加パッケージがインストールされます:
  libcommon-sense-perl libjson-perl libjson-xs-perl libtypes-serialiser-perl
提案パッケージ:
  git-daemon-sysvinit gitweb
以下のパッケージが新たにインストールされます:
  gitolite3 libcommon-sense-perl libjson-perl libjson-xs-perl libtypes-serialiser-perl
アップグレード: 0 個、新規インストール: 5 個、削除: 0 個、保留: 0 個。
292 kB のアーカイブを取得する必要があります。
この操作後に追加で 970 kB のディスク容量が消費されます。
続行しますか? [Y/n]
```

## migrate

[migrating from gitolite v2](http://gitolite.com/gitolite/migr/index.html) をみると gitolite-admin/conf/gitolite.conf で `NAME/` ルールを使っている場合は書き換えが必要とありましたが、 `RW+` と `R` しか使っていなかったので、そのままで大丈夫そうでした。

移行手順には、はっきりとは書いていませんでしたが、後は `gitolite setup` を実行するだけでした。

```
% sudo su - git
$ gitolite setup

FATAL: '/home/git/.gitolite.rc' seems to be for older gitolite; please see
http://gitolite.com/gitolite/migr.html
$
```

`-pk` で公開鍵を指定しても同じエラーでした。

`.gitolite.rc` は変更した覚えがなかったので、リネームして再実行したところ、正常終了しました。

```
$ mv .gitolite.rc .gitolite.rc.v2
$ gitolite setup
$
```

その後、 ssh 経由で接続確認したところ、問題なく使えるようになっていました。

## まとめ

gitolite3 への移行手順の情報があまりなかったようなので、書いてみました。
基本的には gitolite3 パッケージを入れて `gitolite setup` を実行するだけでした。
