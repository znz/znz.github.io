---
layout: post
title: "DOCKER indexのTrusted Buildsで複数バージョンのrubyを試せるimageを作ってみた"
date: 2013-12-19 23:02
comments: true
categories: docker ruby rbenv
---
[DOCKER index](https://index.docker.io/)
には
[Trusted Builds](http://blog.docker.io/2013/11/introducing-trusted-builds/)
という機能があり、
Docker index の方で
image の作成をしてくれます。

image の作成方法は GitHub に Dockerfile を公開して、
それを指定します。

今回はそれを使って複数バージョンの ruby を試せる image を作ってみました。

<!--more-->

## Dockerfile の動作確認

`docker build` への Dockerfile の指定方法は
    docker build DockerfileのあるディレクトリへのパスやURL
という方法と
    docker build - < Dockerfile
のように `-` を指定して標準入力から渡す方法があります。

違いとしては
パスや URL の指定だとファイル名が Dockerfile 固定という制限があり、
`-` の方だと `ADD` が使えない (基準となるディレクトリがわからないため)
という制限があるようです。
(詳しく調べていないだけなので他にもあると思います。)

そこで、
    docker build -t local/rubys .
または
    docker build -t local/rubys - < Dockerfile
のようにして動作確認します。

## DOCKER index のアカウント登録

英小文字か数字で4文字から30文字という制限があったので、
`znzj` というアカウントにしました。

メールアドレスとパスワードを設定して、
メールの確認が済んだらアカウントの作成は完了です。

最近はサービスごとにメールアドレスを分けていて、
アイコンが gravatar のデフォルトになってしまっていたので、
User Settings から Gravatar email も設定しました。

ここはメールアドレスの確認がなかったので、
他人のアイコンでも使えてしまうように見えたのですが、
良いのでしょうか。
ちょっと考えてみましたが、
画像をコピーして使えば同じようなものなので、
気にするほどのことではなさそうに思いました。

## GitHub のアカウントとの連携

hook の登録のため、ということで多少の書き込み権限も要求されるので、
許可したくない場合は Trusted builds は使えません。

## Trusted Builds の追加

ログイン中に
[Trusted Builds](https://index.docker.io/builds/)
のページの `+Add New` から、
GitHub レポジトリを選択します。
ここでは https://github.com/znz/docker-rubys を選択しました。

1. Default Branch は `master` のまま
2. Repo name も `znzj/docker-rubys` のまま (`/` の右の部分は英小文字か数字か `-` か `.` で 3 文字から 30 文字)
3. Docker Tag Name も `latest` のまま
4. Dockerfile Location は `/` から `rubys/` に変更
5. Active にはチェックをいれたまま

という状態で作成しました。

`Dockerfile` の push とどっちが先が良いのかわからなかったので、
`Dockerfile` なしで追加してしまったら、
初回は `Dockerfile` が見つからないという理由で失敗してしまったので、
先に `Dockerfile` を push してから追加するもののようです。

## イメージの使用

しばらくまつと Trusted Builds のページで
Status が Pending から Building に変わって、
最終的に Done になってビルドできて使えるようになるので、
    docker pull znzj/docker-rubys
でダウンロードします。

ダウンロードが完了したら、
    docker run -i -t znzj/docker-rubys
で `/bin/bash -l` を起動します。
(`CMD` で指定されています。)

bash で
    rbenv versions
で入っている ruby のバージョンを確認したり、
    rbenv each ruby -v
や
    rbenv each -v gem list
などのように
[rbenv each](https://github.com/chriseppstein/rbenv-each)
を使って、
それぞれのバージョンの ruby の環境でコマンドを実行できるようにしています。

ホスト側とのファイルのやり取りはどうするのが良いのか
まだ調べていないので、
とりあえず vim でファイルを作成するか、
wget でダウンロードすることを想定しています。

## abuse?

`GitHub: Add Trusted Build`
のところに
`Anyone who abuses the build system, will have their accounts disabled. If you are unsure what might be considered abuse, please ask before you build.`
という注意書きがあって、
ビルドするぐらいなら大丈夫かと思っていたのですが、
他の Trusted Builds をみるとインストールしたり
ファイルを追加したりしているだけのものが
多いので心配になってしまいました。
さらに探してみると
https://index.docker.io/u/sameersbn/gitlab/
で ruby を make しているものもあったので、
ビルドはダメということもなさそうでした。

しかし、頻繁にビルドして負荷をかけるのもよくなさそうなので、
trunk の nightly build などをしようと思ったら、
ローカルで作成して `docker push` する方がよさそうに感じました。

## aufs の制限

https://github.com/dotcloud/docker/issues/332
によると aufs は重ねられる数に限界 (40ぐらい?) があるようなので、
`RUN` コマンドは出来るだけまとめて減らした方が良いのかもしれません。

https://index.docker.io/u/truongsinh/nodejs/
のように `&&` でつなげて 1 個の `RUN` にまとめている例もありました。

aufs 以外が主流になるかもしれないので、
この辺りは Dockerfile の読みやすさや
`docker history` でわかれていた方が良いのかなど、
利点や欠点を考えつつ、
ベストプラクティスが決まっていくまで
試行錯誤するのが良さそうです。

## まとめ

DOCKER index には Trusted Builds という向こう側で
`docker build` してくれる仕組みがあるので、
`docker push` とうまく使い分けて
公開可能なイメージはどんどん公開すると良いのではないでしょうか。
