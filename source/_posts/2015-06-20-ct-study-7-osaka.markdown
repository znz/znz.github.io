---
layout: post
title: "第7回 コンテナ型仮想化の情報交換会＠大阪に参加しました"
date: 2015-06-20 14:30:00 +0900
comments: true
categories: event lxcjp
---
[第7回 コンテナ型仮想化の情報交換会＠大阪](http://ct-study.connpass.com/event/15121/ "第7回 コンテナ型仮想化の情報交換会＠大阪")
に参加しました。

<!--more-->

入り口に pepper くんがいました。
懇親会でも人気でした。

以下メモです。

## いまさら聞けない Linux コンテナの基礎

- https://speakerdeck.com/tenforward/jin-sarawen-kenai-linux-kontenafalseji-chu-2015-06-20
- cgroup などの Linux コンテナの基礎についての話でした。
- [jailing - chroot jailを構築・運用するためのスクリプトを書いた](http://blog.kazuhooku.com/2015/05/jailing-chroot-jail.html "jailing - chroot jailを構築・運用するためのスクリプトを書いた")
- http://criu.org/
- overlayfs

## いまさら聞けない Linux コンテナの基礎 (後半)

- いろいろと namespace のデモ
- kernel のバージョンの他にも unshare コマンドが入っている util-linux のバージョンも重要
- 最近の systemd だと `mount --make-shared /` 相当になっている
- いろいろなコンテナ関係の紹介
- https://github.com/mhiramat/mincs
- LXC のデモ (lxc-なんとか コマンド)
- LXD のデモ (lxc コマンド)

## dokku を本番環境で使ってみた話

- 自分の発表でした。
- http://slide.rabbit-shocker.org/authors/znz/dokku-201506/
- Heroku を使っていない理由は VPS の方が安い
- apache+passenger だと ruby や passenger のバージョンアップが辛かった

## VIMAGE を使ってみた話

- whois owatan.jp
- ConoHa は 17 個の IPv6 アドレスが付いてくる
- epair

## アンケート

- https://goo.gl/g75FNJ
