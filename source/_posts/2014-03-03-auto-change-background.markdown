---
layout: post
title: "ページの切り替わりごとに背景の自動変更"
date: 2014-03-03 23:25:01 +0900
comments: true
categories: rails coffeescript javascript html5
---
ある Rails アプリでページの切り替わりごとに背景を自動で変更するようにしていたのですが、
デザイン変更で使わなくなったので、どういうことをやっていたのか、メモとして残しておくことにします。

<!--more-->

## HTML

まず CSS 用のクラスとして自動変更する目印の `background` と初期状態の `background-0` を `body` に設定しました。

```haml app/views/layouts/application.html.haml
  %body.background.background-0
```

## スクリプト

`sessionStorage` を使ってタブごとに状態を保存するようにしました。
今回は画像として `app/assets/images/background-0.jpg` と `app/assets/images/background-1.jpg` の 2 枚だけだったので、
`background_num = 2` にしています。

以下のコードを書いた時には知らなかったので対処が入っていないのですが、
Safari のプライベートブラウズでは `sessionStorage` があるのに使えないという厄介なことになっているので、必要ならそのあたりのエラー処理も入れた方が良いです。

```coffeescript app/assets/javascripts/background.js.coffee
jQuery ->
  if typeof sessionStorage != 'undefined'
    background_num = 2
    change_background = ->
      body = $("body")
      if !body.hasClass("background")
        return
      background = sessionStorage.getItem("background")
      if background
        background = parseInt(background)
        $("body").removeClass("background-" + background)
        background = (background + 1) % background_num
        $("body").addClass("background-" + background)
      else
        background = 0
      sessionStorage.setItem("background", background)
    $(document).on 'ajax:success.background', change_background
    change_background()
```

## スタイルシート

rails 4.0.3, sass-rails 4.0.1, sprockets 2.11.0, sprockets-rails 2.0.1
で確認したところ、
`depend_on_asset` ではなく `depend_on` を使って拡張子付きのファイル名で
ファイルの先頭に `depend_on` を並べれば画像ファイルを変更したときに
`rake assets:precompile` で `css` も再生成されるように見えました。

つまり `//= depend_on "background-1.jpg"` を `.background-1` の直前に持っていくと反映されませんでした。

`depend_on_asset` だと全く反映されませんでした。

ファイルの指定は `//= depend_on "../images/background-0.jpg"` や `//= depend_on "../images/background-1.jpg"` のように相対パスでも、ファイル名のみでも大丈夫でした。

`image-url` で参照しているのだから、そこで自動的に依存関係に入ってほしいところですが、そういう仕組みにはなっていないようです。
ちゃんと読んでいないのですが、
[Dependent assets might not be recompiled](https://github.com/sstephenson/sprockets/issues/488)
のあたりにすでに話が出ているので、仕様なのかもしれません。

```css app/assets/stylesheets/background.css.scss
//= depend_on "background-0.jpg"
//= depend_on "background-1.jpg"
.background-0 {
  background-image: image-url("background-0.jpg");
}
.background-1 {
  background-image: image-url("background-1.jpg");
}
```
