---
layout: post
title: "boot2docker のシリアルコンソールにつなぐ"
date: 2014-08-06 23:13:18 +0900
comments: true
categories: 
---
boot2docker-vm の設定をみていると、
`~/.boot2docker/boot2docker-vm.sock` にシリアルポートが設定されていたので使ってみました。

<!--more-->

## 対象バージョン

- Mac OS X 10.9.4
- VirtualBox 4.3.14
- boot2docker 1.1.2

## 接続方法

`nc -U ~/.boot2docker/boot2docker-vm.sock`
で接続して一度 Enter を押してログインプロンプトを再表示させて
`docker` でログインできます。

Control+C の INT シグナルで `nc` が終了してしまうので、
普通は素直に `boot2docker ssh` を使う方が良さそうです。

ssh で入れなくなった時の予備の手段として知っておくと良いかもしれません。

```
% nc -U ~/.boot2docker/boot2docker-vm.sock


Core Linux
boot2docker login: docker
docker
                        ##        .
                  ## ## ##       ==
               ## ## ## ##      ===
           /""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
           \______ o          __/
             \    \        __/
              \____\______/
 _                 _   ____     _            _
| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
boot2docker: 1.1.2
             master : 740106c - Thu Jul 24 03:24:10 UTC 2014
docker@boot2docker:~$
```

## 参考サイト

- [Vagrant (VirtualBox) でシリアルコンソールに繋ぐ - hibomaのはてなダイアリー](http://d.hatena.ne.jp/hiboma/20140130/1391061776 "Vagrant (VirtualBox) でシリアルコンソールに繋ぐ - hibomaのはてなダイアリー")
