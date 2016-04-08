---
layout: post
title: "jessie に backports から letsencrypt を入れてみた"
date: 2016-04-08 14:30:02 +0900
comments: true
categories: letsencrypt debian linux
---
現在リリースされている Ubuntu と違って Debian jessie には backports に letsencrypt パッケージがあるので、ちょっと古いですがパッケージ版の letsencrypt を使ってみることにしました。

Ubuntu も今月リリースされる 16.04 (xenial) には universe ですが letsencrypt パッケージが含まれるので、それが使えると思います。

<!--more-->

## 対象バージョン

- Debian GNU/Linux 8.4 (jessie) (amd64)
- letsencrypt 0.4.1-1~bpo8+1
- apache2 2.4.10-10+deb8u4

## インストール

`/etc/apt/sources.list` で `deb http://ftp.jp.debian.org/debian jessie-backports main contrib non-free` のように backports を有効にしておきます。

依存パッケージも backports のものが必要なので `-t jessie-backports` 付きでインストールする必要がありました。

`webroot` を使う予定だったので、 `python-letsencrypt-apache` はインストールしませんでした。

stable にあるパッケージのうち、いくつかのパッケージも backports のものに上がってしまうので、そのリスクが許容できない場合は `letsencrypt-auto` など、他のインストール方法を検討した方が良さそうです。

```
%  sudo apt install letsencrypt
パッケージリストを読み込んでいます... 完了
依存関係ツリーを作成しています
状態情報を読み取っています... 完了
インストールすることができないパッケージがありました。おそらく、あり得
ない状況を要求したか、(不安定版ディストリビューションを使用しているの
であれば) 必要なパッケージがまだ作成されていなかったり Incoming から移
動されていないことが考えられます。
以下の情報がこの問題を解決するために役立つかもしれません:

以下のパッケージには満たせない依存関係があります:
 letsencrypt : 依存: python-letsencrypt (= 0.4.1-1~bpo8+1) しかし、インストールされようとしていませ ん
E: 問題を解決することができません。壊れた変更禁止パッケージがあります。
zsh: exit 100   sudo apt install letsencrypt
%  sudo apt install -t jessie-backports letsencrypt
パッケージリストを読み込んでいます... 完了
依存関係ツリーを作成しています
状態情報を読み取っています... 完了
以下のパッケージが自動でインストールされましたが、もう必要とされていません:
  python-cffi python-ply python-pycparser
これを削除するには 'apt-get autoremove' を利用してください。
以下の追加パッケージがインストールされます:
  dialog python-acme python-cffi python-cffi-backend python-configargparse python-configobj
  python-cryptography python-dialog python-enum34 python-funcsigs python-idna python-ipaddress
  python-letsencrypt python-mock python-ndg-httpsclient python-openssl python-parsedatetime
  python-pbr python-psutil python-pyasn1 python-pyicu python-requests python-rfc3339 python-six
  python-urllib3 python-zope.component python-zope.event python-zope.interface
提案パッケージ:
  python-letsencrypt-apache python-letsencrypt-doc python-configobj-doc python-cryptography-doc
  python-cryptography-vectors python-enum34-doc python-funcsigs-doc python-mock-doc
  python-openssl-doc python-openssl-dbg doc-base python-ntlm
以下のパッケージが新たにインストールされます:
  dialog letsencrypt python-acme python-cffi-backend python-configargparse python-configobj
  python-dialog python-enum34 python-funcsigs python-idna python-ipaddress python-letsencrypt
  python-mock python-ndg-httpsclient python-parsedatetime python-pbr python-psutil python-pyasn1
  python-pyicu python-requests python-rfc3339 python-urllib3 python-zope.component
  python-zope.event python-zope.interface
以下のパッケージはアップグレードされます:
  python-cffi python-cryptography python-openssl python-six
アップグレード: 4 個、新規インストール: 25 個、削除: 0 個、保留: 24 個。
1,906 kB のアーカイブを取得する必要があります。
この操作後に追加で 8,772 kB のディスク容量が消費されます。
続行しますか? [Y/n]
```

## letsencrypt コマンド実行

一般ユーザー権限で実行するとエラーになり、カレントディレクトリに `letsencrypt.log` が作成されていました。
`--help` 付きでの実行は特にエラーもなく、 `letsencrypt.log` も作成されることなくヘルプが表示されました。

```
%  letsencrypt
An unexpected error occurred:
OSError: [Errno 13] Permission denied: '/etc/letsencrypt'
Please see the logfile 'letsencrypt.log' for more details.
%  rm letsencrypt.log
%  letsencrypt --help

  letsencrypt [SUBCOMMAND] [options] [-d domain] [-d domain] ...

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

  (the apache plugin is not installed)
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

## 本番実行

`letsencrypt certonly` で証明書発行します。

```
%  sudo letsencrypt certonly --webroot -w /srv/www/www.example.org/htdocs -d www.example.org
```

まず、アカウントの作成があるので、アカウントの作成はメールアドレス入力しました。
アカウントのリカバリや緊急時の連絡などに使われるだけのようで、今の所ここで入力したメールアドレスに letsencrypt からメールが来たことはありません。

             ┌──────────────────────────────────────────────────────────────────────┐
             │ Enter email address (used for urgent notices and lost key recovery)  │
             │ ┌──────────────────────────────────────────────────────────────────┐ │
             │ │                                                                  │ │
             │ └──────────────────────────────────────────────────────────────────┘ │
             ├──────────────────────────────────────────────────────────────────────┤
             │                     < 了解 >           < 取消 >                      │
             └──────────────────────────────────────────────────────────────────────┘

Terms of Service は前回みた時から変わっていないので、今度も Agree しました。

             ┌──────────────────────────────────────────────────────────────────────┐
             │ Please read the Terms of Service at                                  │
             │ https://letsencrypt.org/documents/LE-SA-v1.0.1-July-27-2015.pdf. You │
             │ must agree in order to register with the ACME server at              │
             │ https://acme-v01.api.letsencrypt.org/directory                       │
             │                                                                      │
             │                                                                      │
             │                                                                      │
             │                                                                      │
             │                                                                      │
             │                                                                      │
             │                                                                      │
             │                                                                      │
             │                                                                      │
             │                                                                      │
             │                                                                      │
             │                                                                      │
             ├──────────────────────────────────────────────────────────────────────┤
             │                     <Agree >           <Cancel>                      │
             └──────────────────────────────────────────────────────────────────────┘

以下のような作成完了のメッセージが出ました。

```
IMPORTANT NOTES:
 - If you lose your account credentials, you can recover through
   e-mails sent to z@n-z.jp.
 - Congratulations! Your certificate and chain have been saved at
   /etc/letsencrypt/live/www.example.org/fullchain.pem. Your cert will
   expire on 2016-07-07. To obtain a new version of the certificate in
   the future, simply run Let's Encrypt again.
 - Your account credentials have been saved in your Let's Encrypt
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Let's
   Encrypt so making regular backups of this folder is ideal.
 - If you like Let's Encrypt, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

問題があれば `/var/log/letsencrypt/letsencrypt.log` でログを確認します。

## apache2 の設定変更

apache2 の設定を変更して、 `sudo service apache2 reload` で反映します。
ブラウザーで Let's Encrypt Authority X3 の証明書になっていることが確認できたら設定完了です。

```
SSLCertificateKeyFile /etc/letsencrypt/live/www.example.org/privkey.pem
SSLCertificateFile /etc/letsencrypt/live/www.example.org/fullchain.pem
```

## 自動更新設定

パッケージの `letsencrypt` でインストールされたものではないということを明示するために `local` をつけて `/etc/cron.daily/local-letsencrypt` に自動更新の設定をしました。

試しに実行してみてちゃんと動いていれば設定完了です。

```
% sudoedit /etc/cron.daily/local-letsencrypt
% sudo chmod +x /etc/cron.daily/local-letsencrypt
% sudo /etc/cron.daily/local-letsencrypt
% sudo cat /var/log/letsencrypt/renew.log
Processing /etc/letsencrypt/renewal/www.example.org.conf

The following certs are not due for renewal yet:
  /etc/letsencrypt/live/www.example.org/fullchain.pem (skipped)
No renewals were attempted.
```

[前回の記事](http://blog.n-z.jp/blog/2016-04-07-letsencrypt.html) のように [debianutils](http://packages.debian.org/debianutils) の `savelog` でログをローテートして、証明書の有効期限の 90 日分残すようにしています。

```bash /etc/cron.daily/local-letsencrypt
#!/bin/sh
LOGFILE=/var/log/letsencrypt/renew.log
if [ -f "$LOGFILE" ]; then
    savelog -c 90 -q "$LOGFILE"
fi
if ! letsencrypt renew > "$LOGFILE" 2>&1 ; then
    echo Automated renewal failed:
    cat "$LOGFILE"
    exit 1
fi
apachectl graceful
```
