---
layout: post
title: "heroku で passenger を使うようにした"
date: 2014-01-24 23:23:25 +0900
comments: true
categories: ruby heroku passenger
---
今は heroku で動いている https://bugs.ruby-lang.org/ が thin から passenger に切り替えて安定したという話を IRC で知ったので、自分が heroku で動かしている Rails アプリも passenger に切り替えてみました。
(ruby-lang.org の方はいろいろ試すために今後他のものに切り替える可能性もあります。)

<!--more-->

## 概要

https://github.com/phusion/passenger-ruby-heroku-demo
を参考にして Gemfile と Procfile を設定するだけです。

## Gemfile の書き換え

`gem "unicorn"` や  `gem "thin"` があればコメントアウトするなり削除するなりして、
代わりに `gem "passenger"` を追加します。

## Procfile の設定

Procfile がなければ新規作成します。
既存の Procfile があって以下のような行があれば削除します。


    web: bundle exec ruby web.rb -p $PORT
    web: bundle exec unicorn -p $PORT
    web: bundle exec thin start -p $PORT

Procfile に以下の設定を追加します。

    web: bundle exec passenger start -p $PORT --max-pool-size 3

## 切り替え反映

以下のように切り替えを反映します。

    bundle install
    git commit -a -m "Switch to Phusion Passenger"
    git push heroku master

ここまでは
https://github.com/phusion/passenger-ruby-heroku-demo
の手順そのままです。

## ローカルで使う

`gem install foreman` で `foreman` を入れておいて `env PORT=3000 foreman start` とか、
`bundle exec passenger start -p 3000 --max-pool-size 3` とかで
3000 番ポートで passenger を使えます。

初回起動時は standalone 版 passenger の初期設定が走るので、
少し時間がかかります。

## まとめ

あまりアクセスが多くない無料の範囲内で使っている Rails アプリなので、
違いはあまりわかっていませんが、簡単に切り替えられるので、
選択肢のひとつとして使われやすくなるのではないかと思います。
