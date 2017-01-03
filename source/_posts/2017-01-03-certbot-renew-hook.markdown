---
layout: post
title: "certbot の renew hook について (その2)"
date: 2017-01-03 13:06:39 +0900
comments: true
categories: letsencrypt
---
[certbot で設定の再読み込みには post-hook よりも renew-hook を使った方が良さそうという話](/blog/2016-11-20-certbot-renew-hook.html) の続きです。

前回の記事の時点では同時に複数証明書が更新される時の挙動が確認できていませんでしたが、
確認できたので、その説明です。

<!--more-->

## 対象バージョン

- ## 対象バージョン

- Ubuntu 14.04.5 LTS (trusty)
- letsencrypt 0.9.3

## RENEWED_DOMAINS の挙動

pre-hook, post-hook, renew-hook でそれぞれ env を実行して環境変数を記録してみたところ、同時に更新される時は pre-hook, post-hook は 1 回ずつ、 renew-hook はドメインごとに呼ばれることがわかりました。

ドメインに応じてメールサーバーの reload などの処理をする場合は post-hook ではなく renew-hook を使う必要がありそうです。

ドメインなどは置き換えていますが、以下のような出力でした。
(`TMPDIR` などが `/tmp/user/0` になっているのは libpam-tmpdir を設定しているからだと思います。)

pre-hook の env:

```
TEMPDIR=/tmp/user/0
HOME=/root
OLDPWD=/root
TMPDIR=/tmp/user/0
LOGNAME=root
TEMP=/tmp/user/0
COLUMNS=80
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
CERTBOT_AUTO=/home/vpsuser/letsencrypt/letsencrypt-auto
LANG=ja_JP.UTF-8
TMP=/tmp/user/0
SHELL=/bin/sh
PWD=/
LINES=24
```

post-hook の env:

```
TEMPDIR=/tmp/user/0
HOME=/root
OLDPWD=/root
RENEWED_LINEAGE=/etc/letsencrypt/live/other.example.net
TMPDIR=/tmp/user/0
LOGNAME=root
TEMP=/tmp/user/0
COLUMNS=80
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
CERTBOT_AUTO=/home/vpsuser/letsencrypt/letsencrypt-auto
LANG=ja_JP.UTF-8
TMP=/tmp/user/0
SHELL=/bin/sh
PWD=/
RENEWED_DOMAINS=other.example.net
LINES=24
```

renew-hook の env:

```
TEMPDIR=/tmp/user/0
HOME=/root
OLDPWD=/root
RENEWED_LINEAGE=/etc/letsencrypt/live/www.example.com
TMPDIR=/tmp/user/0
LOGNAME=root
TEMP=/tmp/user/0
COLUMNS=80
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
CERTBOT_AUTO=/home/vpsuser/letsencrypt/letsencrypt-auto
LANG=ja_JP.UTF-8
TMP=/tmp/user/0
SHELL=/bin/sh
PWD=/
RENEWED_DOMAINS=www.example.com
LINES=24
```

renew-hook 2 回目:

```
TEMPDIR=/tmp/user/0
HOME=/root
OLDPWD=/root
RENEWED_LINEAGE=/etc/letsencrypt/live/other.example.net
TMPDIR=/tmp/user/0
LOGNAME=root
TEMP=/tmp/user/0
COLUMNS=80
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
CERTBOT_AUTO=/home/vpsuser/letsencrypt/letsencrypt-auto
LANG=ja_JP.UTF-8
TMP=/tmp/user/0
SHELL=/bin/sh
PWD=/
RENEWED_DOMAINS=other.example.net
LINES=24
```

## hook の例

hook を cli.ini で設定します。

```
% cat /etc/letsencrypt/cli.ini
rsa-key-size = 4096
pre-hook = /etc/letsencrypt/pre-hook
post-hook = /etc/letsencrypt/post-hook
renew-hook = /etc/letsencrypt/renew-hook
```

hook の実行ファイルを作成します。
忘れずに実行属性をつけておく必要があります。

pre-hook と post-hook は呼ばれたことと環境変数の記録だけですが、
renew-hook は呼ばれたことと環境変数の記録をしつつ、
apache に証明書の反映と、
メールサーバーのドメインの時のみ、
メール関係のデーモンにも反映をしています。
postfix だけ reload 時に標準出力にメッセージを出していたので、
捨てています。

```
% cat /etc/letsencrypt/pre-hook
#!/bin/sh
{
  date
  env
} >> /tmp/pre-hook.env
% cat /etc/letsencrypt/post-hook
#!/bin/sh
{
  date
  env
} >> /tmp/post-hook.env
% cat /etc/letsencrypt/renew-hook
#!/bin/sh
{
  date
  env
} >> /tmp/renew-hook.env
apachectl graceful
for domain in $RENEWED_DOMAINS; do
  case "$domain" in
    mx*)
      service postfix reload >/dev/null
      service dovecot reload
      ;;
  esac
done
```
