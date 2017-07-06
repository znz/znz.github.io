---
layout: post
title: "Hyper-V のゲスト環境で systemd timer がうまく動いていなかった"
date: 2017-07-07 00:30:00 +0900
comments: true
categories: linux debian ubuntu windows
---
Hyper-V のゲストとしてインストールした Linux 環境で systemd timer の `RandomizedDelaySec` がおかしくて timer が実行されないことがあったのですが、Hyper-V の時刻の同期との相性が悪いのが原因でした。

<!--more-->

## 環境

- ホスト: Windows Server 2012
- ゲスト: Ubuntu 16.04.2 LTS (xenial) と Debian GNU/Linux 9.0 (stretch)

## 現象

journalctl で過去のログを確認してみると

```
 6月 28 12:07:56 hostname systemd[9928]: Time has been changed
 6月 28 12:07:59 hostname systemd[9928]: Time has been changed
 6月 28 12:08:04 hostname systemd[9928]: Time has been changed
 6月 28 12:08:09 hostname systemd[9928]: Time has been changed
 6月 28 12:08:14 hostname systemd[9928]: Time has been changed
 6月 28 12:08:19 hostname systemd[9928]: Time has been changed
```

のように Time has been changed が頻繁に記録されていました。

再起動した後からは RandomizedDelaySec が設定されている timer のランダムな時間挿入が Time has been changed の直後におきていました。

```
 6月 28 15:07:51 hostname systemd[1]: Time has been changed
 6月 28 15:07:51 hostname systemd[1]: apt-daily-upgrade.timer: Adding 46min 16.478521s random time.
 6月 28 15:07:51 hostname systemd[1]: apt-daily.timer: Adding 3h 45min 54.621700s random time.
 6月 28 15:07:56 hostname systemd[1]: Time has been changed
 6月 28 15:07:56 hostname systemd[1]: apt-daily-upgrade.timer: Adding 25min 59.320458s random time.
 6月 28 15:07:56 hostname systemd[1]: apt-daily.timer: Adding 11h 34min 9.012513s random time.
 6月 28 15:08:01 hostname systemd[1]: Time has been changed
 6月 28 15:08:01 hostname systemd[1]: apt-daily-upgrade.timer: Adding 42min 37.932995s random time.
 6月 28 15:08:01 hostname systemd[1]: apt-daily.timer: Adding 4h 48min 31.255279s random time.
 6月 28 15:08:06 hostname systemd[1]: Time has been changed
 6月 28 15:08:06 hostname systemd[1]: apt-daily-upgrade.timer: Adding 13min 44.192537s random time.
 6月 28 15:08:06 hostname systemd[1]: apt-daily.timer: Adding 38min 56.349412s random time.
```

自作した timer が実行されなくて `journalctl -u local-backup.timer` のように調べた時に「Adding ... random time.」のログで埋まっていて、他の動いている timer との違いも特になくて悩んでいましたが、ふと `journalctl` (引数なし) を実行してみたら「Time has been changed」とセットでおきていることに気づきました。

## 解決策

「Time has been changed」で検索して最初に出てきた [16.04 - /var/log/syslog 'systemd\[1\]: Time has been changed' message every 5 seconds - Ask Ubuntu](https://askubuntu.com/questions/888493/var-log-syslog-systemd1-time-has-been-changed-message-every-5-seconds) に

> I encountered this issue of "systemd[...]Time has been changed" messages logged every five seconds in /var/log/syslog on a 16.04 server running under Windows 8.1 Hyper-V. To fix it, I disabled time synchronization on the Hyper-V side. In Hyper-V Manager, I highlighted the VM, selected "Settings...", then "Integration Services", unchecked "Time synchronization", and clicked Apply. The messages stopped instantly - no VM restart was required.

と書いてあったので、「設定...」から「統合サービス」の「時刻の同期」のチェックを外して (再起動なしで) 解決しました。

## 解決確認

「Time has been changed」も出なくなって「Adding ... random time.」も出なくなって、翌日まで待ってみるとちゃんと実行されていたので、解決したようです。

## まとめ

ntp サーバー機能も必要で、 ntp パッケージを入れている環境で発生したので、時刻の同期の仕方によっては発生しないのかもしれませんが、「Time has been changed」で検索して出てきた他の方法はログを無視するだけとか、根本的な解決になっていないものが多そうだったので、Hyper-V との組み合わせなら systemd-timesyncd でも発生するのかもしれません。
