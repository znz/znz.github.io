---
layout: post
title: "第 85 回 関西 Debian 勉強会に参加した"
date: 2014-06-22 13:43:06 +0900
comments: true
categories: event debian kansaidebian
---
[第 85 回 関西 Debian 勉強会](https://wiki.debian.org/KansaiDebianMeeting/20140622 "第 85 回 関西 Debian 勉強会")
に参加してきました。

そのときのメモです。
詳細は勉強会のページからリンクされている資料を参照してください。

<!--more-->

## Intro

- MATE 1.8 が Debian に入ったという話
  - [Debian Project News - June 9th, 2014](https://www.debian.org/News/weekly/2014/10/ "Debian Project News - June 9th, 2014")
- Debian 6 の LTS (long term support) の話
  - 全パッケージが対象というわけではない
  - たとえば rails とか chromium とかは対象外
- Berkeley DB を post jessie で外す予定
  - AGPL に変わったから
- 事前課題発表と自己紹介
  - おすすめの IM
  - webwml-git の運用
  - web-mode.el (melpa にはある)

## 「Linuxのドライバメンテナになった体験記」(担当：坂本)

- 質問から派生して残った疑問点
  - character device とは何か (block device との違いは何か)

## 「Debian での systemd とのつきあい方」(担当：佐々木)

状況確認コマンドいろいろのメモです。
一部のコマンドは自動で `$PAGER` を通してくれるようですが、
`PAGER=lv` の場合は `LV=-c` がないとエスケープシーケンスが解釈されなくて読みにくくなります。

- `systemd-analyze`
- `systemd-analyze blame`
- `systemd-analyze plot > systemd-analyze_plot.svg`
- `systemctl list-dependencies`
- `systemctl status`
- `systemctl list-unit-files`
- `systemctl list-units`
- `sudo LV=-c journalctl`
- `systemd-cgls`
