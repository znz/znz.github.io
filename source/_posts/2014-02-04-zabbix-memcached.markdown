---
layout: post
title: "zabbixでmemcachedのヒット率を記録する"
date: 2014-02-04 19:14:01 +0900
comments: true
categories: zabbix memcached
---
rails の dalli_store でキャッシュの保存場所として使っている memcached のキャッシュのヒット率を zabbix で記録して、コードの改善でどのくらい変わるのか調べてみることにしました。

<!--more-->

## 参考

memcached からのデータ取得方法は [zabbixでmemcached監視 at technote](http://blog.withsin.net/2010/09/06/zabbix%E3%81%A7memcached%E7%9B%A3%E8%A6%96/) を参考にしました。

## 前提

zabbix-server は別サーバーで既に設定済みで動いているものを使いました。
zabbix-agent は新規インストールしました。
OS は Ubuntu 12.04.4 LTS です。

## memcached からデータ取得部分

`/etc/cron.d/zabbix-memcached` に以下の内容で取得部分を作成しました。
継続的にエラーが出ている時ぐらいは原因を調べられるように、標準エラー出力も最新のものだけファイルに残すようにしました。

```text /etc/cron.d/zabbix-memcached
*/5 * * * *   zabbix    test -d /run/zabbix && { echo stats | nc localhost 11211 > /run/zabbix/memcached.stats 2>/run/zabbix/memcached.stats.err; }
```

## zabbix_agentd 設定

以下の設定を変更しました。

- Server=`zabbix-serverのIPアドレス`
- Hostname=`zabbix の Web UI のホスト名にあわせる`
- DisableActive=1 : zabbix-agent 側からの接続ができない環境だったので、 zabbix-server からの接続を待ち受けるだけにしました。
- `UserParameter=memcached.stats[*],grep $1 /run/zabbix/memcached.stats | cut -d' ' -f3` : grep と組み合わせるには awk は重い気がしたので cut に変更しました。

## firewall 設定

`ufw allow from zabbix-serverのIPアドレス to any port 10050` で zabbix-server からのみ許可しました。
インターネット上を平文で流れてしまうのは、機密データではないということで、許容することにしました。
暗号化が必要なら別途 openvpn などを使うと思います。

## 動作確認

- Server に localhost 以外を指定したので zabbix-agent を動かしているマシン上で `nc -v localhost 10050` は繋がった後、すぐに切断されます。
- zabbix-server 側で `zabbix_get -s 49.212.178.112 -k agent.ping` を実行して `1` と出れば zabbix-agent へ接続できています。
- zabbix の Web UI からテンプレートを作成して、 `cmd_get` `cmd_set` `cmd_hits` を記録するようにしました。
  (キー: `memcached.stats[cmd_get]` など, 保存時の計算: 差分, 更新間隔: 300 (cronを5分間隔にしているので))
- さらに計算アイテムとして式に `100*last("memcached.stats[get_hits]")/last("memcached.stats[cmd_get]")` を設定したものを追加しました。

差分で記録しているからなのか、データが少ないタイミングだったからか、計算に使うアイテムのタイミングの問題なのかわかりませんが、計算アイテムのヒット率が 100% を超えることがありました。
