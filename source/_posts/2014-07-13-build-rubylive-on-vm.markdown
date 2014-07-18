---
layout: post
title: "RubyLiveを仮想環境で作成"
date: 2014-07-13 09:35:02 +0900
comments: true
categories: live debian vagrant docker
---
最近流行りの仮想環境を使ってクリーンな wheezy 環境で RubyLive を作成できるようにしました。

VirtualBox + Vagrant は特殊な制限のない仮想環境なので Live イメージが作成できたのですが、
docker は後述の制限のために作成できませんでした。

<!--more-->

## RubyLive を Vagrant で作成

Vagrant を使ってクリーンな wheezy 環境で RubyLive の ISO を作成できるようにしました。
こちらは問題なく作成できました。

### 動作確認バージョン

- VirtualBox 4.3.12
- Vagrant 1.6.3

### 使い方

- VirtualBox と Vagrant をインストールしておきます。
- `git clone https://github.com/znz/rubylive-builder` で取得します。
- `cd rubylive-builder` で中に入ります。
- `VM_MEMORY=512 vagrant up` のように適当なメモリ容量を指定して起動します。 (指定なしなら 1024)
  - 他の項目も環境変数である程度変更できるようにしています。
  - 初回起動時は box をダウンロードするので非常に時間がかかります。
  - provision で live-build などの必要なパッケージをインストールしています。
- `vagrant ssh` でゲストにログインします。
- `/vagrant/rubylive.sh` を実行すると `/home/vagrant/rubylive` で RubyLive のイメージを作成します。
  - 実行するたびにタイムスタンプの入ったファイル名の ISO ファイルが作成されます。
  - ネットワークの速度やマシンスペックに影響を受けると思いますが、試した環境では約1時間かかりました。
- 作成できた `/home/vagrant/rubylive/*.iso` を `/vagrant` にコピーまたは移動して、ホスト OS 側に取り出します。
- 取り出した ISO ファイルを使用します。

なぜか
    chroot: failed to run command `/usr/bin/env': No such file or directory
で失敗することがありましたが、再度 `/vagrant/rubylive.sh` を実行すれば問題なく作成できました。

### 片付け方

- `vagrant destroy` で VM を破棄します。
- `git clone` した作業ディレクトリを削除します。
- wheezy の box が不要なら `vagrant box remove opscode_debian-7.4_chef-provisionerless` で削除します。
- Vagrant や VirtualBox も不要ならアンインストールします。

## RubyLive を Docker で作成 (失敗)

docker 環境の中では `chroot /rubylive/chroot mount -t proc proc /proc` が `EPERM` で失敗するため、作成できませんでした。

### 動作確認バージョン

- docker 1.1.1

### 試し方

- docker をインストールしておきます。
- `git clone https://github.com/znz/rubylive-builder` で取得します。
- `docker build rubylive-builder` で作成に挑戦します。
  - または `cd rubylive-builder` で中に入って `docker build .` です。
- `docker ps -a` で最近の CREATED の IMAGE を確認します。
  - もしくは `docker images` で確認します。
  - 最後の失敗した後の状態は残っていないようでした。
- `docker run -i -t --rm 4b8bc4523794 /bin/bash` のように中に入ります。
  - 4b8bc4523794 のところは確認した IMAGE の ID にしてください。
- `cd rubylive` で rubylive ディレクトリに入って `rake` で作成に再挑戦します。
- `less /rubylive/chroot/debootstrap/debootstrap.log` でログを確認したり、
  `chroot /rubylive/chroot mount -t proc proc /proc` や
  `mount -t proc proc /rubylive/chroot/proc` を直接実行してみたりして
  原因を確認します。

### 失敗部分のメッセージ

    W: Failure trying to run: chroot /rubylive/chroot mount -t proc proc /proc
    W: See /rubylive/chroot/debootstrap/debootstrap.log for details
    P: Begin unmounting filesystems...
    P: Saving caches...
    /usr/bin/env: apt-get: No such file or directory
    rake aborted!
    Command failed with status (1): [sudo lb build...]

`/rubylive/chroot/debootstrap/debootstrap.log` をみると `mount: permission denied` と出ていました。

### Dockerfile 直接指定 (失敗)

`docker build https://raw.githubusercontent.com/znz/rubylive-builder/master/Dockerfile`
のように直接 URL を指定する方法は
`sources.list` を国内ミラーに差し替える部分が失敗して使えませんでした。
