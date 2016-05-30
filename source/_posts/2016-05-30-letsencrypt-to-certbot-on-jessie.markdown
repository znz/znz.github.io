---
layout: post
title: "jessie の letsencrypt を certbot にあげてみた"
date: 2016-05-30 21:20:43 +0900
comments: true
categories: letsencrypt debian linux
---
Debian で [letsencrypt パッケージ](https://packages.debian.org/letsencrypt)が [certbot パッケージ](https://packages.debian.org/certbot)に変わって、
jessie-backports にも反映されたので、 certbot パッケージに入れ替えてみました。

<!--more-->

## 対象バージョン

- Debian GNU/Linux 8.4 (jessie) (amd64)
- letsencrypt 0.5.0-1~bpo8+1 から certbot 0.6.0-2~bpo8+1

## アップグレード失敗

普通に upgrade しようとすると以下のように失敗します。

```text
%  sudo aptitude full-upgrade -DV
以下のパッケージが更新されます:
  python-acme{b} [0.5.0-1~bpo8+1 -> 0.6.0-1~bpo8+1] (破: python-letsencrypt)
更新: 1 個、新規インストール: 0 個、削除: 0 個、保留: 0 個。
アーカイブ 54.6 k バイト中 0  バイトを取得する必要があります。展開後に 1,024  バイトのディスク領域が新たに消費されます。以下のパッケージには満たされていない依存関係があります:
 python-acme : 破壊: python-letsencrypt (< 0.6.0) [0.5.0-1~bpo8+1 が既にインストール済みです]
以下のアクションでこれらの依存関係の問題は解決されます:

     以下のパッケージを削除する:
1)     letsencrypt
2)     python-letsencrypt



この解決方法を受け入れますか? [Y/n/q/?]q
これらの依存関係の問題を解決するための努力をすべて放棄します。
中断。
```

## certbot パッケージで入れ替え

以下のように `certbot` パッケージをインストールすることで `letsencrypt` パッケージが削除されます。

```text
% sudo aptitude install certbot
以下の新規パッケージがインストールされます:
  certbot{b} python-certbot{ab}
以下のパッケージが更新されます:
  python-acme{b}
更新: 1 個、新規インストール: 2 個、削除: 0 個、保留: 0 個。
アーカイブ 204 k バイト中 149 k バイトを取得する必要があります。展開後に 816 k バイトのディスク領域が新たに消費されます 。
以下のパッケージには満たされていない依存関係があります:
 python-acme : 破壊: python-letsencrypt (< 0.6.0) [0.5.0-1~bpo8+1 が既にインストール済みです]
 python-certbot : 破壊: python-letsencrypt [0.5.0-1~bpo8+1 が既にインストール済みです]
 certbot : 破壊: letsencrypt [0.5.0-1~bpo8+1 が既にインストール済みです]
以下のアクションでこれらの依存関係の問題は解決されます:

     以下のパッケージを削除する:
1)     letsencrypt
2)     python-letsencrypt



この解決方法を受け入れますか? [Y/n/q/?]
以下の新規パッケージがインストールされます:
  certbot python-certbot{a}
以下のパッケージが削除されます:
  letsencrypt{a} python-letsencrypt{a}
以下のパッケージが更新されます:
  python-acme
更新: 1 個、新規インストール: 2 個、削除: 2 個、保留: 0 個。
アーカイブ 204 k バイト中 149 k バイトを取得する必要があります。展開後に 14.3 k バイトのディスク領域が新たに消費されます。
先に進みますか? [Y/n/?]
```

## 自動更新設定確認

インストール後に `etckeeper` の `commit` が発生して差分があるようだったので、
`sudo etckeeper vcs log -p --stat` で確認してみたところ、
`/etc/cron.d/certbot` ができていました。

```diff
diff --git a/cron.d/certbot b/cron.d/certbot
new file mode 100644
index 0000000..9c3dc35
--- /dev/null
+++ b/cron.d/certbot
@@ -0,0 +1,6 @@
+# Upstream recommends attempting renewal twice a day
+#
+# Eventually, this will be an opportunity to validate certificates
+# haven't been revoked, etc.  Renewal will only occur if expiration
+# is within 30 days.
+* */12 * * * root perl -e 'sleep int(rand(3600))'; certbot -q renew
```

## 自動更新設定変更

自動更新はログを残しつつ、 apache の reload も行う自前のスクリプトを用意していたので、
そちらを引き続き使うことにして、 `/etc/cron.d/certbot` はコメントアウトして動かないようにしました。

```diff
% sudo etckeeper vcs diff
diff --git a/cron.d/certbot b/cron.d/certbot
index 9c3dc35..e81f9aa 100644
--- a/cron.d/certbot
+++ b/cron.d/certbot
@@ -3,4 +3,4 @@
 # Eventually, this will be an opportunity to validate certificates
 # haven't been revoked, etc.  Renewal will only occur if expiration
 # is within 30 days.
-* */12 * * * root perl -e 'sleep int(rand(3600))'; certbot -q renew
+#* */12 * * * root perl -e 'sleep int(rand(3600))'; certbot -q renew
diff --git a/cron.daily/local-letsencrypt b/cron.daily/local-letsencrypt
index 8b83e29..40a23ea 100755
--- a/cron.daily/local-letsencrypt
+++ b/cron.daily/local-letsencrypt
@@ -3,7 +3,7 @@ LOGFILE=/var/log/letsencrypt/renew.log
 if [ -f "$LOGFILE" ]; then
     savelog -c 90 -q "$LOGFILE"
 fi
-if ! letsencrypt renew > "$LOGFILE" 2>&1 ; then
+if ! certbot renew > "$LOGFILE" 2>&1 ; then
     echo Automated renewal failed:
     cat "$LOGFILE"
     exit 1
```

## 現状の自動更新スクリプト

現状の自動更新スクリプト `/etc/cron.daily/local-letsencrypt` は以下のようにしています。

```bash
#!/bin/sh
LOGFILE=/var/log/letsencrypt/renew.log
if [ -f "$LOGFILE" ]; then
    savelog -c 90 -q "$LOGFILE"
fi
if ! certbot renew > "$LOGFILE" 2>&1 ; then
    echo Automated renewal failed:
    cat "$LOGFILE"
    exit 1
fi
if [ -f "$LOGFILE".0 ]; then
    diff -u "$LOGFILE".0 "$LOGFILE"
fi
apachectl graceful
```

以下のようなシンボリックリンクがあるので、 `letsencrypt renew` のままでも大丈夫そうでしたが、念のため変更しました。

```text
% ls -alF /usr/bin/letsencrypt
lrwxrwxrwx 1 root root 7  5月 28 07:30 /usr/bin/letsencrypt -> certbot*
```
