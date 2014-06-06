---
layout: post
title: "Debian squeeze (6.0) LTS の使い方"
date: 2014-06-06 14:07:35 +0900
comments: true
categories: debian
---
OpenSSL の脆弱性
([CCS Injection](http://ccsinjection.lepidum.co.jp/ja.html),[CVE-2014-0224](https://security-tracker.debian.org/tracker/CVE-2014-0224))
の修正が
[squeeze の LTS に入ったというアナウンス](https://lists.debian.org/debian-lts-announce/2014/06/msg00002.html)
があったのに、
今まで通り使っている squeeze には修正が入らないと思って調べてみたところ、
[apt-line の追加](https://wiki.debian.org/LTS/Using#Add_squeeze-lts_to_your_sources.list)
が必要でした。

<!--more-->

## 追加する apt-line

既存の `/etc/apt/sources.list` に

    deb http://ftp.jp.debian.org/debian squeeze main non-free contrib
    deb-src http://ftp.jp.debian.org/debian squeeze main non-free contrib

    deb http://security.debian.org/ squeeze/updates main contrib non-free
    deb-src http://security.debian.org/ squeeze/updates main contrib non-free

    # squeeze-updates, previously known as 'volatile'
    deb http://ftp.jp.debian.org/debian squeeze-updates main contrib non-free
    deb-src http://ftp.jp.debian.org/debian squeeze-updates main contrib non-free

という apt-line を設定したとすると、

    deb http://ftp.jp.debian.org/debian squeeze-lts main non-free contrib
    deb-src http://ftp.jp.debian.org/debian squeeze-lts main non-free contrib

を **追加** します。

ネタ元の Debian Wiki に書いてあるように適切に使うには squeeze と squeeze security の apt-line も残した上で squeeze lts の apt-line を **追加** する必要があります。

## apt pinning を使っている時の注意点

`/etc/apt/apt.conf` で

    APT::Default-Release "squeeze";

のように設定している場合はコメントアウトするか

    APT::Default-Release "squeeze-lts";

に書き換える必要があります。

## 更新の反映

いつも通り `apt-get update` と `apt-get upgrade` で更新できます。

## サポート対象外のパッケージを調べる

`squeeze-lts` の apt-line を追加すると `debian-security-support` というパッケージがインストールできるようになります。

`debian-security-support` パッケージをインストール中にサポート対象外のパッケージについての情報が表示されます。
インストール後は `check-support-status` で同じ内容が表示できます。

Ubuntu には `ubuntu-support-status` というコマンドがあるので、似たようなものが Debian にもできたということでしょうか。
`debian-security-support` パッケージは今のところ `sid` と `squeeze-lts` にだけあるようです。

## aptitude 検索パターンで LTS に収録されているパッケージ一覧

`aptitude search '~A squeeze-lts'` で `squeeze-lts` に収録されているパッケージ一覧が調べられます。

今のところ、

- `chkrootkit`
- `debian-security-support`
- `gnutls` 関係
- `hello`
- `openssl` 関係

がありました。
