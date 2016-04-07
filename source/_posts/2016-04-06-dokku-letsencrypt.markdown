---
layout: post
title: "dokku-letsencrypt を使ってみた"
date: 2016-04-06 23:00:00 +0900
comments: true
categories: dokku letsencrypt ubuntu linux
---
[dokku-letsencrypt](https://github.com/dokku/dokku-letsencrypt) を試してみたのでそのメモです。

<!--more-->

## 対象バージョン

- Ubuntu 14.04.4 LTS
- Docker 1.10.3
- Dokku 0.5.3
- dokku-letsencrypt v0.7.0-7-gb4950b8

## インストール

README.md に書いてある手順の通りインストールして、 `git describe --tags` でバージョンを確認しておきました。

```
$ sudo dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git
-----> Cloning plugin repo https://github.com/dokku/dokku-letsencrypt.git to /var/lib/dokku/plugins/available/letsencrypt
Cloning into 'letsencrypt'...
remote: Counting objects: 233, done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 233 (delta 0), reused 0 (delta 0), pack-reused 229
Receiving objects: 100% (233/233), 48.62 KiB | 0 bytes/s, done.
Resolving deltas: 100% (136/136), done.
Checking connectivity... done.
-----> Plugin letsencrypt enabled
-----> Migrating zero downtime env variables. The following variables have been deprecated
=====> DOKKU_SKIP_ALL_CHECKS DOKKU_SKIP_DEFAULT_CHECKS
=====> Please use dokku checks:[disable|enable] <app> to control zero downtime functionality
=====> Migration complete
=====>
=====> Migration complete
=====>
=====> Migration complete
=====>
=====> Migration complete
=====>
=====> Migration complete
=====>
Adding user dokku to group adm
$ cd /var/lib/dokku/plugins/available/letsencrypt/
$ git describe --tags
v0.7.0-7-gb4950b8
$ cd
```

## アップグレード

README.md にアップグレードの手順も書いてあったので、試しておきました。

```
$ sudo dokku plugin:update letsencrypt
Plugin (letsencrypt) updated
```

## 対象アプリの確認

`dokku apps` でアプリケーション一覧を表示して、対象とするアプリケーションの名前を確認しておきました。

- `dokku help`
- `dokku apps:help`
- `dokku apps`

## メールアドレス設定

Let's Encrypt に登録するメールアドレスを設定しておきます。
[Let's Encrypt の使い方](https://letsencrypt.jp/usage/ "Let's Encrypt の使い方")の説明によると「ここで入力したメールアドレスは、緊急の通知や鍵を紛失したときの復旧に使われます。」

dokku-letsencrypt プラグインでは「利用規約への同意」に相当する手順がありませんが、念のため利用規約 (現在のバージョンは https://letsencrypt.org/documents/LE-SA-v1.0.1-July-27-2015.pdf ) に目を通しておくと良いと思います。

ちなみに今のところ letsencrypt に登録したメールアドレスに letsencrypt からメールが来たことはありません。

```
$ dokku config:set --no-restart staging.example.co.jp DOKKU_LETSENCRYPT_EMAIL=root@example.co.jp
-----> Setting config vars
       DOKKU_LETSENCRYPT_EMAIL: root@example.co.jp
```

## メールアドレスをグローバルに設定するかアプリケーションごとに設定するか

グローバルに設定することも可能だと思いますが、メールアドレスを設定していなければ `dokku letsencrypt APP` の最初のチェックで止まって、既存の TLS 設定を上書きされる心配がないので、すべてのアプリケーションで letsencrypt を使うのでなければ、アプリケーションごとに設定することをおすすめします。

メールアドレスを設定していなければ、以下のように失敗して止まってくれます。

```
$ dokku letsencrypt node-js-app
=====> Let's Encrypt node-js-app...
 !     ERROR: Cannot request a certificate without an e-mail address!
 !       please provide your e-mail address using
 !       dokku config:set --no-restart node-js-app DOKKU_LETSENCRYPT_EMAIL=<e-mail>
```

## 証明書発行と設定

`dokku letsencrypt APP` で証明書発行から設定まで自動で実行されます。

すでに `tls/server.{crt,key}` が存在していても強制的にシンボリックリンクで上書きされるので、他で発行された証明書を使っている場合は注意が必要です。

```
$ dokku letsencrypt staging.example.co.jp
=====> Let's Encrypt staging.example.co.jp...
-----> Updating letsencrypt docker image...
latest: Pulling from m3adow/letsencrypt-simp_le
420890c9e918: Pull complete
acbaf1e6012f: Pull complete
5f71a1a2d3dc: Pull complete
Digest: sha256:be1d7aca214d5277af18d7bf75a2bc78afa5a1eabf98aaa8a606c4ca2a7fdeb5
Status: Downloaded newer image for m3adow/letsencrypt-simp_le:latest
       done
-----> Enabling ACME proxy for staging.example.co.jp...
-----> Getting letsencrypt certificate for staging.example.co.jp...
        - Domain 'staging.example.co.jp'
darkhttpd/1.11, copyright (c) 2003-2015 Emil Mikulic.
listening on: http://0.0.0.0:80/
2016-04-04 03:26:42,946:INFO:__main__:1202: Generating new account key
2016-04-04 03:26:43,831:INFO:requests.packages.urllib3.connectionpool:758: Starting new HTTPS connection (1): acme-v01.api.letsencrypt.org
2016-04-04 03:26:44,110:INFO:requests.packages.urllib3.connectionpool:758: Starting new HTTPS connection (1): acme-v01.api.letsencrypt.org
2016-04-04 03:26:44,302:INFO:requests.packages.urllib3.connectionpool:758: Starting new HTTPS connection (1): acme-v01.api.letsencrypt.org
2016-04-04 03:26:44,841:INFO:requests.packages.urllib3.connectionpool:758: Starting new HTTPS connection (1): letsencrypt.org
2016-04-04 03:26:45,410:INFO:requests.packages.urllib3.connectionpool:758: Starting new HTTPS connection (1): acme-v01.api.letsencrypt.org
2016-04-04 03:26:45,664:INFO:requests.packages.urllib3.connectionpool:758: Starting new HTTPS connection (1): acme-v01.api.letsencrypt.org
2016-04-04 03:26:45,940:INFO:requests.packages.urllib3.connectionpool:207: Starting new HTTP connection (1): staging.example.co.jp
2016-04-04 03:26:45,946:INFO:requests.packages.urllib3.connectionpool:758: Starting new HTTPS connection (1): staging.example.co.jp
2016-04-04 03:26:45,995:INFO:__main__:1294: staging.example.co.jp was successfully self-verified
2016-04-04 03:26:46,022:INFO:requests.packages.urllib3.connectionpool:758: Starting new HTTPS connection (1): acme-v01.api.letsencrypt.org
2016-04-04 03:26:46,272:INFO:__main__:1302: Generating new certificate private key
2016-04-04 03:26:47,528:INFO:requests.packages.urllib3.connectionpool:758: Starting new HTTPS connection (1): acme-v01.api.letsencrypt.org
2016-04-04 03:26:47,723:INFO:requests.packages.urllib3.connectionpool:758: Starting new HTTPS connection (1): acme-v01.api.letsencrypt.org
2016-04-04 03:26:47,987:INFO:requests.packages.urllib3.connectionpool:758: Starting new HTTPS connection (1): acme-v01.api.letsencrypt.org
2016-04-04 03:26:48,215:INFO:__main__:385: Saving account_key.json
2016-04-04 03:26:48,216:INFO:__main__:385: Saving fullchain.pem
2016-04-04 03:26:48,216:INFO:__main__:385: Saving chain.pem
2016-04-04 03:26:48,216:INFO:__main__:385: Saving cert.pem
2016-04-04 03:26:48,216:INFO:__main__:385: Saving key.pem
-----> Certificate retrieved successfully.
-----> Symlinking let's encrypt certificates
-----> Configuring staging.example.co.jp...(using built-in template)
-----> Creating https nginx.conf
-----> Running nginx-pre-reload
       Reloading nginx
-----> Disabling ACME proxy for staging.example.co.jp...
       done
```

## 有効になっているアプリケーション一覧確認

`dokku letsencrypt:ls` で有効になっているアプリケーションとその有効期限を確認します。

```
$ dokku letsencrypt:ls
-----> App name           Certificate Expiry        Time before expiry        Time before renewal
staging.example.co.jp 2016-07-03 11:27:00       89d, 22h, 56m, 55s        59d, 22h, 56m, 55s
```

## 自動更新

有効期限が 30 日 (`DOKKU_LETSENCRYPT_GRACEPERIOD` で変更可能) を切ると自動更新してくれる `dokku letsencrypt:auto-renew` も試しておきます。

```
$ dokku letsencrypt:auto-renew
=====> Auto-renewing all apps...
       staging.example.co.jp still has 59d, 22h, 48m, 36s days left before renewal
=====> Finished auto-renewal
```

問題なさそうなので、`dokku` ユーザーの `crontab` で設定して自動実行するようにしておきます。
リモートからのトリガーで実行されるように ssh で入れるユーザーの `crontab` で `ssh dokku letsencrypt:auto-renew` を設定しておくのでも良いと思います。

## セキュリティ上の問題点

`dokku-letsencrypt` が使用している [Simple Let's Encrypt Client](https://github.com/kuba/simp_le "Simple Let's Encrypt Client") の issue の [private key permissions](https://github.com/kuba/simp_le/issues/29 "private key permissions") で指摘されているように、 `ls -al /home/dokku/staging.example.co.jp/letsencrypt/certs/current/` で確認してみると、他のユーザーからは読めなくするべき `account_key.json` や `key.pem` も誰でも読めるパーミッションになってしまっているので、 `sudo chmod 700 /home/dokku/staging.example.co.jp/letsencrypt` などでパーミッションを落としておく方が良さそうです。

ファイル自体のパーミッションを落としても良さそうですが、更新された後のことも考えると `/home/dokku/APP/letsencrypt` ディレクトリ自体のパーミッションを落としておくのが良さそうです。

## Rate Limit

[Let's Encrypt の証明書に取得数制限はありますか？](https://letsencrypt.jp/faq/#RateLimiting "Let's Encrypt の証明書に取得数制限はありますか？") のリンク先に書いてあるように、この記事執筆時点では「アカウント登録/IP アドレスごと」(3 時間で 10 個) と「証明書発行/ドメインごと」(1 週間で 5 個) の制限があるので、注意が必要です。

特に dokku-letsencrypt では[公式のクライアント](https://github.com/letsencrypt/letsencrypt)が `/etc/letsencrypt/accounts` でアカウントを共有するのと違って、 `account_key.json` をアプリケーションごとに作成しているので、注意が必要そうです。

ただし、現状の制限だと証明書発行数の制限の方が引っかかりやすいので、アカウント登録の制限は問題にならないようにも思います。

証明書発行数の制限については `dokku domains:add` や `dokku domains:remove` で適切にドメインの追加や削除をしてから `dokku letsencrypt` を実行するように README.md の [Dealing with rate limit](https://github.com/dokku/dokku-letsencrypt/tree/b4950b8254f683e4af775bad44e390763a699de1#dealing-with-rate-limit "Dealing with rate limit") に書いてあります。

## 証明書の情報表示

`dokku certs:info` で letsencrypt のものに限らず、証明書の情報を表示できます。

```
adminuser@tk2-213-16013:~$ dokku certs:info staging.example.co.jp
-----> Fetching SSL Endpoint info for staging.example.co.jp...
-----> Certificate details:
=====> Common Name(s):
=====>    staging.example.co.jp
=====>    staging.example.co.jp
=====> Expires At: Jul  3 02:27:00 2016 GMT
=====> Issuer: C=US, O=Lets Encrypt, CN=Lets Encrypt Authority X3
=====> Starts At: Apr  4 02:27:00 2016 GMT
=====> Subject: CN=staging.example.co.jp
=====> SSL certificate is self signed.
adminuser@tk2-213-16013:~$ dokku certs:info production.example.co.jp
-----> Fetching SSL Endpoint info for production.example.co.jp...
-----> Certificate details:
=====> Common Name(s):
=====>    production.example.co.jp
=====>    production.example.co.jp
=====>    example.co.jp
=====> Expires At: Aug  4 00:05:31 2016 GMT
=====> Issuer: C=IL, O=StartCom Ltd., OU=Secure Digital Certificate Signing, CN=StartCom Class 1 Primary Intermediate Server CA
=====> Starts At: Aug  3 18:20:22 2015 GMT
=====> Subject: C=JP; CN=production.example.co.jp; emailAddress=hostmaster@example.co.jp
=====> SSL certificate is self signed.
adminuser@tk2-213-16013:~$ dokku certs:info another.example.co.jp
-----> Fetching SSL Endpoint info for another.example.co.jp...
-----> Certificate details:
=====> Common Name(s):
=====>    another.example.co.jp
=====>    another.example.co.jp
=====>    example.co.jp
=====> Expires At: Apr 23 04:56:14 2016 GMT
=====> Issuer: C=IL, O=StartCom Ltd., OU=Secure Digital Certificate Signing, CN=StartCom Class 1 Primary Intermediate Server CA
=====> Starts At: Apr 22 23:55:13 2015 GMT
=====> Subject: C=JP; CN=another.example.co.jp; emailAddress=hostmaster@example.co.jp
=====> SSL certificate is self signed.
adminuser@tk2-213-16013:~$
```
