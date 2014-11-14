---
layout: post
title: "第5回 コンテナ型仮想化の情報交換会＠大阪に参加しました"
date: 2014-11-14 19:00:32 +0900
comments: true
categories: event lxcjp docker boot2docker
---
[第5回 コンテナ型仮想化の情報交換会＠大阪](http://ct-study.connpass.com/event/9068/ "第5回 コンテナ型仮想化の情報交換会＠大阪")
に参加しました。

<!--more-->

以下メモです。

## いまさら聞けない Docker

- 前半はさくらインターネットの話でした。
- 仮想環境を動かす基盤として専用サーバがまた増えているらしいというのが興味深いと思いました。
- 後半は docker の基本についての話でした。
- http://www.slideshare.net/kunihirotanaka1/immutable-infrastructuredockercontainerstudyosaka20141114

## Docker Registry入門

- https://github.com/docker/docker-registry の話
- 起動は簡単
- pip でインストールも可能 (公式の [Dockerfile](https://github.com/docker/docker-registry/blob/master/Dockerfile) 参照)
- 設定は [config_sample.yml](https://github.com/docker/docker-registry/blob/master/config/config_sample.yml) などを参照
- docker-registry-ui, docker-registry-web で検索すると非公式の Web UI が出てくる
- ミラーリングに使える
  - docker コマンドで `--registry-mirror` オプション
  - private registry では `MIRROR_SOURCE` を指定

## Linuxコンテナの基本と最新情報

- namespace と cgroup
- sane_behavior オプション
- CRIU のデモ

## LT

LT では boot2docker について話しました。
他の発表者も含めて、特に 5 分という制限はなく、緩い感じで発表していました。

<iframe src="http://slide.rabbit-shocker.org/authors/znz/boot2docker-upgrade/viewer.html"
        width="640" height="524"
        frameborder="0"
        marginwidth="0"
        marginheight="0"
        scrolling="no"
        style="border: 1px solid #ccc; border-width: 1px 1px 0; margin-bottom: 5px"
        allowfullscreen> </iframe>
<div style="margin-bottom: 5px">
  <a href="http://slide.rabbit-shocker.org/authors/znz/boot2docker-upgrade/" title="boot2docker upgrade">boot2docker upgrade</a>
</div>
