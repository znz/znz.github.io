---
layout: post
title: "dockerのWeb UI 3種類を比較してみた"
date: 2013-12-15 00:00
comments: true
categories: docker
---
[docker](http://www.docker.io/) の Web UI を比較してみました。

対象は以下の3種類の Web UI です。
検索すると `DockerUI` 以外が見つけにくいようなので、
まとめて紹介します。

- [Shipyard](https://github.com/shipyard/shipyard)
- [Dockland](https://github.com/dynport/dockland)
- [DockerUI](https://github.com/crosbymichael/dockerui)

<!--more-->

## 対象バージョン

先月末頃にも試したのですが、
その時にはこんなに `docker` が流行るとは思っていなかったので、
ちゃんとまとめていませんでした。

そこで改めて新しいバージョンを試しつつ比較したいと思います。

- amd64 の Ubuntu 13.04 (raring)
- docker 0.7.1
- Shipyard version 0dc558
- Dockland 5a02db9d20
- DockerUI v0.3 (5094acc024)

## Shipyard

### インストール

[QuickStart](https://github.com/shipyard/shipyard/wiki/QuickStart)
の説明を参考にして `/etc/default/docker` を作成して `DOCKER_OPTS` を設定します。
`-d` は `/etc/init/docker.conf` の方に書いてあるので不要です。

VirtualBox の中で試したので、
IP アドレスは `10.0.2.15` にしています。

```sh /etc/default/docker
DOCKER_OPTS="-H tcp://10.0.2.15:4243 -H unix:///var/run/docker.sock"
```

公開されている latest Shipyard image を使って起動します。

```console
docker pull shipyard/shipyard
docker run -i -t -d -p 80:80 -p 8000:8000 shipyard/shipyard
```

バージョンアップの時も同様に `docker pull` でとってくると
古いイメージがたまっていくので、
`docker images` で一覧を確認して
`REPOSITORY` や `TAG` が `<none>` になっているイメージを
適当に `docker rmi` で削除すると良いと思います。

### 初期設定

`http://127.0.0.1:8000/`
を開いて

- username: `admin`
- password: `shipyard`

でログインします。

左の `Hosts` を開いて `Add` で `docker` のホストを追加します。
例えば以下のような設定になります。

- Name: raring64
- Hostname: 10.0.2.15
- Public hostname: (空欄のまま)
- Port: 4243

### 設定変更など

以前は右上の `admin` から `administration` を選んで、
Django administration から追加する必要があったと思うのですが、
今は shipyard の方で設定できるようになっているようです。
追加や有効・無効の切り替えや削除は出来ても、
編集はまだ出来ないようなので、
`Name` などを変更したい時は
Django administration
の方を開く必要があるようです。

ログインパスワードの変更などのユーザー管理も
Django administration
の方を開く必要があるようです。

### 使用感

左の `Containers` や `Images` を開いてみて見ると何となく使い方がわかると思います。
バージョンアップで機能が増えていきそうなので、細かくは書きませんが、
ログが見えたりするのは便利です。

`Applications` は
[hipache](https://github.com/dotcloud/hipache)
に関連しているようです。

## Dockland

### バージョン

作りはじめで放置されているようで、
6 commits しかなくて、
latest commit も 2013 年 6 月になっています。
そのためバージョンは前回と変わっていません。

### 依存関係

sinatra ベースで作られていて、
graphviz も必要なので、
クリーンな環境に入れるのは大変です。

前回は docker のホスト側に入れたら大変だったので、
今回は `Dockerfile` を使ってインストールしました。

### インストール

README にあった `dockland.dockerfile`
は古い docker のバージョンが対象のようなので、
最近の docker にあわせて
`EXPOSE` をコメントアウトするなどの
変更をして使います。
`APP_REVISION` のあたりは特定のリビジョンに固定して使いたい時のものだと思ったので、コメントアウトしています。

```plain dockland.dockerfile
# /tmp/dockland.dockerfile
FROM ubuntu:12.04

RUN sed 's/main$/main universe/' -i /etc/apt/sources.list && apt-get update && apt-get upgrade -y
RUN apt-get install ruby1.9.1 ruby1.9.1-dev build-essential git-core graphviz libssl-dev -y

RUN git clone https://github.com/dynport/dockland.git /app

# this is to speed up updates
RUN cd /app && gem install bundler --no-ri --no-rdoc && bundle

# change the revision to update your image
#ENV APP_REVISION 51f5445abeeb080568edeca248d68b29a66f1387
#RUN cd /app && git fetch -q origin  && git reset -q --hard $APP_REVISION && git clean -q -d -x -f && bundle

#EXPOSE 80

CMD cd /app && bundle exec ./bin/dockland -h ${DOCKER_HOST-http://10.0.2.15:4243} -p 80
```

公開するポートの指定は
docker 0.6.5 から
`Dockerfile` の
`EXPOSE` では出来なくなっていて、
`docker run` の引数の `-p` で指定する必要があります。
( [Docker 0.7 runs on all Linux distributions – and 6 other major features | Docker Blog](http://blog.docker.io/2013/11/docker-0-7-docker-now-runs-on-any-linux-distribution/) の Feature 4: Advanced port redirects 参照)
`EXPOSE 80` のような指定はまだ使えるように書いてあるのですが、
ちゃんと使えたとしても外側のポートが可変なのは使いにくいので、
起動する時に指定するようにしました。

```console
docker build -t dockland:dockland - < /tmp/dockland.dockerfile
id=$(docker run -d -p 8080:80 dockland:dockland)
curl -I http://127.0.0.1:8080
```

docker 側の API の公開設定は shipyard の方で設定したのと同じです。

### 使用感

`graphviz` が必須ということからわかるかもしれませんが、
`docker images -viz` から生成できるイメージの有向グラフを使っています。
上の例だと `http://127.0.0.1:8080/` を開くと
矢印で繋がったイメージの ID 一覧が出てきて、
楕円の中の ID をクリックするとイメージの詳細情報が表示できます。

詳細ページには `DELETE` というボタンがあったり、
History から祖先のイメージをたどれたりするだけで
機能はほとんどありません。

`docker images -viz` の図を良い感じに見たい、
という用途には良さそうです。

## DockerUI

最後は DockerUI です。

### インストール

```console
docker pull crosbymichael/dockerui
docker run -d -p 9000:9000 crosbymichael/dockerui -e="http://10.0.2.15:4243"
```

### `-api-enable-cors` は必要?

DockerUI は docker 側の引数に `-api-enable-cors` が必要と書いていますが、
ちょっと表示を試した感じでは付けていなくても動いていました。
本当に必要なのでしょうか?

### 使用感

最初の画面で円グラフが表示されているなど、
見た目は良く、
前回試した時に比べて機能も増えているように感じました。

認証がなかったり、 `-api-enable-cors` が必要と書いていたりして
セキュリティ的に気になる点がありました。

VM の中で動かすだけなので、セキュリティ的なことは気にしないとか、
firewall などで別途セキュリティは確保できるという状態なら
良いのではないでしょうか。

### dokku 環境にインストール

もうちょっと調べていたところ、
[Makes dockerui compatible with dokku](https://github.com/crosbymichael/dockerui/pull/14)
という変更が入っているようなので、
`dokku` 環境では `docker pull` の代わりに
`dokku` への `git push` で簡単に最新版を使えるようになるようです。

`dokku` への `git push` は Installing のところで少し時間がかかります。

deploy 出来れば他の `dokku` に deploy したアプリと同じように
`http://dockerui.deploy.127.0.0.1.xip.io`
のような `dokku` で設定した URL で見えるようになります。

参考のため、実際に試したログを載せておきます。
`~/.ssh/config` で `raring64` という名前で `ssh` 接続できるようにしていて、
`dokku` ユーザーに接続する必要があるので、
`dokku@raring64` になっています。

```console
% git clone https://github.com/crosbymichael/dockerui
Cloning into 'dockerui'...
remote: Counting objects: 709, done.
remote: Compressing objects: 100% (385/385), done.
remote: Total 709 (delta 315), reused 705 (delta 314)
Receiving objects: 100% (709/709), 2.50 MiB | 585.00 KiB/s, done.
Resolving deltas: 100% (315/315), done.
Checking connectivity... done.
% cd dockerui
% git remote add raring64 dokku@raring64:dockerui
% git push raring64 master
Counting objects: 705, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (389/389), done.
Writing objects: 100% (705/705), 2.50 MiB | 0 bytes/s, done.
Total 705 (delta 313), reused 698 (delta 306)
-----> Cleaning up ...
-----> Building dockerui ...
       Go app detected
-----> Installing Go 1.1.2... done
       Installing Virtualenv... done
       Installing Mercurial... done
       Installing Bazaar... done
-----> Running: go get -tags heroku ./...
-----> Discovering process types
       Procfile declares types -> web
-----> Releasing dockerui ...
-----> Deploying dockerui ...
=====> Application deployed:
       http://dockerui.deploy.127.0.0.1.xip.io

To dokku@raring64:dockerui
 * [new branch]      master -> master
Killed by signal 1.
% ssh dokku@raring64 config:set dockerui DOCKER_ENDPOINT=http://10.0.2.15:4243
-----> Setting config vars and restarting dockerui
DOCKER_ENDPOINT: http://10.0.2.15:4243
-----> Releasing dockerui ...
-----> Release complete!
-----> Deploying dockerui ...
-----> Deploy complete!
Killed by signal 1.
```

最初は pull request の説明にあった `config:add` で試していて、
`ssh dokku@raring64 logs dockerui` で確認してみても
`http: proxy error: unsupported protocol scheme ""`
とログに出ているだけでうまく情報がとれていないと思ったら、
`config:set` が正しい、というのが原因でした。

## まとめ

以前に試した時は `Shipyard`
が一番高機能で開発も活発に見えて良さそうに感じましたが、
今は `DockerUI` も機能が増えていて、
今後、どちらも主流になる可能性があるように感じました。

`DockerUI` の方は認証がなかったり、
docker の設定に `-api-enable-cors` が必要と書いていたりするので、
セキュリティを気にするのなら、
`Shipyard` の方がお勧めできると思いました。

`Dockland` は `docker images -viz` を使っているというアイデアは
良さそうに感じたので、ぜひ他の Web UI でも採用されると便利そうだと
思いました。
