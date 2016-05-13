---
layout: post
title: "letsencrypt-auto が certbot-auto になった"
date: 2016-05-13 23:40:09 +0900
comments: true
categories: letsencrypt linux
---
[EFF の Let's Encrypt クライアントが Certbot になった](https://www.eff.org/deeplinks/2016/05/announcing-certbot-new-tls-robot)という話です。

<!--more-->

## 経緯

[Let's Encrypt の使い方](https://letsencrypt.jp/usage/) から引用しつつまとめます。

- 2016年4月12日 に、Let's Encrypt の公開ベータプログラム（Public Beta Program）が終了し、正式サービスが開始されました。
- この時点でベータがとれたのは Let's Encrypt のサービス側で `letsencrypt-auto` コマンドを含む github.com/letsencrypt/letsencrypt にあったクライアントはまだベータのままでした。
- クライアントの 0.6.0 のリリースにあたり github.com/letsencrypt/letsencrypt は github.com/certbot/certbot に移動して Certbot に改名されました。
- [Announcing Certbot: EFF's Client for Let's Encrypt](https://www.eff.org/deeplinks/2016/05/announcing-certbot-new-tls-robot) に書いてあるように Certbot もまだベータで、今年中に Certbot 1.0 のリリースが予定されているようです。

改名されたのは EFF の Let's Encrypt クライアントだけでサービス自体は Let's Encrypt という名前のままです。

## その他の変更点

インストール方法も `git clone` して `letsencrypt-auto` を実行する方法から、 `https://dl.eff.org/certbot-auto` をダウンロードして実行する方法に変わっています。

## インストール済み環境への影響

インストール済み環境ではどうなるのかと思って、 `letsencrypt-auto --help` を実行してアップグレードさせてみたところ、以下のようになりました。

出力をよく見ると `sudo` のところで `CERTBOT_AUTO` という環境変数の指定が増えていて、 `letsencrypt-auto` コマンド自体はそのまま使えるようです。

```
% ~/letsencrypt/letsencrypt-auto --help
Checking for new version...
Upgrading letsencrypt-auto 0.5.0 to 0.6.0...
Replacing letsencrypt-auto...
   sudo cp -p /home/vpsuser/letsencrypt/letsencrypt-auto /tmp/user/1000/tmp.AiqGtrjioJ/letsencrypt-auto.permission-clone
   sudo cp /tmp/user/1000/tmp.AiqGtrjioJ/letsencrypt-auto /tmp/user/1000/tmp.AiqGtrjioJ/letsencrypt-auto.permission-clone
   sudo mv -f /tmp/user/1000/tmp.AiqGtrjioJ/letsencrypt-auto.permission-clone /home/vpsuser/letsencrypt/letsencrypt-auto
Creating virtual environment...
Installing Python packages...
Installation succeeded.
Requesting root privileges to run certbot...
   sudo CERTBOT_AUTO=/home/vpsuser/letsencrypt/letsencrypt-auto /home/vpsuser/.local/share/letsencrypt/bin/letsencrypt --help

  letsencrypt-auto [SUBCOMMAND] [options] [-d domain] [-d domain] ...

Certbot can obtain and install HTTPS/TLS/SSL certificates.  By default,
it will attempt to use a webserver both for obtaining and installing the
cert. Major SUBCOMMANDS are:

  (default) run        Obtain & install a cert in your current webserver
  certonly             Obtain cert, but do not install it (aka "auth")
  install              Install a previously obtained cert in a server
  renew                Renew previously obtained certs that are near expiry
  revoke               Revoke a previously obtained certificate
  rollback             Rollback server configuration changes made during install
  config_changes       Show changes made to server config during installation
  plugins              Display information about installed plugins

Choice of server plugins for obtaining and installing cert:

  --apache          Use the Apache plugin for authentication & installation
  --standalone      Run a standalone webserver for authentication
  (nginx support is experimental, buggy, and not installed by default)
  --webroot         Place files in a server's webroot folder for authentication

OR use different plugins to obtain (authenticate) the cert and then install it:

  --authenticator standalone --installer apache

More detailed help:

  -h, --help [topic]    print this message, or detailed help on a topic;
                        the available topics are:

   all, automation, paths, security, testing, or any of the subcommands or
   plugins (certonly, install, nginx, apache, standalone, webroot, etc)
```
