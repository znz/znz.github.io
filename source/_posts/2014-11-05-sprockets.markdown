---
layout: post
title: "Rails 4.1.7 にあげたら一部の画像が表示されなくなった"
date: 2014-11-05 21:11:09 +0900
comments: true
categories: ruby rails
---
Rails 4.1.7 にあげたら一部の画像が表示されなくなるという現象が起きて、しばらく悩んでいたのですが解決しました。

<!--more-->

## 原因

- Rails か sprockets のバージョンアップでファイル名につけられる digest が変わった。
- `image_tag` で画像を参照している HTML が `cache` の中にあった。

## 現象

`rails console` で `Rails.application.assets.find_asset(asset_name).digest` を試しても新しい digest がちゃんと返ってくるのに、
ブラウザー上では古い digest の URL が参照されていてファイルが見つからないという現象になっていました。

アセットはキャッシュ期間の長いファイルとして設定しているので、複数ブラウザーで確認すると最初は現象が起きたり起きなかったりしていて、サーバー側の問題だと気づくのが遅れました。

## 対処

`dalli` gem 経由で `memcached` をキャッシュ使っていたので `sudo service memcached restart` してキャッシュを破棄しました。
