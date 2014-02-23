---
layout: post
title: "upstartに不足している機能"
date: 2014-02-23 11:15:27 +0900
comments: true
categories: upstart
---
Ubuntu の標準の init の upstart で機能不足を感じた話です。

<!--more-->

## workaround-upstart-snafu

2012年6月の頃の Ubuntu で起きた話です。
昔の話でバージョンなどもちゃんとメモしていなかったのですが、
当時サポート対象だったバージョンのどれかです。

現象としては自作の `/etc/init/hoge.conf` を作成するために
`expect` stanza を試行錯誤していたら、
設定が間違っていたらしく、
プロセスの終了をちゃんと認識できなくて、
もう存在しないプロセス ID を終了したジョブのプロセス ID として認識したまま
`start` も `stop` もきかなくなってしまった、
ということがありました。

ここまでは自分が書いた設定が間違いということで、
別に良いのですが、
その状態を解消する方法が upstart 自体には存在していなくて、
http://irclogs.ubuntu.com/2011/01/31/%23upstart.html
で紹介されていた
[workaround-upstart-snafu](https://github.com/ion1/workaround-upstart-snafu)
を使うしかありませんでした。

このスクリプトは指定したプロセス ID になるまで fork してすぐに終了するのを繰り返して
upstart に該当するプロセス ID のプロセス終了を認識させる、という仕組みのようです。

このような無理矢理な方法ではなく upstart 自体にプロセスがまだ存在しているかどうかチェックし直す機能が存在するべきだと思いました。

## Should-Start

`/etc/init.d/` に置かれる
[LSB の Init スクリプト](https://wiki.debian.org/LSBInitScripts)
には、起動時の順番 (依存関係) を指定するためのヘッダーを書くようになっています。

この指定の中には `Should-Start` というインストールされていれば先に起動する必要があり、
インストールされていなければ影響しない、ということが出来ます。

たとえば、
デーモンが依存しているデータベースが同じマシンに入っていれば
先に起動している必要があるけれど、
別のマシンのデータベースを使っているかもしれないので、
インストールされていなくても良い、
という用途が想定されているようです。

具体的な例としては
Ubuntu 13.10 の
`zabbix-server-pgsql` 2.0.6+dfsg-1ubuntu2
の
`/etc/init.d/zabbix-server`
では

```
### BEGIN INIT INFO
# Provides:          zabbix-server
# Required-Start:    $remote_fs $network
# Required-Stop:     $remote_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Should-Start:      postgresql
# Should-Stop:       postgresql
# Short-Description: Start zabbix-server daemon
# Description: Start zabbix-server daemon (PostgreSQL)
### END INIT INFO
```

となっていて、
`postgresql` より後に起動するようになっているのに、

`/etc/init/zabbix-server.conf`
では

```
start on (filesystem and net-device-up IFACE=lo)
stop on runlevel [!2345]
```

となっていて、起動順序の関係が不足しています。

その結果、
`zabbix-server` の方が `postgresql` より早く起動してしまって、
データベースに接続できなくて起動に失敗する、
ということが起きることがありました。
(以前の Ubuntu で起きていた現象で Ubuntu 13.10 だと別の現象になっているかもしれないので、
再度検証が必要そうです。)

対策として
`/etc/cron.d/local`
に

```
# zabbix-server may fail to start before postgresql
@reboot root /usr/sbin/service zabbix-server start
```

という設定を置いていて、
`cron` の `@reboot` が実行されるタイミングで起動し直すようにしています。

## まとめ

起動時の処理ということで、
実運用環境では検証しづらく、
問題が起きてもマシンの再起動時に注意していれば良いだけなので、
優先度が低くて、
検証環境を用意してまで調査するという手間をかけるまでは出来ていないのですが、
upstart はいろいろと sysvinit (insserv) に比べて劣化している部分があるようなので、
困ります。

主流が systemd に変わりつつあるという話もあるようなので、
どうなっていくのかわかりませんが、
init は OS の根幹に関わる部分なので、
もっと安定して信頼性のあるものになってほしいと思いました。
