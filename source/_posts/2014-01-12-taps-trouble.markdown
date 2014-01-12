---
layout: post
title: "tapsでMySQLからPostgreSQLに移行した後に起きたトラブル事例"
date: 2014-01-12 14:15:41 +0900
comments: true
categories: redmine rails mysql postgresql
---
2013年7月ごろに [taps](http://rubygems.org/gems/taps) を使って redmine のデータベースを MySQL から PostgreSQL に移行しました。

その時に移行が不十分で問題が少し起きていたので、その原因を調べて修正しました。

<!--more-->

## taps での移行

taps の使い方については既に情報がたくさんあるので、簡単に書いておくだけにします。
昨年7月時点の情報なので、今だとさらにバージョン指定を増やさないとダメかもしれません。

まず、 Ruby 1.9.1 以降に対応できていないようなので、 Ruby 1.8.7 を使いました。
rack も新しいバージョンに対応できていないので、以下のような Gemfile でバージョンを指定して`bundle install --path vendor/bundle` して使いました。

```ruby Gemfile
  source "https://rubygems.org"
  gem "taps"
  gem "sqlite3"
  gem "mysql"
  gem "pg"
  gem "rack", "1.0.1"
```

元の `config/database.yml` の設定が

```yaml config/database.yml
production:
  adapter: mysql2
  database: redmine_db
  host: localhost
  username: redminemyuser
  password: my_password
  encoding: utf8
```

という感じなら、 MySQL を動かしている方で
`bundle exec taps server mysql://redminemyuser:my_password@localhost/redmine_db tapsuser tapspass`
と起動します。

PostgreSQL の方では

```console sudo -u postgres psql
CREATE ROLE redminepguser LOGIN ENCRYPTED PASSWORD 'pg_password' NOINHERIT VALID UNTIL 'infinity';
CREATE DATABASE redmine_db WITH ENCODING='UTF8' OWNER=redminepguser;
```

という感じでデータベースを作っておいて、
`bundle exec taps pull postgres://redminepguser:pg_password@localhost/redmine_db http://tapsuser:tapspass@localhost:5000`
という感じでデータを移行しました。

同じサーバーで移行したので taps の http も localhost にしていますが、別サーバーなら ssh のポートフォワーディングなどを使ってつなげば良いと思います。

## redmine でのトラブル

管理者権限でログインして、管理からグループを追加 (/groups/new) しようとするとエラーになって、 log/production.log を確認すると
`ActiveRecord::StatementInvalid (PG::NotNullViolation: ERROR:  null value in column "mail_notification" violates not-null constraint`
または
`(ActiveRecord::StatementInvalid (PG::NotNullViolation: ERROR:  列"mail_notification"内のNULL値はNOT NULL制約違反です`
というエラーが出ていました。

## トラブルの原因

いろいろ調べてみると `db/schema.rb` でいうと `mail_notification` に default がないのが問題だとわかりました。
後で追加調査したところ、 default が `""` (空文字列) のときにうまく移行できていないことがあるようでした。

## 原因修正

`rails dbconsole` で
`ALTER TABLE users ALTER mail_notification SET DEFAULT '';`
で設定して、
`rake db:migrate` で `db/schema.rb` にも反映して
redmine を再起動すると直りました。

## その他の DEFAULT の修正

別途 PostgreSQL の環境を用意して redmine を新規インストールして `rake db:migrate` で生成した `db/schema.rb` と比較して全体としては以下の修正をしました。

```sql rails dbconsole
ALTER TABLE attachments ALTER filename SET DEFAULT '';
ALTER TABLE attachments ALTER disk_filename SET DEFAULT '';
ALTER TABLE attachments ALTER content_type SET DEFAULT '';
ALTER TABLE attachments ALTER digest SET DEFAULT '';
ALTER TABLE auth_sources ALTER type SET DEFAULT '';
ALTER TABLE auth_sources ALTER name SET DEFAULT '';
ALTER TABLE auth_sources ALTER account_password SET DEFAULT '';
ALTER TABLE boards ALTER name SET DEFAULT '';
ALTER TABLE changes ALTER action SET DEFAULT '';
ALTER TABLE comments ALTER commented_type SET DEFAULT '';
ALTER TABLE custom_fields ALTER type SET DEFAULT '';
ALTER TABLE custom_fields ALTER name SET DEFAULT '';
ALTER TABLE custom_fields ALTER field_format SET DEFAULT '';
ALTER TABLE custom_fields ALTER regexp SET DEFAULT '';
ALTER TABLE custom_values ALTER customized_type SET DEFAULT '';
ALTER TABLE documents ALTER title SET DEFAULT '';
ALTER TABLE enumerations ALTER name SET DEFAULT '';
ALTER TABLE wiki_contents ALTER comments SET DEFAULT '';
ALTER TABLE issue_categories ALTER name SET DEFAULT '';
ALTER TABLE issue_relations ALTER relation_type SET DEFAULT '';
ALTER TABLE issue_statuses ALTER name SET DEFAULT '';
ALTER TABLE issues ALTER subject SET DEFAULT '';
ALTER TABLE journal_details ALTER property SET DEFAULT '';
ALTER TABLE journal_details ALTER prop_key SET DEFAULT '';
ALTER TABLE journals ALTER journalized_type SET DEFAULT '';
ALTER TABLE messages ALTER subject SET DEFAULT '';
ALTER TABLE news ALTER title SET DEFAULT '';
ALTER TABLE news ALTER summary SET DEFAULT '';
ALTER TABLE projects ALTER name SET DEFAULT '';
ALTER TABLE projects ALTER homepage SET DEFAULT '';
ALTER TABLE queries ALTER name SET DEFAULT '';
ALTER TABLE repositories ALTER url SET DEFAULT '';
ALTER TABLE repositories ALTER login SET DEFAULT '';
ALTER TABLE repositories ALTER password SET DEFAULT '';
ALTER TABLE repositories ALTER root_url SET DEFAULT '';
ALTER TABLE roles ALTER name SET DEFAULT '';
ALTER TABLE settings ALTER name SET DEFAULT '';
ALTER TABLE tokens ALTER action SET DEFAULT '';
ALTER TABLE tokens ALTER value SET DEFAULT '';
ALTER TABLE trackers ALTER name SET DEFAULT '';
ALTER TABLE users ALTER hashed_password SET DEFAULT '';
ALTER TABLE users ALTER firstname SET DEFAULT '';
ALTER TABLE users ALTER mail SET DEFAULT '';
ALTER TABLE users ALTER language SET DEFAULT '';
ALTER TABLE versions ALTER name SET DEFAULT '';
ALTER TABLE versions ALTER description SET DEFAULT '';
ALTER TABLE watchers ALTER watchable_type SET DEFAULT '';
ALTER TABLE wiki_content_versions ALTER compression SET DEFAULT '';
ALTER TABLE wiki_content_versions ALTER comments SET DEFAULT '';
ALTER TABLE users ALTER mail_notification SET DEFAULT '';
ALTER TABLE users ALTER lastname SET DEFAULT '';
```

```sql rails dbconsole
ALTER TABLE users ALTER mail_notification SET DEFAULT '';
```

はすでに実行済みで、

```sql rails dbconsole
ALTER TABLE users ALTER lastname SET DEFAULT '';
```

は既に default が `""` になっていたので不要でしたが、
重複して実行しても問題が起きないようなので、上の SQL にも入れています。

## まとめ

最初のデータベース選択は重要で、アプリケーションが両方対応していても、後から変更するのは大変です。

問題が起きそうなデータ型を使っていなければ `taps` ですんなり移行できるように見えますが、別途クリーンな環境を用意して生成した `db/schema.rb` と比較するなどのチェックも必要でした。
