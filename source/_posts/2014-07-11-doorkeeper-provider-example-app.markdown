---
layout: post
title: "Rails 4.1 で Doorkeeper を使った OAuth2 Provider のサンプルを実装した"
date: 2014-07-11 23:30:00 +0900
comments: true
categories: ruby rails doorkeeper devise oauth2
---
ソースは github の
[znz/doorkeeper-provider-app](https://github.com/znz/doorkeeper-provider-app "znz/doorkeeper-provider-app")
で公開しています。

基本的にはソースをみて参考にしてもらうと良いと思いますが、
説明が必要な部分を続きに書いてみました。

<!--more-->

## 対象バージョン

- ruby 2.1.2
- rails 4.1.4
- bootstrap 3.2.0
- devise 3.2.4
- devise-i18n-views 0.2.8
- doorkeeper 1.3.1
- cancancan 1.8.4
- rolify 3.4.0
- rspec-rails 3.0.1

## 試し方

README に書いたようにローカルで動かすか heroku に deploy して
[Example Applications](https://github.com/doorkeeper-gem/doorkeeper/wiki/Example-Applications "Example Applications")
にある Client examples の Sinatra and OAuth2 gem の
[Doorkeeper Sinatra Client](https://github.com/doorkeeper-gem/doorkeeper-sinatra-client "Doorkeeper Sinatra Client")
を使って試しました。

## 初期設定

devise, doorkeeper, cancancan, rolify, rspec の個別の初期設定は普通に `rails generate` を使いました。

## ユーザー情報追加

とれる情報を増やすために `User` に `name` を追加しました。
`devise-i18n-views` を使っている関係で view のカスタマイズはしていないので、
`rake db:seed` で設定したユーザーだけ `name` が設定されています。

必要に応じて view もカスタマイズしてください。

また `devise-i18n` の ja.yml を devise.ja.yml として入れています。
これは flash のメッセージやメールのメッセージなど、
`app/views` 以外の翻訳になるようです。

`devise-i18n-views` は `app/views` を翻訳可能な view にするプロジェクトです。
なぜ `devise` とは別プロジェクトでやっているのかはよくわかりません。

## `I18n.available_locales`

`devise-i18n-views` を入れてしまうと `I18n.available_locales` が増えてしまうので、
困るのなら、カスタマイズ用の view を generate して、必要な言語だけ取り込んで
`Gemfile` から外してしまうのが良いと思います。

今回はそのまま残して右上の `Locale` で選択できるようにしています。
選択肢の翻訳は Wikipedia の左や www.debian.org の下などを参考にしたのですが `es-AR` はわからなかったので、
`es` と同じになってしまっています。

## `/oauth/applications` のアクセス制限

`cancancan` と `rolify` を使って admin role があるユーザーだけに制限しています。
secret も見えてしまうので、 read 権限までしっかり制限する必要があるようです。

`load_and_authorize_resource` でのロードと親クラス (`Doorkeeper::ApplicationsController`) の中でのロードでモデルの読み込みが二重になってしまうのですが、変更を少なくするためにそこは許容しました。

## `GET /api/v1/me.json`

Doorkeeper gem の Wiki の例にあるようにユーザー情報をとれるようにしています。
制限していないと以下のような情報がとれました。

```json
{ "id": 1,
  "email": "admin@example.com",
  "created_at": "2014-07-11T06:32:22.077Z",
  "updated_at": "2014-07-11T09:33:42.143Z",
  "name": "admin" }
```

制限したり関連するモデルの情報を増やしたりするなら
[Rails のモデル関係と to_json(to_xml) - すがブロ](http://sugamasao.hatenablog.com/entry/20100914/1284415669 "Rails のモデル関係と to_json(to_xml) - すがブロ")
に書いてあるように `respond_with` に `:only` をつけたり `:include` をつけたりすると出来るようです。

入ったり入らなかったりする条件がよくわからなかったのですが、
`devise` 関連では `authentication_token` が入っていることがあったので、
User モデルにいろんな情報を入れているなら、
きちんと制限した方が良さそうに思いました。

## microposts

[Ruby on Rails チュートリアル：実例を使って Rails を学ぼう](http://railstutorial.jp/ "Ruby on Rails チュートリアル：実例を使って Rails を学ぼう")
のように `Micropost` モデルを作成して、 API からも投稿できるようにしました。

投稿は scope で制限していて、デフォルトの `public` のみでは書き込めずに `write` も必要にしています。

API としては

- `GET /api/v1/microposts` で投稿一覧
- `POST /api/v1/microposts` で新規投稿

を用意しています。
