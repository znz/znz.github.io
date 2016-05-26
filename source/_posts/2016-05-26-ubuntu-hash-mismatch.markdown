---
layout: post
title: "ubuntuで「ハッシュサムが適合しません」で backports のパッケージが入ってしまって大変だった"
date: 2016-05-26 22:23:35 +0900
comments: true
categories: ubuntu linux
---
`jp.archive.ubuntu.com` のミラーが不完全だったのか、「ハッシュサムが適合しません」という警告が出て、そのまま `sudo aptitude full-upgrade -DV` をしたら backports のパッケージが入ってしまって大変な思いをしたという話です。

<!--more-->

## 環境

- Ubuntu 14.04.4 LTS

## 警告

`sudo aptitude update` で以下のような警告とエラーが出ていました。

```
W: http://jp.archive.ubuntu.com/ubuntu/dists/trusty-updates/main/source/Sources を取得できませんでした: ハッシュサムが適合しません
W: http://jp.archive.ubuntu.com/ubuntu/dists/trusty-updates/universe/source/Sources を取得できませんでした: ハッシュサムが適合しません
W: http://jp.archive.ubuntu.com/ubuntu/dists/trusty-updates/main/binary-amd64/Packages を取得できませんでした: ハッシュ サムが適合しません
W: http://jp.archive.ubuntu.com/ubuntu/dists/trusty-updates/universe/binary-amd64/Packages を取得できませんでした: ハッ シュサムが適合しません
W: http://jp.archive.ubuntu.com/ubuntu/dists/trusty-updates/main/binary-i386/Packages を取得できませんでした: ハッシュサムが適合しません
W: http://jp.archive.ubuntu.com/ubuntu/dists/trusty-updates/universe/binary-i386/Packages を取得できませんでした: ハッシュサムが適合しません
E: 一部のインデックスファイルのダウンロードに失敗しました。無視されたか古いものを代わりに利用しています。
E: パッケージキャッシュを再構築できませんでした
```

## 不用意なアップグレード

あまり気にせず、セキュリティアップデートがあれば更新しておこうと思って、 `sudo aptitude full-upgrade -DV` を実行してみたところ、以下のパッケージが更新対象に上がりました。

証明書の設定で `SSLCertificateChainFile` が必要なくなる 2.4.8 をまたぐので、ちょっと危険な気はしましたが、個人のサーバーなので多少の停止時間は構わないだろうと思ってアップグレードしてしまいました。

```
以下のパッケージが更新されます:
  apache2 [2.4.7-1ubuntu4.9 -> 2.4.10-1ubuntu1.1~ubuntu14.04.1]
  apache2-bin [2.4.7-1ubuntu4.9 -> 2.4.10-1ubuntu1.1~ubuntu14.04.1]
  apache2-data [2.4.7-1ubuntu4.9 -> 2.4.10-1ubuntu1.1~ubuntu14.04.1]
  apache2-dev [2.4.7-1ubuntu4.9 -> 2.4.10-1ubuntu1.1~ubuntu14.04.1]
  apache2-mpm-prefork [2.4.7-1ubuntu4.9 -> 2.4.10-1ubuntu1.1~ubuntu14.04.1]
  apache2-utils [2.4.7-1ubuntu4.9 -> 2.4.10-1ubuntu1.1~ubuntu14.04.1]
  libcgmanager0 [0.24-0ubuntu7.5 -> 0.39-2ubuntu2~ubuntu14.04.1]
```

## backports のパッケージだった

ダウンロード中の URL を見て backports のパッケージが入ってしまっている、と気づいたのですが、手遅れで止める間もなく上がってしまいました。

```
% apt-cache policy apache2
apache2:
  インストールされているバージョン: 2.4.7-1ubuntu4.9
  候補:               2.4.10-1ubuntu1.1~ubuntu14.04.1
  バージョンテーブル:
     2.4.10-1ubuntu1.1~ubuntu14.04.1 0
        100 http://jp.archive.ubuntu.com/ubuntu/ trusty-backports/main amd64 Packages
 *** 2.4.7-1ubuntu4.9 0
        100 /var/lib/dpkg/status
     2.4.7-1ubuntu4.5 0
        500 http://security.ubuntu.com/ubuntu/ trusty-security/main amd64 Packages
     2.4.7-1ubuntu4 0
        500 http://jp.archive.ubuntu.com/ubuntu/ trusty/main amd64 Packages
```

## ダウングレード失敗

`/etc/apt/preferences` を設定してダウングレードすることにしました。

まず以下を試してみたところ、 `base-files` などを含む大量のパッケージがダウングレード対象になってしまったので中断しました。

```
Package: *
Pin: release a=trusty
Pin-Priority: 1001
```

`apt-cache policy` で確認してみると、現在のバージョンのインストール候補がなくなっているのが原因の一つでした。

```
% apt-cache policy base-files
base-files:
  インストールされているバージョン: 7.2ubuntu5.4
  候補:               7.2ubuntu5
  バージョンテーブル:
 *** 7.2ubuntu5.4 0
        100 /var/lib/dpkg/status
     7.2ubuntu5 0
       1001 http://ubuntutym.u-toyama.ac.jp/ubuntu/ trusty/main amd64 Packages
```

そこで、 `/etc/apt/sources.list` の末尾に `archive.ubuntu.com` を一時的に追加することにしました。

```
deb http://archive.ubuntu.com/ubuntu/ trusty main restricted universe multiverse
#deb-src http://archive.ubuntu.com/ubuntu/ trusty main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu/ trusty-updates main restricted universe multiverse
deb-src http://archive.ubuntu.com/ubuntu/ trusty-updates main restricted universe multiverse
```

すると `trusty-updates` も出てきたのですが、 priority が 500 になって期待した値になっていませんでした。

```
% apt-cache policy base-files
base-files:
  インストールされているバージョン: 7.2ubuntu5.4
  候補:               7.2ubuntu5
  バージョンテーブル:
 *** 7.2ubuntu5.4 0
        500 http://archive.ubuntu.com/ubuntu/ trusty-updates/main amd64 Packages
        100 /var/lib/dpkg/status
     7.2ubuntu5 0
       1001 http://jp.archive.ubuntu.com/ubuntu/ trusty/main amd64 Packages
       1001 http://archive.ubuntu.com/ubuntu/ trusty/main amd64 Packages
```

次に `n=trusty` を試してみたところ、 `backports` も priority が 1001 になってしまってダウングレードできませんでした。

```
Package: *
Pin: release n=trusty
Pin-Priority: 1001
```

```
% apt-cache policy base-files
base-files:
  インストールされているバージョン: 7.2ubuntu5.4
  候補:               7.2ubuntu5.4
  バージョンテーブル:
 *** 7.2ubuntu5.4 0
       1001 http://archive.ubuntu.com/ubuntu/ trusty-updates/main amd64 Packages
        100 /var/lib/dpkg/status
     7.2ubuntu5 0
       1001 http://jp.archive.ubuntu.com/ubuntu/ trusty/main amd64 Packages
       1001 http://archive.ubuntu.com/ubuntu/ trusty/main amd64 Packages
% apt-cache policy apache2
apache2:
  インストールされているバージョン: 2.4.10-1ubuntu1.1~ubuntu14.04.1
  候補:               2.4.10-1ubuntu1.1~ubuntu14.04.1
  バージョンテーブル:
 *** 2.4.10-1ubuntu1.1~ubuntu14.04.1 0
       1001 http://jp.archive.ubuntu.com/ubuntu/ trusty-backports/main amd64 Packages
        100 /var/lib/dpkg/status
     2.4.7-1ubuntu4.9 0
       1001 http://archive.ubuntu.com/ubuntu/ trusty-updates/main amd64 Packages
     2.4.7-1ubuntu4.5 0
       1001 http://security.ubuntu.com/ubuntu/ trusty-security/main amd64 Packages
     2.4.7-1ubuntu4 0
       1001 http://jp.archive.ubuntu.com/ubuntu/ trusty/main amd64 Packages
       1001 http://archive.ubuntu.com/ubuntu/ trusty/main amd64 Packages
```

## ダウングレード成功

最終的に `a=trusty-updates` と `a=trusty-security` も追加することでダウングレードできました。

```
Package: *
Pin: release a=trusty
Pin-Priority: 1001

Package: *
Pin: release a=trusty-updates
Pin-Priority: 1001

Package: *
Pin: release a=trusty-security
Pin-Priority: 1001
```

```
% apt-cache policy apache2
apache2:
  インストールされているバージョン: 2.4.10-1ubuntu1.1~ubuntu14.04.1
  候補:               2.4.7-1ubuntu4.9
  バージョンテーブル:
 *** 2.4.10-1ubuntu1.1~ubuntu14.04.1 0
        100 http://jp.archive.ubuntu.com/ubuntu/ trusty-backports/main amd64 Packages
        100 /var/lib/dpkg/status
     2.4.7-1ubuntu4.9 0
       1001 http://archive.ubuntu.com/ubuntu/ trusty-updates/main amd64 Packages
     2.4.7-1ubuntu4.5 0
       1001 http://security.ubuntu.com/ubuntu/ trusty-security/main amd64 Packages
     2.4.7-1ubuntu4 0
       1001 http://jp.archive.ubuntu.com/ubuntu/ trusty/main amd64 Packages
       1001 http://archive.ubuntu.com/ubuntu/ trusty/main amd64 Packages
% apt-cache policy base-files
base-files:
  インストールされているバージョン: 7.2ubuntu5.4
  候補:               7.2ubuntu5.4
  バージョンテーブル:
 *** 7.2ubuntu5.4 0
       1001 http://archive.ubuntu.com/ubuntu/ trusty-updates/main amd64 Packages
        100 /var/lib/dpkg/status
     7.2ubuntu5 0
       1001 http://jp.archive.ubuntu.com/ubuntu/ trusty/main amd64 Packages
       1001 http://archive.ubuntu.com/ubuntu/ trusty/main amd64 Packages
```

## backports のコメントアウト

backports のパッケージは使っていなかったので、この後 backports の apt-line はコメントアウトしました。

## まとめ

`apt` の `update` に失敗した時はアップグレード対象のパッケージに注意しましょう。

ダウングレードする時は `apt_preferences(5)` の `Pin` 設定をうまく工夫しましょう。

backports のパッケージが不要なら backports の apt-line を追加するのは避けましょう。
