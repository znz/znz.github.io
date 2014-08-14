---
layout: post
title: "boot2dockerでdockerのvolumeの保存状況を調べてみた"
date: 2014-08-14 20:36:13 +0900
comments: true
categories: docker boot2docker
---
[Managing data in containers - Docker Documentation](https://docs.docker.com/userguide/dockervolumes/ "Managing data in containers - Docker Documentation")
に
「Volumes persist until no containers use them」
(ボリュームは使っているコンテナがなくなるまで存続する)
と書いてあり、実際のところどうなのかを boot2docker で確認してみました。

確認した範囲ではコンテナを消した後でも残っていました。
確認の仕方が悪いなど気づいた点があればコメントなり twitter で指摘などをよろしくお願いします。

<!--more-->

## 確認バージョン

- Mac OS X 10.9.4
- VirtualBox 4.3.14
- docker 1.1.2
- boot2docker v1.1.2

## クリーンな環境で起動する

まず一度 boot2docker の環境を消してから、作成し直しました。

```console
%  boot2docker delete
%  boot2docker init
2014/08/14 20:35:43 Creating VM boot2docker-vm...
2014/08/14 20:35:43 Apply interim patch to VM boot2docker-vm (https://www.virtualbox.org/ticket/12748)
2014/08/14 20:35:43 Setting NIC #1 to use NAT network...
2014/08/14 20:35:43 Port forwarding [ssh] tcp://127.0.0.1:2022 --> :22
2014/08/14 20:35:43 Port forwarding [docker] tcp://127.0.0.1:2375 --> :2375
2014/08/14 20:35:43 Setting NIC #2 to use host-only network "vboxnet0"...
2014/08/14 20:35:43 Setting VM storage...
2014/08/14 20:35:50 Done. Type `boot2docker up` to start the VM.
% export DOCKER_HOST=tcp://192.168.59.103:2375
% boot2docker up
2014/08/14 20:38:02 Waiting for VM to be started...
...........
2014/08/14 20:38:35 Started.
2014/08/14 20:38:35 Your DOCKER_HOST env variable is already set correctly.
```

## volume 作成

[Folder sharing](https://github.com/boot2docker/boot2docker#folder-sharing "Folder sharing")
の方法で volume を使ったコンテナを作成しました。

```console
% docker run -v /data --name my-data busybox true
Unable to find image 'busybox' locally
Pulling repository busybox
a9eb17255234: Download complete
511136ea3c5a: Download complete
42eed7f1bf2a: Download complete
120e218dd395: Download complete
```

## 実体確認

`/` が入ると `docker inspect -f '{{ '{{' }} index .Volumes./data }}' my-data` のようには確認できないので、
https://github.com/docker/docker/issues/6966 を参考にしてボリュームの実体ディレクトリを確認しました。

一覧で見るだけなら `docker inspect -f '{{ '{{' }} .Volumes }}' my-data` で確認できます。

```console
% docker inspect -f '{{ '{{' }} index .Volumes "/data" }}' my-data
/mnt/sda1/var/lib/docker/vfs/dir/7625e8091bafdaa0cf1342bc33f29b2708e59b1388505fdd860960b09adf1846
% docker inspect -f '{{ '{{' }} .Volumes }}' my-data
map[/data:/mnt/sda1/var/lib/docker/vfs/dir/7625e8091bafdaa0cf1342bc33f29b2708e59b1388505fdd860960b09adf1846]
```

## 適当なファイル作成

目的のボリュームを確認しやすくするために適当なファイルを作成しました。

```console
% docker run -it --rm --volumes-from my-data ubuntu
Unable to find image 'ubuntu' locally
Pulling repository ubuntu
c4ff7513909d: Download complete
511136ea3c5a: Download complete
1c9383292a8f: Download complete
9942dd43ff21: Download complete
d92c3c92fa73: Download complete
0ea0d582fd90: Download complete
cc58e55aa5a5: Download complete
root@bf9a312400fa:/# echo foo > /data/foo.txt
root@bf9a312400fa:/# exit
exit
```

## コンテナ削除

連携しているコンテナを削除します。

```console
% docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
ddd48c305f62        busybox:latest      true                6 minutes ago       Exited (0) 6 minutes ago                       my-data
%  docker ps -a -q | xargs docker rm
ddd48c305f62
```

## 実体確認

`boot2docker ssh` で入って確認したところ、残っていました。

```console
% boot2docker ssh
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
boot2docker: 1.1.2
             master : 740106c - Thu Jul 24 03:24:10 UTC 2014
docker@boot2docker:~$ ls -al /mnt/sda1/var/lib/docker/vfs/dir/7625e8091bafdaa0cf1342bc33f29b2708e59b
1388505fdd860960b09adf1846
ls: /mnt/sda1/var/lib/docker/vfs/dir/7625e8091bafdaa0cf1342bc33f29b2708e59b1388505fdd860960b09adf1846: Permission denied
docker@boot2docker:~$ sudo ls -al /mnt/sda1/var/lib/docker/vfs/dir/7625e8091bafdaa0cf1342bc33f29b270
8e59b1388505fdd860960b09adf1846
total 12
drwxr-xr-x    2 root     root          4096 Aug 14 11:52 .
drwx------    4 root     root          4096 Aug 14 11:49 ..
-rw-r--r--    1 root     root             4 Aug 14 11:52 foo.txt
docker@boot2docker:~$
```
