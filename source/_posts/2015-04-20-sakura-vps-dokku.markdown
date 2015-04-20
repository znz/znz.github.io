---
layout: post
title: "さくらのVPSにdokkuをdebで入れてみた"
date: 2015-04-20 21:54:51 +0900
comments: true
categories: dokku docker ubuntu linux
---
さくらの VPS を新しく借りて初期設定をして、
dokku 0.3.17 を Debian パッケージで入れてみたので、そのメモです。

<!--more-->

## Ubuntu 14.04 インストール

[カスタムOSインストールガイド - Ubuntu 12.04/14.04](https://help.sakura.ad.jp/app/answers/detail/a_id/2403 "カスタムOSインストールガイド - Ubuntu 12.04/14.04")
を参考にしてインストールしました。

## etckeeper インストール

etckeeper だけをインストールすると bzr が一緒に入って使われてしまうので、
git と一緒にインストールすることで bzr が入らないようにします。

    sudo aptitude install git etckeeper

次に etckeeper.conf を編集して `VCS="bzr"` の代わりに `VCS="git"` を有効にします。
ついでにコミットするときに差分をみたいので `GIT_COMMIT_OPTIONS="-v"` を設定しました。

    EDITOR=vi sudoedit /etc/etckeeper/etckeeper.conf

パッケージをインストールしたときに自動で初期コミットされていないので、
手動で初期コミットをしておきます。

    sudo etckeeper init
    sudo etckeeper commit "Initial commit"

git なので `git gc` もしておきます。
`etckeeper vcs コマンド` で `/etc` ディレクトリで `git コマンド` を実行するのと同じ意味になります。

    sudo etckeeper vcs gc

## ufw を有効にする

firewall 設定のために ufw を有効にします。

    sudo ufw enable
    sudo etckeeper commit "Enable ufw"

## ssh を許可

初期設定のために 22 番ポートを許可します。

    sudo ufw allow 22/tcp

リモートから ssh で入って `~/.ssh/authorized_keys` の設置などをします。

次にポート番号の変更や
`PasswordAuthentication no` への設定変更、
`PermitRootLogin` が `yes` 以外になっていることの確認、
`ChallengeResponseAuthentication no` の確認、
`AllowUsers` の追加をしました。

    sudo ufw delete allow 22/tcp
    EDITOR=vi sudoedit /etc/ssh/sshd_config
    sudo service ssh restart
    sudo etckeeper commit "Setup sshd"

## `~/.ssh/config` 設定

クライアント側の `~/.ssh/config` に以下のような設定をして、
ポート番号などの指定を省略できるようにします。

ついでに後で使う dokku 用の設定も追加しました。

    Host サーバーのホスト名
    	Hostname サーバーのIPアドレス
    	User 初期ユーザー名
    	Port 変更したポート番号
    	IdentityFile ~/.ssh/id_rsa
    	IdentitiesOnly yes
    Host dokku-vps
    	Hostname サーバーのIPアドレス
    	User dokku
    	Port 変更したポート番号
    	IdentityFile ~/.ssh/id_rsa
    	IdentitiesOnly yes
    	RequestTTY yes

## nano を purge

vi に慣れていて、
nano は使いにくいと感じているので、
purge しました。

    sudo aptitude purge nano

## IPv6 設定

interfaces ファイルに以下の inet6 の設定を追加します。

    iface eth0 inet6 static
    	address コントロールパネルで確認できるIPv6アドレス
    	netmask 64
    	gateway fe80::1
    	accept_ra 0
    	autoconf 0
    	privext 0
    	dns-nameservers コントロールパネルで確認できるDNS

編集して設定を反映して疎通確認をしました。

    sudoedit /etc/network/interfaces
    sudo ifdown eth0 && sudo ifup eth0
	ping6 -c 3 www.kame.net

`ifup` のときに `Waiting for DAD... Done` と出ましたが、
IPv6 の Duplicate Address Detection が動いているだけのようなので
問題はなさそうでした。

## dokku の Debian パッケージインストール

dokku 0.3.18 からは Debian パッケージでのインストールがデフォルトになると
[Debian Package Installation Notes](http://progrium.viewdocs.io/dokku/getting-started/install/debian "Debian Package Installation Notes")
に書いてあったので、この手順を参考にしてインストールしました。

    wget https://get.docker.io/gpg
    sudo apt-key add gpg
	rm gpg
	wget https://packagecloud.io/gpg.key
	sudo apt-key add gpg.key
	rm gpg.key
    echo "deb http://get.docker.io/ubuntu docker main" | sudo tee /etc/apt/sources.list.d/docker.list
    echo "deb https://packagecloud.io/dokku/dokku/ubuntu/ trusty main" | sudo tee /etc/apt/sources.list.d/dokku.list
    sudo apt-get update
    sudo apt-get install dokku

なぜか
`Importing buildstep into docker (around 5 minutes)`
で 5 分どころではなく 1 時間ぐらいかかったので、
他のことをしながらのんびり待つ必要がありました。

## 初期設定用ポート開放

いきなり開放してしまうと Dokku Setup を勝手に実行されてしまう可能性があるので、
まず `SSH_CLIENT` 環境変数でサーバーに接続している自分のグローバル IP アドレスを確認して、
その IP アドレスのみから HTTP を許可しました。

    env | grep SSH
    sudo ufw allow proto tcp from 接続元IPアドレス to any port 80

そして `http://サーバーのホスト名/` を開いて Dokku Setup を表示しました。

空欄になっていた `Public Key` には自分の `~/.ssh/id_rsa.pub` を貼付けました。
`Hostname` には IPv6 アドレスが表示されていたので、
`xip.io` (IP アドレスのサブドメインで IP アドレスを返してくれるサービス) を使って
`サーバーのIPv4アドレス.xip.io` (例えば `192.0.2.1.xip.io` のような感じ) を設定しました。
`Use virtualhost naming for apps` にチェックを入れて
`Finish Setup` を押しました。
ブラウザーは `http://progrium.viewdocs.io/dokku/application-deployment` にリダイレクトされました。

サーバー側では
`/home/dokku` 以下に設定が保存される他に、
`/etc/init/dokku-installer.conf` と `/etc/nginx/conf.d/dokku-installer.conf` が削除されるので、
`etckeeper commit` しました。

    sudo etckeeper commit "Finish Dokku Setup"

## HTTP ポート開放

初期設定が終了したので、初期設定用のルールを削除して、
一般に開放するように変更しました。

    sudo ufw delete allow proto tcp from 接続元IPアドレス to any port 80
    sudo ufw allow 80/tcp

## dokku の ssh を許可

`sshd_config` に `AllowUsers dokku` を追加しました。

    sudoedit /etc/ssh/sshd_config
    sudo service ssh restart
	sudo etckeeper commit "Allow ssh to dokku"

## ssh の接続確認

    ssh dokku-vps

で dokku のヘルプが表示されるのを確認しておきます。

## サンプルアプリのデプロイ

最小限のサンプルとして node-js-sample をデプロイします。

    git clone https://github.com/heroku/node-js-sample
    cd node-js-sample
    git remote add dokku-vps dokku-vps:node-js-app
    git push dokku-vps master

`http://node-js-app.サーバーのIPアドレス.xip.io/` を開いて
`Hello World!` と表示されたら成功です。
