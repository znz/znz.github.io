---
layout: post
title: "dockerのカスタムベースイメージを作成する"
date: 2013-12-13 18:49
comments: true
categories: docker debian ubuntu
---
例などにある ubuntu の base image は
apt-line が archive.ubuntu.com になっていて、
apt-get install などが遅いです。

日本で使うのなら日本のミラーを使った方が良いので、
そういう base image を作ります。

base image はあまりカスタマイズせずに、
派生するイメージにDockerfile などを使って
カスタマイズをした方が望ましいのですが、
ほぼ必須のものを毎回インストールするのは無駄なので、
ついでに日本語 locale を入れるというカスタマイズもしておきます。

<!--more-->

## docker 向けのポイント

最初に docker 向けのポイントをまとめておきます。

- 最小限にするなら `--variant=minbase`
- `--include=iproute` などで `iproute` パッケージを入れておかないとネットワークにつながらない
- `policy-rc.d` とか `initctl` を対処しておかないとパッケージのインストール時に変なことになるかも
- `dpkg` に `force-unsafe-io` を設定すると `apt` を高速化できる
- [mkimage-debootstrap.sh](https://github.com/dotcloud/docker/blob/master/contrib/mkimage-debootstrap.sh) をそのまま使う場合も日本のミラーを指定する方が良い

## base image の作り方

公式ドキュメントの
[Base Image Creation](http://docs.docker.io/en/latest/use/baseimages/)
を参考にして、基本は
[mkimage-debootstrap.sh](https://github.com/dotcloud/docker/blob/master/contrib/mkimage-debootstrap.sh)
の手順を使います。

カスタマイズのため、手順を追いかけるだけで直接は使いません。

## debootstrap の実行

最初は以下のように `/tmp/wheezy64` などの適当な場所に
`debootstrap` で `chroot` 環境を作成します。
proxy 環境なら `sudo http_proxy=$http_proxy debootstrap ...`
のように指定すれば良いようです。


Debian での例:
```
 sudo debootstrap --verbose --variant=minbase --include=iproute --arch=amd64 wheezy /tmp/wheezy64 http://cdn.debian.or.jp/debian
```

Ubuntu での例:
```
 sudo debootstrap --verbose --variant=minbase --arch=amd64 precise /tmp/precise64 http://ftp.jaist.ac.jp/pub/Linux/ubuntu/
```

### variant

`variant` で `minbase` を指定するとインストールされるパッケージが減って、
本当に最小限の環境になります。
具体的には `Essential: yes` のパッケージ
( `aptitude search '~E'` または `aptitude search '?essential'` で一覧)
と `apt` がインストールされます。

`buildd` という `variant` もあって `minbase` に加えて
`build-essential` が追加でインストールされるようなので、
CI 環境用などの base image なら `--variant=buildd` の方が
良いかもしれません。

デフォルトだと `Priority` が `imporant` のパッケージ
( `aptitude search '~pimportant` または `aptitude search '?priority(important)' で一覧)
がインストールされるようです。
インストールされるパッケージの差分は
`aptitude search '~pimportant!~E`
で調べられます。

### include

元の `mkimage-debootstrap.sh` では `iproute,iputils-ping` と指定してますが、
`iputils-ping` は必須ではないのでここでは省略しています。

`iproute` は docker 環境では必須です。
このパッケージに含まれる `ip` コマンドが入っていないとネットワークにつながりません。

`iproute` パッケージは `Priority` が `optional` なので
普通に `debootstrap` を実行しても入らないので、
注意が必要です。

### その他の引数

arch の指定とか suite の指定とか生成先ディレクトリの指定とか、
ミラーの指定とかは見てわかる通りです。

## docker 向けのカスタマイズ

次に生成されたディレクトリの中で
`mkimage-debootstrap.sh`
にデフォルト (`-d` オプションが指定されなかったとき) の処理をしていきます。

### policy-rc.d

ファイルの作成方法は何でも良いのですが、
exit status で 101 を返す `usr/sbin/policy-rc.d` を作成して、
パッケージのインストールやアップデートなどで init スクリプトが
実行されないようにします。

ちなみに
`$'...'` は bash に `\n` を解釈させるための書き方なので、
`'...'` や `"..."` の間違いではありません。

```
 echo $'#!/bin/sh\nexit 101' | sudo tee usr/sbin/policy-rc.d > /dev/null
 sudo chmod +x usr/sbin/policy-rc.d
```

`policy-rc.d` については `invoke-rc.d` の man を参照してください。

### sbin/initctl

initctl を実行してしまう upstart スクリプトがあるらしく、
その対処もします。

policy-rc.d は存在しなかったので、作成するだけでしたが、
`sbin/initctl` はパッケージ管理のファイルとして存在するので
`dpkg-divert` でパッケージの更新などで上書きされないようにしています。

```
 sudo chroot . dpkg-divert --local --rename --add /sbin/initctl
 sudo ln -sf /bin/true sbin/initctl
```

### パッケージのキャッシュの削除

```
 sudo chroot . apt-get clean
```

を実行して不要な deb ファイルなどを削除して、
イメージのサイズを削減しています。

後で独自カスタマイズのところでパッケージをインストールして、
その後でまた実行するので、その場合はここでは実行しなくてもかまいません。

### apt の高速化など

`mkimage-debootstrap.sh` のコメントには
`dpkg` がパッケージの展開後に `sync()` を呼んでいるのが
原因で無駄に遅くなっているので、
強制的に `sync()` を呼ばなくさせると書いています。

```
 echo 'force-unsafe-io' | sudo tee etc/dpkg/dpkg.cfg.d/02apt-speedup > /dev/null
```

それから、 deb ファイルを残さないようにして image ファイルが大きくならないようにしています。

```
 echo 'DPkg::Post-Invoke {"/bin/rm -f /var/cache/apt/archives/*.deb || true";};' | sudo tee etc/apt/apt.conf.d/no-cache > /dev/null
```

### 元に戻す方法

`mkimage-debootstrap.sh` のコメントに
元に戻す方法も書いてありました。
`dpkg-divert` 以外はファイルを消すだけです。

```
 rm /usr/sbin/policy-rc.d
 rm /sbin/initctl; dpkg-divert --rename --remove /sbin/initctl
 rm /etc/dpkg/dpkg.cfg.d/02apt-speedup
 rm /etc/apt/apt.conf.d/no-cache
```

### apt-line の変更

`etc/apt/sources.list` をみると

```
 deb http://cdn.debian.or.jp/debian wheezy main
```

だけになっているので、 `updates` と `security` を追加します。

`mkimage-debootstrap.sh` もデフォルト
(`-d` も `-s` も指定されていないとき)
の動作では追加します。

この例では以下のようにしました。

debian での例:
```
 deb http://cdn.debian.or.jp/debian wheezy main
 deb http://cdn.debian.or.jp/debian wheezy-updates main
 deb http://security.debian.org/ wheezy/updates main
```

ubuntu での例:
```
 deb http://ftp.jaist.ac.jp/pub/Linux/ubuntu precise main universe
 deb http://ftp.jaist.ac.jp/pub/Linux/ubuntu precise-updates main universe
 deb http://ftp.jaist.ac.jp/pub/Linux/ubuntu precise-security main universe
```

main 以外を追加したい場合はここで追加しておくと良さそうです。
`mkimage-debootstrap.sh` でも ubuntu の場合は `universe` が追加されていました。

## 独自カスタマイズ

ここから独自カスタマイズになります。

### アップデート実行

base image にセキュリティアップデートも入れておきたいなら、
更新しておきます。

```
 sudo chroot . apt-get update
 sudo chroot . apt-get dist-upgrade
```

### 日本語 locale 追加

debian の場合は
別環境で `debconf-get-selections` で調べておいた設定を使って、
`debconf-set-selections` で設定を入れておいて
`DEBIAN_FRONTEND=noninteractive` でインストールします。

```
 echo locales locales/locales_to_be_generated multiselect ja_JP.EUC-JP EUC-JP, ja_JP.UTF-8 UTF-8 | sudo chroot . debconf-set-selections
 echo locales locales/default_environment_locale select ja_JP.UTF-8 | sudo chroot . debconf-set-selections
 sudo chroot . env DEBIAN_FRONTEND=noninteractive apt-get install locales
```

ubuntu の場合は
`language-pack-ja` パッケージを入れても良いのですが、
不要なパッケージを入れるのが嫌なら `locale-gen` コマンドで
生成しても良いです。

```
 sudo chroot . locale-gen ja_JP.UTF-8
 sudo chroot . locale-gen ja_JP.EUC-JP
```

### パッケージのキャッシュの削除

カスタマイズが終わったら clean を実行しておきます。
`etc/apt/apt.conf.d/no-cache` を作成していれば不要かもしれません。

```
 sudo chroot . apt-get clean
```

## イメージ作成と取り込み

### tarball 作成

`mkimage-debootstrap.sh` は `-t` オプションが指定されたときに
docker のイメージではなく tarball を作成します。
直接取り込むならこの手順は不要です。

作成方法としては
最初に `touch` で一般ユーザー権限のファイルになるようにしておいて、
中身は `root` 権限で入れるようにしています。

```
 touch /tmp/wheezy64.tar.xz
 sudo tar --numeric-owner -caf /tmp/wheezy64.tar.xz .
```

### イメージ取り込み

`sudo docker` は root 権限が不要な設定にいていれば `docker` だけでかまいません。

`mkimage-debootstrap.sh` は安定版や LTS に `latest` タグを設定したり、
`etc/debian_version` や `etc/lsb-release` をみて
タグを設定しているので、必要に応じて設定しておきます。

イメージ名としては「ユーザー名/レポジトリ名」という形式が推奨されていますが、
ここでは例として「ユーザー名」の部分は「local」にしておきます。
そして「レポジトリ名」としては日本語 locale を入れたということで
`-ja` を付けました。
日本のミラーを使っているということで `-ja-jp` にしても良かったのですが、
長かったので、 `-ja` だけにしました。

Debian での例:
```
 sudo tar --numeric-owner -c . | sudo docker import - local/debian-ja:wheezy
 sudo docker tag local/debian-ja:wheezy local/debian-ja:latest
 sudo docker tag local/debian-ja:wheezy local/debian-ja:7.2
```

Ubuntu での例:
```
 sudo tar --numeric-owner -c . | sudo docker import - local/ubuntu-ja:precise
 sudo docker tag local/ubuntu-ja:precise local/ubuntu-ja:latest
 sudo docker tag local/ubuntu-ja:precise local/ubuntu-ja:12.04
```

## まとめ

[mkimage-debootstrap.sh](https://github.com/dotcloud/docker/blob/master/contrib/mkimage-debootstrap.sh)
で何をやっているのか、
同じことを手動でやるのはどうするのかということを説明しました。

最初のポイントにも書きましたが
`mkimage-debootstrap.sh`
を直接使うのも良いですが、
最低限ミラーを指定するのがおすすめです。

目的によっては `variant` を変更したり、
`include` でインストールするパッケージを増やしておくだけでも
便利になると思います。
