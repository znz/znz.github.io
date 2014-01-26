---
layout: post
title: "第 80 回 関西 Debian 勉強会に参加した"
date: 2014-01-26 14:05:18 +0900
comments: true
categories: event debian kansaidebian
---

[第 80 回 関西 Debian 勉強会](https://wiki.debian.org/KansaiDebianMeeting/20140126)
に参加してきました。

今回は事前課題の発表で話が盛り上がって、その後の佐々木さんの話も盛り上がって長かったので、もくもくの会は少しだけでした。

事前課題で募集があったので、 LT もしました。

<!--more-->

## メモ

以下 URL などのメモです。

- Debian で redmine を使う方法
  - stable を使う
  - gem2deb
  - deb パッケージの ruby は使わない
- UEFI のマシンでのインストールが大変という話
- [docker](http://www.docker.io/) が [docker.io](http://packages.qa.debian.org/d/docker.io.html) という名前で Debian に入っている
  - upstream 側の Ubuntu 用のパッケージ名は [lxc-docker](http://docs.docker.io/en/latest/installation/ubuntulinux/)
- [Jenkins](http://jenkins-ci.org/)
- [jenkins-debian-glue](http://jenkins-debian-glue.org/)
- [freight](https://github.com/rcrowley/freight) : A modern take on the Debian archive.

## LT しました

さくらの VPS の Debian wheezy で IPv6 設定をした話の LT 用スライドです。

- https://github.com/znz/sakura-vps-debian-ipv6
- http://slide.rabbit-shocker.org/authors/znz/sakura-vps-debian-ipv6/
- https://speakerdeck.com/znz/sakurafalsevpsdeipv6she-ding
- (`slideshare.net` は 30分以上たっても Conversion in progress のままなので後で)

LT ということで 5 分に収まるような軽い話をしました。
内容としては [debian-users:56921](http://mla.n-z.jp/?debian-users:56921) に投稿した内容にちょっと説明を足しました。

## もくもくの会

13時からと勘違いしていたので、早めに到着して、出発前から作成していた `rd2markdown` の Web アプリ作成の続きをしていました。
もくもくの時間でも引き続き作業していました。
これは後日公開します。

## まとめ

今回の [関西Debian勉強会](https://wiki.debian.org/KansaiDebianMeeting) は今までの発表をきくのがメインだったのを、変えていっている途中でもくもくの会など、今後どういう内容にしていくのが良いのか、試行錯誤中という感じでした。
