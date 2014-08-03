---
layout: post
title: "boot2dockerでdockerを試す"
date: 2014-02-23 00:47:22 +0900
comments: true
categories: docker boot2docker
---
boot2docker で最新の docker を試してみました。
他ではあまり書いていないアンインストール方法も書いているので、
不要になった時や何か変になった時の削除方法も参考になると思います。

<!--more-->

## 対象バージョン

- Mac OS X 10.9.1 で試しましたが Linux でも同様に動くはずです
- VirtualBox 4.3.6
- boot2docker 0.6.0
- docker 0.8.1

014-08-03 追記: 追記部分の確認バージョン

- boot2docker 1.1.2
- docker 1.1.2

## VirtualBox

あらかじめ VirtualBox をインストールしておきます。

## boot2docker

http://docs.docker.io/en/latest/installation/mac/ の手順に従って
`boot2docker` を入れてみます。

### インストール

適当なディレクトリに `boot2docker` をダウンロードして実行可能にします。

```
 mkdir -p ~/boot2docker
 cd ~/boot2docker
 curl https://raw.github.com/boot2docker/boot2docker/master/boot2docker > boot2docker
 chmod +x boot2docker
```

この段階でのアンインストールは `boot2docker` を削除するだけです。

(2014-08-03 追記:
Mac OS X なら
`brew install boot2docker`
などでもインストールできます。)

### VM イメージの作成

初回は `./boot2docker init` を実行して iso のダウンロードと VM の作成をします。

(2014-08-03 追記:
ISO のダウンロードだけなら `boot2docker download` で、
既に ISO があっても最新にするなら `boot2docker upgrade` で
ダウンロードできます。)

```
% ./boot2docker init
[2014-02-22 23:43:36] Creating VM boot2docker-vm
Virtual machine 'boot2docker-vm' is created and registered.
UUID: 61e0ec36-fc80-4b70-ac11-899f6526a57e
Settings file: '/Users/kazu/VirtualBox VMs/boot2docker-vm/boot2docker-vm.vbox'
[2014-02-22 23:43:36] Setting VM settings
[2014-02-22 23:43:36] Setting VM networking
[2014-02-22 23:43:36] boot2docker.iso not found.
[2014-02-22 23:43:38] Latest version is v0.6.0, downloading...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   346  100   346    0     0    319      0  0:00:01  0:00:01 --:--:--   319
100 24.0M  100 24.0M    0     0   295k      0  0:01:23  0:01:23 --:--:--  498k
[2014-02-22 23:45:02] Done
[2014-02-22 23:45:02] Setting VM disks
[2014-02-22 23:45:02] Creating 40000 Meg hard drive...
Converting from raw image file="stdin" to file="/Users/kazu/.boot2docker/boot2docker-vm.vmdk"...
Creating dynamic image with size 41943040000 bytes (40000MB)...
[2014-02-22 23:45:02] Done.
[2014-02-22 23:45:02] You can now type boot2docker up and wait for the VM to start.
```

この段階のやり直しは VM が起動していれば `./boot2docker stop` で止めて、
`./boot2docker delete` で VM を削除するだけではダメで、
`~/.boot2docker` も削除する必要があります。

パスは `boot2docker` の中の `BOOT2DOCKER_CFG_DIR` に書かれていて、
以下の設定は `$HOME/.boot2docker/profile` で設定変更できます。

- `VM_NAME` : VirtualBox での VM の名前 (デフォルト: `boot2docker-vm`)
- `DOCKER_PORT` : ホスト側で docker 用に使うポート (デフォルト: `4243`)
- `SSH_HOST_PORT` : ホスト側で ssh サーバー用に使うポート (デフォルト: `2022`)
- `VM_DISK` : vmdk ファイルのパス (デフォルト: `${BOOT2DOCKER_CFG_DIR}/${VM_NAME}.vmdk`)
- `VM_DISK_SIZE` : vmdk の容量 (デフォルト: `40000` メガバイト)
- `VM_MEM` : メモリ (デフォルト: `1024`)
- `BOOT2DOCKER_ISO` : ISO ファイルのパス (デフォルト: `${BOOT2DOCKER_CFG_DIR}/boot2docker.iso`)


VM を作り直すだけなら `./boot2docker delete` して `./boot2docker init` すれば
ISO はダウンロードし直さずに VM だけ作り直せます。

間違えて先に `~/.boot2docker` を削除してしまうと VirtualBox の状態が不整合になるので、
VirtualBox の GUI の方の仮想メディアマネージャーで存在しなくっているファイルを除去するなどの対処をして直します。

### VM の起動

docker デーモンの VM を `./boot2docker up` で起動します。

```
% ./boot2docker up
[2014-02-22 23:50:43] Starting boot2docker-vm...
[2014-02-22 23:51:03] Started.

To connect the docker client to the Docker daemon, please set:
export DOCKER_HOST=tcp://localhost:4243

```

止めるのは `./boot2docker stop` です。

### ssh で中に入る

`vagrant` と似た感じで `./boot2docker ssh` で中に入れます。
ユーザーは `docker` でパスワードは `tcuser` です。
Tiny Core Linux ベースなので、こういうパスワードになっているのだと思います。

(2014-08-03 追記:
最近の boot2docker では、
ssh でのログインや sudo はパスワードなしで出来るようになっているようです。
`su - docker` で確認したところ、パスワードは `tcuser` のままでした。)

```
% ./boot2docker ssh
docker@localhost's password:
                        ##        .
                  ## ## ##       ==
               ## ## ## ##      ===
           /""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
           \______ o          __/
             \    \        __/
              \____\______/
 _                 _   ____     _            _
| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
boot2docker: 0.6.0
docker@boot2docker:~$
```

## docker を使ってみる

ホスト OS 側に docker コマンドをインストールして、そこから使っても良いのですが、
Homebrew でインストールできたバージョンが 0.8.0 で少し古かったので、
主に VM の中の docker コマンドを使いました。

ドキュメントでは `sudo docker` で実行していますが、
TCP 接続だったり、ローカルの unix socket の場合でもグループのアクセス権限で許可されている場合などは
`sudo` なしの `docker` コマンドだけで大丈夫です。

`boot2docker` の VM の場合は `/var/run/docker.sock` が `docker` グループに読み書きが許可されていて、
`docker` ユーザーが `docker` グループに属しているため、 `sudo` が不要になっています。

### バージョンなどの情報確認

`docker version` や `docker info` で情報を確認します。

`boot2docker` の場合は問題ないと思いますが、
docker のサーバーとの接続がうまくいかない場合はここでエラーなどになるので、
環境変数 `DOCKER_HOST` やポートフォワーディングや firewall などの設定を確認します。

まず VM の外の Homebrew でインストールした docker コマンドから確認しました。
ここでバージョンが古かったので、後は ssh で入った VM の中で docker コマンドを使うことにしました。

(2014-08-03 追記:
最近のバージョンでは `boot2docker up` の時に
`export DOCKER_HOST=tcp://192.168.59.103:2375`
のようなメッセージが出てくるように docker の接続に使う IP アドレスとポート番号が変わっています。)

```
% export DOCKER_HOST=tcp://127.0.0.1:4243
% docker version
Client version: 0.8.0
Go version (client): go1.2
Git commit (client): cc3a8c8d8ec57e15b7b7316797132d770408ab1a

Server version: 0.8.1
Git commit (server): a1598d1
Go version (server): go1.2
Last stable version: 0.8.1, please update docker
```

VM の中では Server と Client のバージョンが最新版で一致しています。
ついでに `docker info` も確認しました。

```
docker@boot2docker:~$ docker version
Client version: 0.8.1
Go version (client): go1.2
Git commit (client): a1598d1
Server version: 0.8.1
Git commit (server): a1598d1
Go version (server): go1.2
Last stable version: 0.8.1
docker@boot2docker:~$ docker info
Containers: 0
Images: 0
Driver: aufs
 Root Dir: /mnt/sda1/var/lib/docker/aufs
 Dirs: 0
Debug mode (server): true
Debug mode (client): false
Fds: 10
Goroutines: 13
Execution Driver: lxc-0.8.0
EventsListeners: 0
Kernel Version: 3.13.3-tinycore64
Init Path: /usr/local/bin/docker
docker@boot2docker:~$
```

### docker を試してみる

`docker pull ubuntu` であらかじめダウンロードしてから `docker run` を実行しても良いのですが、
いきなり `docker run -i -t ubuntu /bin/bash` を実行しても、
自動的にダウンロードしてから実行されます。

ダウンロードはサーバー側が遅いのか、時間がかかることがあるようなので、
ゆっくり待った方が良さそうです。

引数に指定した `/bin/bash` がコンテナの中の唯一のプロセスとして起動するので、
自由に試してみて、 `exit` などで `/bin/bash` が終了すればコンテナも終了します。

```
docker@boot2docker:~$ docker run -i -t ubuntu /bin/bash
Unable to find image 'ubuntu' locally
Pulling repository ubuntu
9cd978db300e: Download complete
eb601b8965b8: Download complete
5ac751e8d623: Download complete
9cc9ea5ea540: Download complete
9f676bd305a4: Download complete
511136ea3c5a: Download complete
f323cf34fd77: Download complete
6170bb7b0ad1: Download complete
1c7f181e78b9: Download complete
7a4f87241845: Download complete
321f7f4200f4: Download complete
root@a64cca91db41:/# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 15:32 ?        00:00:00 /bin/bash
root         9     1  0 15:33 ?        00:00:00 ps -ef
root@a64cca91db41:/# exit
exit
docker@boot2docker:~$
```

[Dockerで不要になったコンテナやイメージを削除する](http://blog.n-z.jp/blog/2013-12-24-docker-rm.html)
に書いたようにコンテナが溜まっていくので、削除しておきます。

```
docker@boot2docker:~$ docker ps -a -q
a64cca91db41
docker@boot2docker:~$ docker rm `docker ps -a -q`
a64cca91db41
docker@boot2docker:~$
```

ちょっと試すだけなら `-rm` オプションを付けて自動削除するのが良さそうです。

(2014-08-03 追記:
最近の `docker` では `-rm` ではなく `--rm` を使うように警告
「`Warning: '-rm' is deprecated, it will be replaced by '--rm' soon. See usage.`」
が出てくるので、
将来のバージョンでは長いオプションには `-` 2個が必須になるようです。)

```
docker@boot2docker:~$ docker run -i -t -rm ubuntu /bin/bash
root@f810cbccf394:/# exit
exit
docker@boot2docker:~$ docker ps -a -q
docker@boot2docker:~$
```
