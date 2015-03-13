---
layout: post
title: "Ubuntu 14.04 LTS に Zabbix 2.4 を PostgreSQL を使う設定で入れた"
date: 2015-03-13 21:00:00 +0900
comments: true
categories: zabbix postgresql ubuntu debian
---
[3 Installation from packages [Zabbix Documentation 2.4]](https://www.zabbix.com/documentation/2.4/manual/installation/install_from_packages "3 Installation from packages [Zabbix Documentation 2.4]")
だと MySQL の例しかなく、
PostgreSQL で入れるとちょっとひっかかったところがあったので、
そのメモです。

Debian や Ubuntu の公式パッケージの zabbix との違いも気づいた範囲で書いておきました。

<!--more-->

## zabbix-release のインストール

`zabbix-release` をインストールして apt-line を設定して
`apt-get update` するところまでは公式マニュアルと同じです。

    wget http://repo.zabbix.com/zabbix/2.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_2.4-1+trusty_all.deb
    sudo dpkg -i zabbix-release_2.4-1+trusty_all.deb
    sudo apt-get update

## zabbix-server のインストール

`zabbix-server-mysql` の代わりに `zabbix-server-pgsql` をインストールします。
途中で入力するパスワードは後で必要になるのでメモするか覚えておきます。
(Debian や Ubuntu の公式パッケージの zabbix と違って `dbconfig-common` を使うようになっています。)

    sudo apt-get install zabbix-server-pgsql

ただし、先に postgresql を入れておかないとデータベースの作成のところでエラーになるようです。

## zabbix-frontend-php のインストール

`/usr/share/doc/zabbix-frontend-php/README.Debian` に書いてあるのですが、
`php5-pgsql` を入れておくと初期設定の時のデータベースの選択肢に PostgreSQL が出てきます。

    sudo apt-get install zabbix-frontend-php php5-pgsql

## PHP のタイムゾーンの設定

    sudoedit /etc/zabbix/apache.conf

で

    # php_value date.timezone Europe/Riga

を

    php_value date.timezone Asia/Tokyo

に変更して

    sudo service apache2 reload

で反映します。
(Debian や Ubuntu の公式パッケージの zabbix と違って `apache2` のみ対応です。
`php5-fpm`+`nginx` には対応していないので `nginx` で使うなら完全に独自設定が必要です。)

## 初期設定

`http://localhost/zabbix/` を開いて初期設定を開始します。
`Next` ボタンで進んでいってデータベースの設定のところは

- Database Type : PostgreSQL
- Database host : localhost のまま
- Database port : 0 のまま
- Database name : zabbix のまま
- User : root から zabbix に変更
- Password : `zabbix-server-pgsql` をインストールしたときに設定したパスワード

と設定します。

そして `Next` で進んでいって `Finish` まで行くと初期設定終了です。
(Debian や Ubuntu の公式パッケージの zabbix と違って
`/etc/zabbix/web/` が `www-data` から書き込み可能になっていて
ダウンロードして自分で設置しなくても設定完了するようになっています。)

## ログイン

- Username : Admin
- Password : zabbix

でログインします。

右上の Profile から Language を Japanese (ja_JP) に変更して Update すると
日本語で使えるようになります。
パスワードもここで変更できます。

ログイン前の画面は guest ユーザーの言語が反映されているので、
「管理」-「ユーザー」からメンバーの「guest」を開いて言語を変更して更新します。

## zabbix-agent のインストール

    sudo apt-get install zabbix-agent

でインストールできます。

## Zabbix server の監視

`zabbix-agent` をインストールした後、
「設定」-「ホスト」でステータスを「無効」から「有効」に切り替えます。

しばらくするとエージェントの状態の「Z」が緑色になって監視できていることがわかります。

## グラフの日本語の文字化け対策

グラフなどの図の中の日本語が文字化けするときは
適当な日本語フォントを入れて
`zabbix-frontend-php` の設定をし直せば直ります。

    sudo apt-get install fonts-vlgothic
    sudo dpkg-reconfigure zabbix-frontend-php
