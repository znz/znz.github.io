---
layout: post
title: "rsyslogでherokuのログを記録する"
date: 2014-07-18 19:37:27 +0900
comments: true
categories: heroku rsyslog
---
heroku のログは何も設定していないと最近のログしか残らないので、
VPS で動かしている Ubuntu の rsyslog で受け取って好きなだけ
残せるように設定してみました。

<!--more-->

## 対象バージョン

- Ubuntu 12.04 LTS
- rsyslog 5.8.6-1ubuntu8.6

## rsyslog の設定

rsyslog でリモートからのログを受け取って `/var/log/remote` 以下に記録する設定をします。

ポート番号は外からの攻撃を減らすためや rsyslog の起動時に root 権限を不要にするために、
デフォルトの 514 ではなく適当なポート番号に変更しました。

リモートからのログはここで日付とホスト名から決まるファイル名のログに記録して、
ローカルのログと混ざらないようにしています。

ファイル名は日付を前にするか、送信元ホスト名を前にするかは好みで決めると良いと思います。

```text /etc/rsyslog.d/10-local.conf
# provides UDP syslog reception
$ModLoad imudp
$UDPServerRun 51514

# provides TCP syslog reception
$ModLoad imtcp
$InputTCPServerRun 51514

$template RemoteLog,"/var/log/remote/%$year%%$month%%$day%_%hostname%.log"
:fromhost-ip,!isequal,"127.0.0.1" ?RemoteLog
& ~
```

## `/var/log/remote` 作成

ディレクトリを自動で作成はしてくれないので、あらかじめ作成しておきます。
同じ理由でログファイルをディレクトリ分けすることは出来ませんでした。

syslog ユーザーが書き込めて adm グループで読めるようにしました。

    sudo install -o syslog -g adm -m 2750 -d /var/log/remote

## ufw 設定

`ufw allow 51514` で外から udp も tcp も受け付けるようにします。

## heroku からの送信設定

`heroku drains` コマンドで設定しました。

Heroku 側のドキュメントは
[Syslog drains](https://devcenter.heroku.com/articles/logging#syslog-drains "Syslog drains")
にありますが、情報が少ないので、同様のログを蓄積するアドオンの設定を参考にするのが良いようです。

```console
% heroku drains
No drains for this app
% heroku drains:add
 !    Usage: heroku drains:add URL
zsh: exit 1     heroku drains:add
% heroku drains:add syslog://syslog.example.jp:51514
Successfully added drain syslog://syslog.example.jp:51514
% heroku drains
syslog://syslog.example.jp:51514 (d.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx)
```
