---
layout: post
title: "redmine の運用停止前のバックアップ"
date: 2014-10-09 21:55:27 +0900
comments: true
categories: ruby rails redmine
---
ある redmine の運用を止める前にリポジトリとの連携部分は止まってしまうのは仕方がないとして、
wiki やチケットなどは後からでも参照できるようにローカルにバックアップを残しておくことにしました。
その手順のメモです。

<!--more-->

## データベースのバックアップ

[yaml_dbでMySQLからPostgreSQLに移行した](http://blog.n-z.jp/blog/2014-01-28-yaml-db.html "yaml_dbでMySQLからPostgreSQLに移行した")話で使った `yaml_db` を使いました。

まず、
`Gemfile.local` をなければ作成して、以下の内容を追加します。

```ruby Gemfile.local
gem 'yaml_db', github: 'jetthoughts/yaml_db'
```

そして `RAILS_ENV=production bundle exec rake db:data:dump` で `db/data.yml` を作成します。

## ローカル環境の作成

ローカルにも redmine 環境を作成します。

1. redmine-2.5.2.tar.gz をダウンロードして展開
2. `config/database.yml.example` を元に `config/database.yml` を作成 (今回はバックアップ環境なので sqlite3 を使用)
3. Gemfile.local 作成
4. `plugins/` に使っていたプラグインをインストール (サーバーから rsync でコピーなど)
5. プラグインの設定も必要に応じてコピー (`config/wiki_external_filter.yml` など)
6. `bundle install` で gem のインストール
7. `RAILS_ENV=production bundle exec rake db:migrate` と `RAILS_ENV=production bundle exec rake redmine:plugins:migrate` でデータベース作成
8. `db/data.yml` をコピーしてきて `RAILS_ENV=production bundle exec rake db:data:load` で読み込み (`db:migrate` していないとテーブル不足で読み込めない)
9. `files/` の添付ファイルをコピー (サーバーから rsync でコピーなど)
10. `config/configuration.yml` や `favicon.ico` などカスタマイズしているものがあれば必要に応じて設定やコピー
11. `bundle exec rake generate_secret_token` で secret token を生成
12. `bundle exec rails server -e production` で起動

ログインして今まで通り見えることを確認して終了です。
