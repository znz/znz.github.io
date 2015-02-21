---
layout: post
title: "owncloud-client が conflict した話"
date: 2015-01-29 09:30:00 +0900
comments: true
categories: linux debian owncloud
---
Debian 7 に [ownCloud 公式の Desktop Client](https://owncloud.org/install/#desktop) を入れていたら、
なぜか `owncloud-client` の 1.7.1 から 1.7.1 への更新が発生して `libqtkeychain0` と conflict していたので、
`libqtkeychain0` をダウングレードして解決しました。

<!--more-->

## 現象

このように競合が検出されました。

```
$ sudo aptitude full-upgrade -DV
以下のパッケージが更新されます:
  owncloud-client{b} [1.7.1 -> 1.7.1] (競: libqtkeychain0)
更新: 1 個、新規インストール: 0 個、削除: 0 個、保留: 0 個。
700 k バイトのアーカイブを取得する必要があります。展開後に 0  バイトのディスク領域が新たに消費されます。
以下のパッケージには満たされていない依存関係があります:
 owncloud-client : 競合: libqtkeychain0 (= 0.20140128) [0.20140128 が既にインストール済みです]
以下のアクションでこれらの依存関係の問題は解決されます:

     以下のパッケージを削除する:
1)     libowncloudsync0
2)     libqtkeychain0
3)     owncloud-client
4)     owncloud-client-l10n



この解決方法を受け入れますか? [Y/n/q/?]q
```

## 状況

`apt-cache show` でみてみると `Conflicts: libqtkeychain0 (= 0.20140128)` と明示されていました。

```
$ apt-cache show owncloud-client
Package: owncloud-client
Version: 1.7.1
Architecture: amd64
Maintainer: Markus Rex <msrex@owncloud.com>
Installed-Size: 1521
Depends: libc6 (>= 2.4), libgcc1 (>= 1:4.1.1), libneon27, libowncloudsync0 (= 1.7.1), libqt4-dbus (>= 4:4.5.3), libqt4-network (>= 4:4.5.3), libqt4-sql (>= 4:4.5.3), libqt4-xml (>= 4:4.5.3), libqt4-xmlpatterns (>= 4:4.5.3), libqtcore4 (>= 4:4.8.0), libqtgui4 (>= 4:4.7.0~beta1), libqtkeychain0, libqtwebkit4 (>= 2.2.0), libsqlite3-0 (>= 3.5.9), libstdc++6 (>= 4.4.0), owncloud-client-l10n, libqt4-sql-sqlite
Conflicts: libqtkeychain0 (= 0.20140128)
Filename: ./amd64/owncloud-client_1.7.1_amd64.deb
Size: 699596
MD5sum: 23e2bfa2467b45fd9b70f3b946203b4b
SHA1: 422242ad170ad1aa8d75587fe18a0469cc23ad72
SHA256: 319e7253309835d028ec527ee9b3abea014c2db8b7edc5d3d57270ede9dbb5af
Section: devel
Priority: optional
Multi-Arch: same
Description: The ownCloud client is based on Mirall - github.com/owncloud/mirall
 .
 ownCloud client enables you to connect to your private
 ownCloud Server. With it you can create folders in your home
 directory, and keep the contents of those folders synced with your
 ownCloud server. Simply copy a file into the directory and the
 ownCloud client does the rest.
 .
 ownCloud gives your employees anytime, anywhere access to the files
 they need to get the job done, whether through this desktop application,
 our mobile apps, the web interface, or other WebDAV clients. With it,
 your employees can easily view and share documents and information
 critical to the business, in a secure, flexible and controlled
 architecture. You can easily extend ownCloud with plug-ins from the
 community, or that you build yourself to meet the requirements of
 your infrastructure and business.
 .
 ownCloud - Your Cloud, Your Data, Your Way!  www.owncloud.com
 .
 Authors
 =======
 Duncan Mac-Vicar P. <duncan@kde.org>
 Klaas Freitag <freitag@owncloud.com>
 Daniel Molkentin <danimo@owncloud.com>

Package: owncloud-client
Status: install ok installed
Priority: optional
Section: devel
Installed-Size: 1521
Maintainer: Markus Rex <msrex@owncloud.com>
Architecture: amd64
Multi-Arch: same
Version: 1.7.1
Depends: libc6 (>= 2.4), libgcc1 (>= 1:4.1.1), libneon27, libowncloudsync0 (= 1.7.1), libqt4-dbus (>= 4:4.5.3), libqt4-network (>= 4:4.5.3), libqt4-sql (>= 4:4.5.3), libqt4-xml (>= 4:4.5.3), libqt4-xmlpatterns (>= 4:4.5.3), libqtcore4 (>= 4:4.8.0), libqtgui4 (>= 4:4.7.0~beta1), libqtkeychain0, libqtwebkit4 (>= 2.2.0), libsqlite3-0 (>= 3.5.9), libstdc++6 (>= 4.4.0), owncloud-client-l10n, libqt4-sql-sqlite
Description: The ownCloud client is based on Mirall - github.com/owncloud/mirall
 .
 ownCloud client enables you to connect to your private
 ownCloud Server. With it you can create folders in your home
 directory, and keep the contents of those folders synced with your
 ownCloud server. Simply copy a file into the directory and the
 ownCloud client does the rest.
 .
 ownCloud gives your employees anytime, anywhere access to the files
 they need to get the job done, whether through this desktop application,
 our mobile apps, the web interface, or other WebDAV clients. With it,
 your employees can easily view and share documents and information
 critical to the business, in a secure, flexible and controlled
 architecture. You can easily extend ownCloud with plug-ins from the
 community, or that you build yourself to meet the requirements of
 your infrastructure and business.
 .
 ownCloud - Your Cloud, Your Data, Your Way!  www.owncloud.com
 .
 Authors
 =======
 Duncan Mac-Vicar P. <duncan@kde.org>
 Klaas Freitag <freitag@owncloud.com>
 Daniel Molkentin <danimo@owncloud.com>

Package: owncloud-client
Version: 1.5.0+dfsg-4~bpo70+1
Installed-Size: 1076
Maintainer: ownCloud for Debian maintainers <pkg-owncloud-maintainers@lists.alioth.debian.org>
Architecture: amd64
Depends: libowncloudsync0 (= 1.5.0+dfsg-4~bpo70+1), libqt4-sql-sqlite, owncloud-client-l10n, libc6 (>= 2.2.5), libgcc1 (>= 1:4.1.1), libneon27-gnutls, libocsync0 (>= 0.60.3), libqt4-dbus (>= 4:4.5.3), libqt4-network (>= 4:4.6.1), libqt4-sql (>= 4:4.5.3), libqt4-xml (>= 4:4.5.3), libqt4-xmlpatterns (>= 4:4.5.3), libqtcore4 (>= 4:4.8.0), libqtgui4 (>= 4:4.7.0~beta1), libqtkeychain0 (>= 0.1.0), libqtwebkit4 (>= 2.1.0~2011week13), libstdc++6 (>= 4.1.1)
Description-en: folder synchronization with an ownCloud server - GUI
 The ownCloudSync system lets you always have your latest files wherever
 you are. Just specify one or more folders on the local machine to and a server
 to synchronize to. You can configure more computers to synchronize to the same
 server and any change to the files on one computer will silently and reliably
 flow across to every other.
 .
 owncloud-client provides the graphical client specialising in
 synchronizing with cloud storage provided by ownCloud.
Homepage: http://owncloud.org/sync-clients/
Description-md5: a754a2b9b06d1c7c880afd05aa24e101
Section: net
Priority: optional
Filename: pool/main/o/owncloud-client/owncloud-client_1.5.0+dfsg-4~bpo70+1_amd64.deb
Size: 417650
MD5sum: 42a3f6355f9d5f5af0fb42141bee9ecf
SHA1: 52102eacc1d81438cb88e639c09269484279dde1
SHA256: d38e11879d6439dad97ff9f59449077193dbc89331bae8d2866b335ce8b41856
```

さらに `apt-cache policy` でどこからきたパッケージなのかみてみるとどちらも ownCloud 公式のパッケージだったように見えました。
bpo (backports) のパッケージならバージョンが下がることはないはずなので、そう判断しました。

```
$ apt-cache policy owncloud-client
owncloud-client:
  インストールされているバージョン: 1.7.1
  候補:               1.7.1
  バージョンテーブル:
     1.7.1 0
        500 http://download.opensuse.org/repositories/isv:/ownCloud:/desktop/Debian_7.0/  Packages
 *** 1.7.1 0
        100 /var/lib/dpkg/status
     1.5.0+dfsg-4~bpo70+1 0
        100 http://ftp.jp.debian.org/debian/ wheezy-backports/main amd64 Packages
$ apt-cache policy libqtkeychain0
libqtkeychain0:
  インストールされているバージョン: 0.20140128
  候補:               0.20140128
  バージョンテーブル:
 *** 0.20140128 0
        100 /var/lib/dpkg/status
     0.4 0
        500 http://download.opensuse.org/repositories/isv:/ownCloud:/desktop/Debian_7.0/  Packages
     0.1.0-2~bpo70+1 0
        100 http://ftp.jp.debian.org/debian/ wheezy-backports/main amd64 Packages
```

## 対処

`libqtkeychain0` を ownCloud 公式の 0.4.0 にダウングレードすることで解決しました。

```
$ sudo aptitude full-upgrade -DV
以下のパッケージが更新されます:
  owncloud-client{b} [1.7.1 -> 1.7.1] (競: libqtkeychain0)
更新: 1 個、新規インストール: 0 個、削除: 0 個、保留: 0 個。
700 k バイトのアーカイブを取得する必要があります。展開後に 0  バイトのディスク領域が新たに消費されます。
以下のパッケージには満たされていない依存関係があります:
 owncloud-client : 競合: libqtkeychain0 (= 0.20140128) [0.20140128 が既にインストール済みです]
以下のアクションでこれらの依存関係の問題は解決されます:

     以下のパッケージを削除する:
1)     libowncloudsync0
2)     libqtkeychain0
3)     owncloud-client
4)     owncloud-client-l10n



この解決方法を受け入れますか? [Y/n/q/?]n
以下のアクションでこれらの依存関係の問題は解決されます:

     以下のパッケージを削除する:
1)     owncloud-client
2)     owncloud-client-l10n



この解決方法を受け入れますか? [Y/n/q/?]n
以下のアクションでこれらの依存関係の問題は解決されます:

     以下のパッケージを現在のバージョンに一時固定する:
1)     owncloud-client [1.7.1 (now)]



この解決方法を受け入れますか? [Y/n/q/?]n
以下のアクションでこれらの依存関係の問題は解決されます:

     以下のパッケージをダウングレードする:
1)     libqtkeychain0 [0.20140128 (now) -> 0.4 (<NULL>)]



この解決方法を受け入れますか? [Y/n/q/?]y
以下のパッケージがダウングレードされます:
  libqtkeychain0 [0.20140128 -> 0.4]
以下のパッケージが更新されます:
  owncloud-client [1.7.1 -> 1.7.1]
更新: 1 個、新規インストール: 0 個、ダウングレード: 1 個、削除: 0 個、保留: 0 個。
753 k バイトのアーカイブを取得する必要があります。展開後に 0  バイトのディスク領域が新たに消費されます。
先に進みますか? [Y/n/?]
取得: 1 http://download.opensuse.org/repositories/isv:/ownCloud:/desktop/Debian_7.0/  libqtkeychain0 0.4 [53.4 kB]
取得: 2 http://download.opensuse.org/repositories/isv:/ownCloud:/desktop/Debian_7.0/  owncloud-client 1.7.1 [700 kB]
Fetched 753 kB in 4秒 (156 kB/s)
changelog を読んでいます... 完了
dpkg: 警告: libqtkeychain0 を 0.20140128 から 0.4 にダウングレードしています
(データベースを読み込んでいます ... 現在 124701 個のファイルとディレクトリがインストールされています。)
libqtkeychain0 0.20140128 を (.../libqtkeychain0_0.4_amd64.deb で) 置換するための準備をしています ...
libqtkeychain0 を展開し、置換しています...
owncloud-client:amd64 1.7.1 を (.../owncloud-client_1.7.1_amd64.deb で) 置換するための準備をしています ...
owncloud-client:amd64 を展開し、置換しています...
man-db のトリガを処理しています ...
libqtkeychain0 (0.4) を設定しています ...
owncloud-client:amd64 (1.7.1) を設定しています ...

現在の状態: 更新が 0 個 [-1]。
```

## 対処後の状況

`owncloud-client` パッケージも `libqtkeychain0` パッケージもどちらも ownCloud 公式のバージョンになって解決しました。

```
$ apt-cache policy owncloud-client
owncloud-client:
  インストールされているバージョン: 1.7.1
  候補:               1.7.1
  バージョンテーブル:
 *** 1.7.1 0
        500 http://download.opensuse.org/repositories/isv:/ownCloud:/desktop/Debian_7.0/  Packages
        100 /var/lib/dpkg/status
     1.5.0+dfsg-4~bpo70+1 0
        100 http://ftp.jp.debian.org/debian/ wheezy-backports/main amd64 Packages
$ apt-cache policy libqtkeychain0
libqtkeychain0:
  インストールされているバージョン: 0.4
  候補:               0.4
  バージョンテーブル:
 *** 0.4 0
        500 http://download.opensuse.org/repositories/isv:/ownCloud:/desktop/Debian_7.0/  Packages
        100 /var/lib/dpkg/status
     0.1.0-2~bpo70+1 0
        100 http://ftp.jp.debian.org/debian/ wheezy-backports/main amd64 Packages
```

## 考察

Debian 公式パッケージならこういうときは
[Version](https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-Version) に
epoch をつけて解決しますが、
ownCloud 公式は Debian からみると非公式パッケージになるので、
将来の Debian のアップグレードの邪魔にならないように epoch を使わなかったため、
こういう問題が起きたのでないかと推測しました。
