---
layout: post
title: "Vagrant Cloudでboxを公開してみた"
date: 2014-07-21 15:52:35 +0900
comments: true
categories: vagrant vagrantcloud debian
---
[Vagrant CloudからWheezyを入れてみた](http://blog.n-z.jp/blog/2014-07-18-wheezy-from-vagrant-cloud.html "Vagrant CloudからWheezyを入れてみた")で公開されているものを使ってみたので、
今回は
[Vagrant Cloud](https://vagrantcloud.com/)
で日本語で日本向けの Box の公開も試してみました。

<!--more-->

## 手順概要

1. https://vagrantcloud.com/ にログイン
2. [Create Box](https://vagrantcloud.com/boxes/new) で作成
3. Create new version でバージョンを作成
4. Create new provider でバージョンに対応する provider を作成
5. 無料アカウントだと Upload は使えないようなので URL を指定
6. バージョンの編集で Release すると公開

## 登録される情報

ユーザーアカウントに複数の Box が対応していて、
Box に複数のバージョンが対応していて、
バージョンに複数の provider (VirtualBox とか VMware とか) が対応している、
という構造になっているようです。

バージョンは Release するまでは公開されないようです。

古いバージョンは Revoke で破棄できるようなので、
box を置く URL を使い回すなら Revoke してから
ファイルを置き換えて新しいバージョンを登録するのが
良さそうに思いました。

## 作成した box の packer テンプレート

[packer-templates](https://github.com/znz/packer-templates)
で公開しています。

使い方は

    git clone https://github.com/znz/packer-templates
	cd debian-7.6.0-amd64-ja_jp
    packer build debian-7.6.0-amd64-ja_jp.json

で `debian-7.6.0-amd64-ja_jp_virtualbox.box` が作成できます。
試した環境では1時間ぐらいかかりました。

## 使用方法

`vagrant init znzj/debian-7.6.0-amd64-ja_jp`
のように `vagrant init` の引数に `ユーザー名/BOX名` を指定して
`Vagrantfile` を作成すると
`config.vm.box = "znzj/debian-7.6.0-amd64-ja_jp"`
と指定されていて `vagrant up` で自動ダウンロードされて使えます。

## 登録した URL の扱い

box は URL で登録したので、
`https://vagrantcloud.com/znzj/debian-7.6.0-amd64-ja_jp/version/1/provider/virtualbox.box`
のように `vagrantcloud.com` の URL に見えるところからダウンロードしようとした時、
リダイレクトされて登録した URL からのダウンロードになるようです。

`vagrantcloud` 側でキャッシュなどをしてくれるわけではないようなので、
置き場所には注意する必要がありそうです。

今回は需要も多くなさそうで、
日本向けということで
さくらのVPS
に置いてみました。
