---
layout: post
title: "関西 Debian 勉強会 + openSUSE Meetup + LILO & 東海道らぐLT大会に参加しました"
date: 2017-01-29 13:00:00 +0900
comments: true
categories: lilo event linux
---
[関西 Debian 勉強会 + openSUSE Meetup + LILO &amp; 東海道らぐLT大会](https://opensuseja.connpass.com/event/47907/ "関西 Debian 勉強会 + openSUSE Meetup + LILO &amp; 東海道らぐLT大会")
に参加しました。

<!--more-->

## 自己紹介など

- 人数が多いので、一言ずつの自己紹介と出欠チェックのための connpass のアカウントの申告でした。
- 会場費は人数が多かったのと学生が少なかったので、学生無料で社会人 100 円になりました。
- Debian の発表者の到着が遅れていたので、順番が入れ替えになりました。
- ハッシュタグは connpass のページに書いてあるように `#Kansai4cJointSession`

## Leap 42.2, 42.3とAsia Summitについて語る

- Tumbleweed : 常に最新 (ローリングリリース)
- Leap : 安定
- Leap 42 系は 42.1 が最初
- デスクトップ環境のデフォルトは存在しないが KDE Plasma を使っている人が多いらしい
- YaST は GUI, TUI でいろいろ設定できる
- パーティショニングから fstab の設定までまとめてできる
- Samba の共有の設定も簡単にできる
- YaST2 と YaST の違い : 特にない? 会場にいる人は YaST1 の頃を誰も知らなかった。 Ruby に書き換わった時も YaST3 にはならなかった。
- openSUSE.Asia Summit
- http://blog.geeko.jp/ftake/1405
- KDE の翻訳が危うい

## openSUSE Build Serviceへの愛を語る

- etckeeper や tamago などのパッケージのメンテナ
- デフォルトのファイルシステムが btrfs
- VirtualBox のゲスト OS としてインストールしたら Guest Additions が最初から入っている
- DE (デスクトップ環境) が選べるからというのがカメレオン (Geeko) の由来らしい
- 1枚のインストーラDVDで複数DE、複数ロケール対応
- dpkg 系は git build package が楽なのでは? → 会場から CI などに使える一式の環境が一発で用意できるのが良いという話
- アカウントは Novell のシングルサインオン
- OBS にログインして実演
- github の fork 的なこともできる
- ビルドされたパッケージを公開するかどうかを選べる
- ビルドされたパッケージのユーザーとしては Ubuntu PPA のように使える
- github の pull request のようなこともできる (Submit Request)
- spec ファイルに clean 処理が不要
- OBS の弱点はネットに繋がっていないとほとんど何もできない
- [Mini Debian Conference Japan 2016](http://miniconf.debian.or.jp/) の Open Build Service in Debian の資料も参照
- upstream が git だった場合の話

## Debian Updates

- 到着が遅れた話
- LaTeX Beamer のテーマを更新した話
- Debian とは?
- Debian Updates
- autoreconf の話
- autoconf-archive パッケージがあるので古い autoconf に依存しているものも動く
- Next Debian Release
- ビルドインフラが足りないからサポートアーキテクチャから落ちる話
- 新テーマ Soft waves
- Tシャツが作りにくい
- https://wiki.debian.org/DebianDesktop/Artwork/Stretch
- PHP5 が消える話
- Wayland の話
- emacs 25 があるが emacs で入るのは 24
- vim が 8
- Linux kernel は 4.9 が LTS https://lkml.org/lkml/2017/1/19/339
- KDE Connect

## 集合写真

休憩時間に集合写真を撮りました。

## LT 大会

- Debian を Windows タブレットに入れる話
- [sshでed25519鍵](https://github.com/znz/rabbit-slide-ssh-ed25519) の話をしました。([sshでed25519鍵を使うようにした話](/blog/2016-12-04-ssh-ed25519.html)と基本的に同じ内容なので、スライドサイトへの公開はしていません。)
- 16進数が好きになりました
- 年末恒例、IM飲み会に行ってみた
- Debconf 16, cape town, South Africa

## 今後の予定の告知

- https://wiki.debian.org/KansaiDebianMeeting
- https://lilo.linux.or.jp/
- [姫路IT系勉強会](https://histudy.github.io/)
