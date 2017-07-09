---
layout: post
title: "GitLab Runner の設定"
date: 2017-07-09 14:00:00 +0900
comments: true
categories: gitlab linux ubuntu
---
GitLab と Dokku (と一部 Heroku) を使って CI/CD (Continuous Integration / Continuous Deployment) 環境を作ってみた話の続きです。
CI 部分のメインとなる GitLab Runner の設定の話です。

[gitlab カテゴリー](/blog/categories/gitlab/)で一覧が見えます。

<!--more-->

## 対象バージョン

- Ubuntu 16.04.2 LTS
- Omnibus GitLab 9.1.4-ce.0 (インストール時) から 9.3.5-ce.0 (記事執筆時)
- gitlab-ci-multi-runner 9.2.0 (インストール時) から 9.3.0 (記事執筆時)
- Dokku 0.9.4 (インストール時) から 0.10.2 (記事執筆時)

## GitLab Runner とは?

[GitLab Runner](https://docs.gitlab.com/runner/) とは GitLab CI のジョブを実行する部分のことです。

Jenkins でいうと (ジョブを実行しない設定にした) master が GitLab CI で、 slave が GitLab Runner に相当します。

## 名前について

昔は gitlab-ci-multi-runner という名前だったらしく、 repository は https://gitlab.com/gitlab-org/gitlab-ci-multi-runner になっています。

GitLab CI は GitLab 8.x の時に GitLab 本体に統合されたということで、古いドキュメントを参照するときは注意が必要そうです。

## executor 選択

GitLab Runner を入れたマシン上で直接実行する Shell executor や Docker で実行する Docker executor などがありますが、一番簡単で便利そうな Docker executor を使うことにしました。
また、管理を簡単にするために全てのプロジェクトで共通の Runner を使うことにしました。

必要なら複数登録してタグで使い分けることもできます。
プロジェクトごとに使い分けることもできます。

## インストール

GitLab とは別マシンが推奨のようなので、別仮想マシンを用意しました。
[Install GitLab Runner using the official GitLab repositories](https://docs.gitlab.com/runner/install/linux-repository.html) の方法でパッケージをインストールしました。

「Note: Debian users should use APT pinning」とありますが、[Debian の gitlab-ci-multi-runner パッケージ](https://packages.debian.org/gitlab-ci-multi-runner)をみてもリリースされた Stretch には入っていないようです。[Ubuntu の gitlab-ci-multi-runner パッケージ](https://packages.ubuntu.com/gitlab-ci-multi-runner)も xenial (16.04LTS) には入っていないようなので、 APT pinning はなくても良さそうです。(今回の設定では念のため入れています。)

## GitLab CI に登録

[Registering Runners](https://docs.gitlab.com/runner/register/index.html) に書いてあるように `sudo gitlab-runner register` で登録します。
オプションなしで実行すると対話的にいろいろきかれます。

gitlab-ci coordinator URL は古いドキュメントだと `/ci` が付いていることがあるようですが、今はあってもなくてもどちらでも大丈夫のようです。

自動設定するなら token は GitLab 側の初期設定をする時に `gitlab_rails['initial_shared_runners_registration_token']` で固定しておくと良いと思います。

タグを空欄にした場合はタグなしのジョブも実行されるのですが、タグを設定した場合は run untagged jobs も設定しておかないとジョブが実行されなくて悩むことになります。
(タグは GitLab の Web UI で後から空欄にできなくて、実行されない条件を絞り込むのに unregister して register し直す必要があって面倒だったのですが、当時のバージョンのバグだったのか、GitLab 9.3.5 では問題なく空欄に変更できました。)

Docker executor を選んだ時のデフォルトの Docker image は `hello-world` ぐらいの使いにくいものにしておいて、 `.gitlab-ci.yml` で image を常に指定する方が Runner の設定に影響されないので無難かもしれません。

## GitLab CI から登録解除

`gitlab-runner list` で token などを確認して `gitlab-runner unregister` で登録解除できます。

unregister せずに GitLab Runner がいなくなってしまった場合は GitLab の Admin Area の Runners から Remove できます。

## 同時実行数指定

`/etc/gitlab-runner/config.toml` の `concurrent =` で同時に実行するジョブ数を指定できます。
自動で再読み込みしているらしく、試した感じだと runner の restart をしなくても設定が反映されるようでした。

## .gitlab-ci.yml の Lint

[GitLab CIの設定ファイルを手元でLintする](http://qiita.com/sei40kr/items/407e08cc45f218738d4c)によると `/api/v4/ci/lint` で Lint できるようなので、 curl と jq をインストールしておいて、

    linter_api_url='https://gitlab.com/api/v4/ci/lint'
    curl -sH 'Content-Type: application/json' "$linter_api_url" -d "$(jq -Rs '{"content":.}' < .gitlab-ci.yml)" | jq

のようにすれば良いようです。

`linter_api_url` を自前の GitLab の URL にしておけば外部に送信することなく Lint できます。

## GitLab Container Registry と連携

Docker executor を使って GitLab Container Registry を扱いたい場合、 Docker executor と同じホスト側の docker を使う方法と docker in docker を使う方法があります。

詳細は [Using Docker Build](https://docs.gitlab.com/ce/ci/docker/using_docker_build.html) を参考にしてください。

GitLab Container Registry にちゃんとした https の証明書が設定できる場合は良いのですが、テスト環境などで http しか有効にできない場合は困ることがあります。

### ホスト側の docker を使う方法

ホスト側の docker を使うということは、ホスト側の docker の権限を GitLab CI で使えるようにしてしまうということなので、信頼できないユーザーが `.gitlab-ci.yml` を操作できる環境では使えません。

使い方としては GitLab Runner の register の時に `--docker-volumes /var/run/docker.sock:/var/run/docker.sock` を指定します。

GitLab Container Registry が http のみの場合は、ホスト側の Docker に insecure-registry の設定をすることになります。

`/etc/systemd/system/docker.service.d/local.conf` などで `ExecStart` を上書きして設定するか、 `/etc/docker/daemon.json` に

    {"insecure-registries": ["registry.example.test"]}

のように設定することで http で接続できるようになります。
(daemon.json では[デーモン設定ファイル](http://docs.docker.jp/engine/reference/commandline/daemon.html#daemon-configuration-file)に書いてあるようにコマンドラインオプションの `--insecure-registry` が複数形の `"insecure-registries"` になります。 `dns` のように複数形でも変わらないものもあるので、指定できるオプションはドキュメントを参照するのが無難です。)

### docker in docker を使う方法

`.gitlab-ci.yml` の services に `docker:dind` を指定して使う、という方法になります。
`docker:dind` サービスに insecure-registry を設定する方法がないので、試していませんが http のみで使いたい場合は派生 image を自分で作るなどの対応が必要そうです。

使い方としては GitLab Runner の register の時に `--docker-privileged` を指定します。
register の時に指定し忘れていた場合は、後から config.toml に `privileged = true` を追加でも良いと思います。

### .gitlab-ci.yml での設定

[Using Docker Build](https://docs.gitlab.com/ce/ci/docker/using_docker_build.html) に書いてあるように

    - docker login -u gitlab-ci-token -p $CI_JOB_TOKEN registry.example.com

か

    - docker login -u gitlab-ci-token -p $CI_JOB_TOKEN $CI_REGISTRY

のようにログインした後、 build や push などができるようになります。

`gitlab-ci-token` も固定ではなく環境変数を参照するようにできますが、固定で書いてある設定例が多いので、変わる可能性は低そうなので、固定で書いておいても良いと思います。

## まとめ

GitLab Runner の設定と Lint の使い方、 GitLab Container Registry と連携方法を紹介しました。
次は Dokku の設定になります。

[gitlab カテゴリー](/blog/categories/gitlab/)で一覧が見えます。
