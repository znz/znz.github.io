---
layout: post
title: "boot2docker で VM のファイルをコンテナやホストと共有する"
date: 2014-08-06 23:17:27 +0900
comments: true
categories: docker boot2docker
---
Mac OS X 上の `boot2docker` でホストとコンテナでファイルを共有する方法を試してみました。
さらに `boot2docker ssh` で入ったときにも見えるような設定でも試してみました。

<!--more-->

## 参考

- [boot2dockerでコンテナからホストのファイルを参照する - Qiita](http://qiita.com/numa08/items/e52bd18611ac159af1ac "boot2dockerでコンテナからホストのファイルを参照する - Qiita")
- [Folder sharing](https://github.com/boot2docker/boot2docker#folder-sharing "Folder sharing")
- [Managing data in containers - Docker Documentation](https://docs.docker.com/userguide/dockervolumes/ "Managing data in containers - Docker Documentation")

## 対象バージョン

- Mac OS X 10.9.4
- VirtualBox 4.3.14
- docker 1.1.2
- boot2docker 1.1.2

## 実行コマンド

- `docker run -v /mnt/sda1/data:/data --name my-data busybox true` で共有ボリューム用コンテナ作成
- `docker run --rm -v /usr/local/bin/docker:/docker -v /var/run/docker.sock:/docker.sock svendowideit/samba my-data` で samba 起動
- `docker run -it --rm --volumes-from my-data ubuntu /bin/bash` で確認

### 共有ボリューム用コンテナ作成

[Dockerで不要になったコンテナやイメージを削除する](http://blog.n-z.jp/blog/2013-12-24-docker-rm.html "Dockerで不要になったコンテナやイメージを削除する")
のように `docker ps -a -q | xargs docker rm` などで停止しているコンテナを削除してしまうと
`my-data` という名前を付けたデータ保存用のコンテナも消えてしまうので、
`boot2docker` では永続化されているパーティションの `/mnt/sda1` に `data` をおくことにしました。

run の時点で `/mnt/sda1/data` は自動作成されるので、
あらかじめ作っておく必要はありません。

間違えてコンテナを削除してしまった場合は
`docker run -v /mnt/sda1/data:/data --name my-data busybox true`
で作成し直せばデータは残ったまま `my-data` コンテナを再作成できます。

このやり方は docker を動かすホストに依存してしまうので、
一般には標準のボリュームコンテナを作成する方法の方がおすすめのようです。

### 共有ボリューム用コンテナ再作成 (標準の方法の場合)

`--volumes-from` で指定した共有は使っているコンテナがなくなってしまっても内容が残っていますが、
名前で指定して取り出す方法がなくなってしまうように見えます。

`my-data` コンテナを削除してしまった場合、
`--volumes-from my-data` は使えなくなるので、
`docker run --volumes-from samba-server --name my-data busybox true`
のように残っているコンテナを `--volumes-from` で指定して再作成すれば、
また `docker run -it --rm --volumes-from my-data ubuntu /bin/bash` のように
`--volumes-from` に `my-data` を指定できるようになります。

### samba 起動

`docker run --rm -v /usr/local/bin/docker:/docker -v /var/run/docker.sock:/docker.sock svendowideit/samba my-data`
で samba を起動します。

`docker.sock` も渡しているので、多重起動しないように既存の `samba-server` は止めてくれるようです。

起動時に以下のようにホスト側からの接続方法の説明が出ます。

    % docker run --rm -v /usr/local/bin/docker:/docker -v /var/run/docker.sock:/docker.sock svendowideit/samba my-data
    stopping and removing existing server
    starting samba server container sharing my-data:/data
    
    # run 'docker logs samba-server' to view the samba logs
    
    ================================================
    
    Your data volume (/data) should now be accessible at \\<docker ip>\ as 'guest' user (no password)
    
    For example, on OSX, using a typical boot2docker vm:
        goto Go|Connect to Server in Finder
        enter 'cifs://192.168.59.103
        hit the 'Connect' button
        select the volumes you want to mount
        choose the 'Guest' radiobox and connect
    
    Or on Linux:
        mount -t cifs //192.168.59.103/data /mnt/data -o username=guest
    
    Or on Windows:
        Enter '\\192.168.59.103\data' into Explorer
        Log in as Guest - no password

### samba に接続

`boot2docker ip` で IP アドレスを確認して、
`192.168.59.103` なら、
`Finder` の `サーバへ接続` (メニューの `移動` の `サーバーへ接続...`) を開いて、
サーバアドレスとして `cifs://192.168.59.103/data` を入力して `接続` します。
`ユーザの種類` は `ゲスト` を選んで `接続` します。
すると `/Volumes/data` で見えるようになります。

Linux なら `mount -t cifs //192.168.59.103/data /mnt/data -o username=guest` のようにマウントするそうです。

Windows ならエクスプローラーで `\\192.168.59.103\data` にパスワードなしのゲスト接続すれば見えるそうです。

### 別コンテナで確認

`docker run -it --rm --volumes-from my-data ubuntu /bin/bash` などで別コンテナを起動すると、
`/data` にマウントされているので、
`ls -l /data` で中身を確認したり、
`/data` の中にファイルを作成して他で見えることを確認しました。

## まとめ

[README に書いてある Folder sharing](https://github.com/boot2docker/boot2docker#folder-sharing)
だと間違えて消してしまうことがあったので、ちょっと工夫した方法を紹介しました。
