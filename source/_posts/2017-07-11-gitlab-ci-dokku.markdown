---
layout: post
title: ".gitlab-ci.ymlでDokkuにdeployする"
date: 2017-07-11 22:15:00 +0900
comments: true
categories: gitlab linux ubuntu dokku
---
GitLab と Dokku (と一部 Heroku) を使って CI/CD (Continuous Integration / Continuous Deployment) 環境を作ってみた話の続きです。
CI 部分のメインとなる `.gitlab-ci.yml` の設定の話です。

[gitlab カテゴリー](/blog/categories/gitlab/)で一覧が見えます。

<!--more-->

## 対象バージョン

- Ubuntu 16.04.2 LTS
- Omnibus GitLab 9.1.4-ce.0 (インストール時) から 9.3.5-ce.0 (記事執筆時)
- gitlab-ci-multi-runner 9.2.0 (インストール時) から 9.3.0 (記事執筆時)
- Dokku 0.9.4 (インストール時) から 0.10.2 (記事執筆時)

## 構成

[GitLab Continuous Integration (GitLab CI)](https://docs.gitlab.com/ce/ci/README.html)の冒頭の画像にあるように、
git push すると CI が走り、 master 以外のブランチなら review 環境にデプロイされて、
master ブランチなら staging 環境にデプロイして、その後、手動で production 環境にデプロイできる、
という状態にします。

## .gitlab-ci.yml

この後は `.gitlab-ci.yml` に設定する内容の説明になります。

## image 設定

ここでは [library/ruby - Docker Hub](https://hub.docker.com/r/_/ruby/) の 2.3.3 を使いました。
`.ruby-version` や `Gemfile` と同じバージョンを指定します。

    image: ruby:2.3.3

2.3 だと 2.3.4 になるので、 2.3.3 まで指定しています。
2.3.4 はセキュリティアップデートではなかったので、まだ 2.3.3 のままですが、様子をみてあげる予定です。

## cache 設定

[cache:key](https://docs.gitlab.com/ce/ci/yaml/README.html#cache-key) から per-branch caching を選んで以下のように設定しました。

    cache:
      key: "$CI_COMMIT_REF_NAME"
      untracked: true

## 環境変数設定

[library/postgres - Docker Hub](https://hub.docker.com/r/_/postgres/) で使う `POSTGRES_PASSWORD` などと、それに接続するための Rails 用の `DATABASE_URL`、
デプロイ用の省略表記のための `DOKKU` などを設定しています。

`APP_NAME` と `DB_NAME` は `CI_ENVIRONMENT_SLUG` を使ってブランチごとに自動生成される名前を使っています。

    variables:
      # for test
      POSTGRES_DB: dbname
      POSTGRES_USER: dbuser
      POSTGRES_PASSWORD: dbpass
      DATABASE_URL: "postgres://dbuser:dbpass@postgres:5432/dbname"
      # for deploy
      DOKKU: ssh dokku@$DOKKU_HOST
      APP_NAME: $CI_ENVIRONMENT_SLUG
      DB_NAME: $CI_ENVIRONMENT_SLUG-database

## stages 設定

最初に説明したように、 test の後に review、 staging の後に production となるように stages を設定します。

    stages:
      - test
      - review
      - staging
      - production

## before_script

テスト用の before_script を設定しています。(デプロイの方は個別に上書きしています。)

apt や bundler ではキャッシュ用のディレクトリである `/cache` を使うように指定しています。

開発環境と共通になっている都合上、 sqlite3 を入れています。
JavaScript ランタイムも必要なので、 nodejs も入れています。

    before_script:
      - 'apt-get update -qq && apt-get -o dir::cache::archives="/cache/apt" install -y -qq sqlite3 libsqlite3-dev nodejs'
      - ruby -v
      - gem install bundler --no-ri --no-rdoc
      - bundle install --jobs $(nproc) --path=/cache/bundler
      - ln -nfs .test.env .env

## デプロイ用の before_script

[Hidden keys](https://docs.gitlab.com/ce/ci/yaml/README.html#hidden-keys)に書いてあるようにキーが `.` で始まるものは無視されるので、
YAML のアンカーを使って参照用に使えます。

ruby image には openssh-client は入っていたのですが、他の image に変えても動くように参考にした [Using SSH keys](https://docs.gitlab.com/ce/ci/ssh_keys/README.html) に書いてあった通り、 openssh-client のインストール手順も入れています。

`~/.ssh/config` で Hostname や Port や User などを指定したかったことがあったので、設定できるようにしました。

    .before_ssh: &before_ssh
      # https://docs.gitlab.com/ce/ci/ssh_keys/README.html
      - 'which ssh-agent || ( apt-get update -y && apt-get -o dir::cache::archives="/cache/apt" install -y openssh-client )'
      - eval $(ssh-agent -s)
      - ssh-add <(echo "$SSH_PRIVATE_KEY")
      - mkdir -p ~/.ssh
      # Set `ssh-keyscan $DOKKU_HOST` to SSH_SERVER_HOSTKEYS
      - '[[ -f /.dockerenv ]] && echo "$SSH_SERVER_HOSTKEYS" > ~/.ssh/known_hosts'
      - '[[ -f /.dockerenv ]] && echo "$SSH_CONFIG" > ~/.ssh/config'

## GitLab の Web での環境変数設定

ここで `.gitlab-ci.yml` の話は中断して、 Secret variables の設定の話です。

GitLab で該当プロジェクトを開いて、 Settings の Pipelines から Secret variables で `SSH_PRIVATE_KEY` などを設定します。

ここでは、以下のように設定しました。

- `DOKKU_HOST` : Dokku に ssh するときのホスト名 (`dokku.example.com` など)
- `DOKKU_DOMAIN` : Dokku の VHOST に設定したドメイン (`10.1.2.3.xip.io` など)
- `SSH_PRIVATE_KEY` : 秘密鍵 (`~/.ssh/id_gitlab` など) の内容
- `SSH_SERVER_HOSTKEYS` : `ssh-keyscan $DOKKU_HOST` の出力
- `PRODUCTION_DOKKU_HOST` : production 用の Dokku に ssh するときのホスト名
- `SSH_CONFIG` : `~/.ssh/config` に設定したい内容

`SSH_PRIVATE_KEY` は Protected を Yes にしておくと良さそうですが、他は No のままでも良さそうな気がします。

## review 環境への deploy 用 script

まず Dokku でアプリとデータベースを作成して接続します。
2度目以降は同じアプリを更新するので、作成などのエラーは無視します。

環境変数は `TZ` と `RAILS_SERVE_STATIC_FILES` あたりがほぼ必須だと思いますが、他は staging 環境や review 環境用に独自に設定できるようにしています。
`RACK_DEV_MARK_ENV` は rack-dev-mark gem の設定です。

`Procfile` で web だけではなく clockwork gem を使ったプロセスも動かしている関係で `letter_opener` 用のディレクトリをマウントしていますが、
`/usr/bin/find '/var/lib/dokku/data/storage/letter_opener' -mtime '+2' -delete` のような感じで古いファイルは自動削除する予定です。
(自動削除はまだしていません。)

デプロイ本体部分の `git push` は、単純に `master` だとうまくいかないことがあったので、 `HEAD:refs/heads/master` という指定にしています。
Heroku にデプロイするときも git でデプロイするなら同様になります。

`db:seed` の実行は `ssh` に `-tt` をつけて強制的に tty を確保する必要がありました。

    .deploy_script: &deploy_script
      - $DOKKU apps:create $APP_NAME || echo $?
      # require `sudo dokku plugin:install https://github.com/dokku/dokku-postgres`
      - $DOKKU postgres:create $DB_NAME || echo $?
      - $DOKKU postgres:link $DB_NAME $APP_NAME || echo $?
      - $DOKKU config:set --no-restart $APP_NAME
        TZ=Asia/Tokyo
        RAILS_SERVE_STATIC_FILES=1
        NO_FORCE_SSL=1
        USE_LETTER_OPENER_WEB=1
        RACK_DEV_MARK_ENV=review
        RACK_ENV=review
        RAILS_ENV=review
      - $DOKKU storage:mount $APP_NAME /var/lib/dokku/data/storage/letter_opener/$APP_NAME:/app/tmp/letter_opener || echo $?
      - git push dokku@$DOKKU_HOST:$APP_NAME HEAD:refs/heads/master
      - $DOKKU -tt run $APP_NAME bundle exec rake db:seed db:seed_fu

## test stage のジョブ

postgres を使って rake でテストを走らせます。

データベースの設定は `DATABASE_URL` で指定しているので、 `config/database.yml` は特に何もしていません。

    rake:
      stage: test
      services:
        - postgres:latest
      script:
        - bundle exec rake db:setup RAILS_ENV=test
        - bundle exec rake

rubocop なども使うなら同様に設定します。

## production 環境への deploy

順番が前後しますが、最初に production 環境への deploy 設定です。

production 環境はちゃんと名前が決まっているので `APP_NAME` を上書きします。

`before_script` はテスト環境用のものを `before_ssh` で上書きします。
(デプロイ用のジョブの方が多いので逆にテスト用のジョブで `before_script` を上書きする方がよかったかもしれません。)

production 環境は環境がすでに整っているはずなので、デプロイは `git push` による更新のみです。

environment を設定することで GitLab の Web のプロジェクトの Pipelines の Environments からリンクが貼られます。

`when: manual` にすることで手動で開始するようにしています。

only で master のみに制限しています。

    production:
      stage: production
      variables:
        APP_NAME: hello-app
      before_script: *before_ssh
      script:
	    - git push dokku@$PRODUCTION_DOKKU_HOST:$APP_NAME HEAD:refs/heads/master
      environment:
        name: production
        url: https://hello-app.example.jp/
      when: manual
      only:
        - master

## staging 環境への deploy

production 環境と同様に staging 環境への deploy 設定をしています。

テストが通ったら自動実行されるのと `APP_NAME` や URL などが違う以外は基本的に production と同じです。

    staging:
      stage: staging
      variables:
        APP_NAME: hello-app-staging
      before_script: *before_ssh
      script:
	    - git push dokku@$PRODUCTION_DOKKU_HOST:$APP_NAME HEAD:refs/heads/master
      environment:
        name: staging
        url: https://hello-app-staging.example.jp/
      only:
        - master

## review 環境への deploy

review 環境への deploy も似たような感じです。

deploy 本体の script は事前に定義した `deploy_script` を使います。
結局ここでしか使っていないので、直接ここに書いても良かったかもしれません。

environment は name に `review/` をつけることで複数の review 環境が同時に存在している時に折りたたまれるようになります。

review 環境は動的に作ったり消したりするので、 https ではなく http になっています。

`on_stop` を指定することで環境の削除ジョブを指定できます。

only と except で master 以外のブランチの時に review 環境が作成されるようにしています。

    review:
      stage: review
      before_script: *before_ssh
      script: *deploy_script
      environment:
        name: review/$CI_COMMIT_REF_NAME
        url: http://$CI_ENVIRONMENT_SLUG.$DOKKU_DOMAIN
        on_stop: stop_review
      only:
        - branches
      except:
        - master

## review 環境の削除

`action: stop` で環境を削除するジョブとして設定しています。

`when: manual` で手動実行するように設定していますが、基本的にはマージリクエストがマージされた時に Remove source branch にチェックを入れて、自動で停止しています。

`GIT_STRATEGY: none` で git 関連の操作はせずに速やかに停止処理のみするようにしています。

postgres は使用中だと停止できないので、先にアプリケーションを削除してからデータベースを削除しています。
自動実行なので `--force` で確認なしに削除するようにしています。

    stop_review:
      stage: review
      variables:
        GIT_STRATEGY: none
      before_script: *before_ssh
      script:
        - $DOKKU apps:destroy $CI_ENVIRONMENT_SLUG --force || echo $?
        - $DOKKU postgres:destroy $CI_ENVIRONMENT_SLUG-database --force || echo $?
      environment:
        name: review/$CI_COMMIT_REF_NAME
        action: stop
      when: manual
      only:
        - branches
      except:
        - master

## まとめ

GitLab CI と Dokku を組み合わせて CI/CD 環境を作る例を紹介しました。

ちゃんと動く review 環境にするには seed の設定だったり、メール関係の設定だったり、色々とアプリケーション側でも対応が必要ですが、一度環境ができてしまえば変更の確認を他の人にもしてもらいやすくなるので、便利になるのではないでしょうか。

次は staging 環境と production 環境に Heroku を使う例を紹介します。

GitLab + Dokku 関連の記事は[gitlab カテゴリー](/blog/categories/gitlab/)で一覧が見えます。
