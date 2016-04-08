---
layout: post
title: "letsencrypt-auto の自動アップグレードを止めて手動でアップグレード"
date: 2016-04-07 23:00:00 +0900
comments: true
categories: letsencrypt ubuntu linux
---
`/etc/cron.daily/letsencrypt` は `root` 権限で実行されるため、そこで自動アップグレードがかかるとファイルのオーナーが `root` になってしまうかもしれないと思ったので、自動更新を止めて手動でアップグレードするようにしました。

<!--more-->

## 自動アップグレードを止めた `/etc/cron.daily/local-letsencrypt`

[前回の記事](http://blog.n-z.jp/blog/2016-03-06-letsencrypt.html) からの差分としては [debianutils](http://packages.debian.org/debianutils) の `savelog` でログをローテートして、証明書の有効期限の 90 日分残すようにしたのと、 `--no-self-upgrade` をつけて自動アップグレードを止めたことです。
それから、パッケージで入れたものではないということを明示するために `local` という文字列を入れたファイル名に変更しました。


```bash /etc/cron.daily/local-letsencrypt
#!/bin/sh
LOGFILE=/var/log/letsencrypt/renew.log
if [ -f "$LOGFILE" ]; then
    /usr/bin/savelog -c 90 -q "$LOGFILE"
fi
if ! /home/hoge/letsencrypt/letsencrypt-auto renew --no-self-upgrade > "$LOGFILE" 2>&1 ; then
    echo Automated renewal failed:
    cat "$LOGFILE"
    exit 1
fi
apachectl graceful
service postfix reload >/dev/null
service dovecot reload
```

## 手動アップグレードしたログ

手動実行したところ、ちょうど 0.4.2 から 0.5.0 へのアップグレードが実行されました。

```
% ~/letsencrypt/letsencrypt-auto --help
Checking for new version...
Upgrading letsencrypt-auto 0.4.2 to 0.5.0...
Replacing letsencrypt-auto...
   sudo cp -p /home/hoge/letsencrypt/letsencrypt-auto /tmp/user/1000/tmp.MclJH3TO68/letsencrypt-auto.permission-clone
   sudo cp /tmp/user/1000/tmp.MclJH3TO68/letsencrypt-auto /tmp/user/1000/tmp.MclJH3TO68/letsencrypt-auto.permission-clone
   sudo mv -f /tmp/user/1000/tmp.MclJH3TO68/letsencrypt-auto.permission-clone /home/hoge/letsencrypt/letsencrypt-auto
Creating virtual environment...
Installing Python packages...
Installation succeeded.
Requesting root privileges to run letsencrypt...
   sudo /home/hoge/.local/share/letsencrypt/bin/letsencrypt --help

  letsencrypt-auto [SUBCOMMAND] [options] [-d domain] [-d domain] ...

The Let's Encrypt agent can obtain and install HTTPS/TLS/SSL certificates.  By
default, it will attempt to use a webserver both for obtaining and installing
the cert. Major SUBCOMMANDS are:

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

## アップグレードがないときのログ

```
% ~/letsencrypt/letsencrypt-auto --help
Checking for new version...
Requesting root privileges to run letsencrypt...
   sudo /home/hoge/.local/share/letsencrypt/bin/letsencrypt --help

  letsencrypt-auto [SUBCOMMAND] [options] [-d domain] [-d domain] ...

The Let's Encrypt agent can obtain and install HTTPS/TLS/SSL certificates.  By
default, it will attempt to use a webserver both for obtaining and installing
the cert. Major SUBCOMMANDS are:

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
