---
layout: post
title: "Rail 3.2でcache_digestsを使ってみた"
date: 2013-12-08 00:00
comments: true
categories: ruby rails
---
まだ Rails 3.2.16 のままのアプリで `cache_digests` gem を使って
fragment cache を導入してみました。

Rails 4.0 では標準になっているはずなので、
使い方は同じだと思います。

<!--more-->

## インストール

まず `Gemfile` に以下の gem の指定を追加してインストールしました。

```ruby Gemfile
gem 'cache_digests'
gem 'dalli', group: :production
```

## 環境設定

キャッシュの影響の確認などのデバッグ用に `development` 環境でも
キャッシュを有効にしました。
ちゃんとキャッシュで来ているかどうかの確認や削除がしやすいように
保存先はデフォルトの `:file_store` のままにしています。

```ruby config/environments/development.rb
  config.action_controller.perform_caching = true
```

`production` 環境では `dalli` を使って `memcached` に保存するようにしました。
`dalli` の設定はこの記事の本題ではないので、
`memcached` の設定などは他のサイトを参考にしてください。

```ruby config/environments/production.rb
  config.cache_store = :dalli_store
```

`staging` 環境もあったので `config/environments/staging.rb` にも同様の設定をしました。

## fragment cache

すでに
`app/views/comments/_comment.html.haml` や
`app/views/posts/_post.html.haml` のような view を使って
`render @comments` や `render @posts` のように使っていたので、

```text app/views/comments/_comment.html.haml
- cache comment do
  -# 今までの内容
```

のように `cache comment do ... end` で今までの内容をくくるだけでした。
`cache` メソッド自体の返り値は `=` で埋め込んだりせずにそのまま呼ぶだけで大丈夫でした。

## 動作確認

`log/developement.log` などを `fragment` で検索してみると

```text log/developement.log
Write fragment views/comments/29-20130905083500/96f0ec0ce36af8132826f3bfbe0079db 0.5ms
Read fragment views/comments/30-20130905083518/96f0ec0ce36af8132826f3bfbe0079db 0.4ms
```

などと記録されていて、キャッシュが使われていることが確認できました。

`development` 環境では実際のキャッシュファイルは `tmp/cache/` 以下にありました。

## キャッシュの無効化 (invalidate)

キャッシュが古くなってもう有効ではないという状態にすることを invalidate というと思いますが、
内容が更新された時に古いキャッシュが使われると問題があるので、
その対処をする必要があります。

`cache` メソッドの引数に `ActiveRecord` のオブジェクトを渡した時の
キャッシュのキーは先ほどの例だと
`views/comments/:id-:updated_at/:md5`
という感じで `comment` オブジェクトの `id` と `updated_at` と
`app/views/comments/_comment.html.haml` の MD5 が使われていて、
view のファイルが変更したり、
`comment` ファイルの `updated_at` を更新したりした時に
自動で無効になるようです。

つまり、この view の中で別の partial render を使っていると
反映されないということなので、
内側の方でも `cache` を使うなどの対処が必要そうです。
実際に `_post.html.haml` の中で `render post.comments` のようなことをしました。

さらに以下のように `belongs_to` に `touch: true` を付けて、
コメントが付いた時に `post` の `updated_at` も更新されるようにしました。

```ruby app/models/comment.rb
  belongs_to :commentable, polymorphic: true, touch: true
```

## キャッシュの完全削除

`rails console` で `Rails.cache.clear` を実行すれば削除できました。
他の sass などのキャッシュも `tmp/cache/` の中にあるので
一緒に削除されてしまうようです。

## まとめ

とりあえず使い始めるための最低限の知識をまとめてみました。

後は
[ASCIIcasts - “Episode 387 - Cache Digests”](http://ja.asciicasts.com/episodes/387-cache-digests)
で説明されている
`rake cache_digests:nested_dependencies`
などを知っておけば良さそうです。
