---
layout: post
title: "jquery_mobile_rails と production 環境と画像ファイル"
date: 2013-10-02 15:58
comments: true
categories: rails jquery-mobile
---
rails 4.0.0
と
[jquery_mobile_rails](https://rubygems.org/gems/jquery_mobile_rails) 1.3.2
で production 環境だと画像が表示されないという現象が起きていました。

2013-10-03 追記:
`jquery_mobile_rails` gem のファイル配置の問題もあるのかもしれません。
詳細は次の記事を参照してください。

<!--more-->

`RAILS_ENV=production rake assets:precompile`
で調べてみると `public/assets/` 以下には
`jquery_mobile_rails` の画像は入っていませんでした。

いろいろ調べた結果、
`config.assets.precompile`
に
`*.gif`
と
`*.png`
を足せば良いとわかったので、
`jquery_mobile_rails`
用に
`application.css`
や
`application.js`
とは別に作成した
`mobile.css`
や
`mobile.js`
と合わせて、以下のように追加して解決しました。
`jquery_mobile_rails` 1.3.2
には
`*.jpg`
は入っていないのですが、
念のため追加しておきました。

```ruby config/environments/production.rb
   config.assets.precompile += %w( mobile.js mobile.css *.gif *.png *.jpg )
```

この状態で再度
`RAILS_ENV=production rake assets:precompile`
で調べてみると `public/assets/`
は以下のようになっていました。

```console
% ls public/assets/jquery-mobile
ajax-loader-5c6592c3263e7de88985668db733b08f.gif
icons-18-black-dbc49700bc8bd5a9d3e0120cb111ea62.png
icons-18-white-f8f5999f3ea0d9ebea6a4ec193442c1f.png
icons-36-black-76b4944fda10128b365344f06377dad5.png
icons-36-white-178cc38be265514b341e111ed7d38712.png
```

rails 4 になって、ハッシュなしの
`ajax-loader.gif`
のようなファイルは作成されなくなっているので、
ちゃんと view の中では
`image_path`
を使ったり、
スタイルシートの中では
`image-url`
を使ったりしていないと
production
環境だけ問題がおきることがあるようです。
