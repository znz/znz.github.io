---
layout: post
title: "font-awesome-sass を 4.1.0 から 4.2.0 にあげたらアイコンが表示されなくなったので対処した"
date: 2014-09-03 20:52:56 +0900
comments: true
categories: rails
---
Rails 4.1.5 で使っている `font-awesome-sass` gem を 4.1.0 から 4.2.0 にあげたところ、
アイコンが表示されなくなったので、原因を調べてみました。

## 解決方法

https://github.com/FortAwesome/font-awesome-sass には 3.x からのアップグレード方法しか
書いていないのですが、
4.1.0 などの 4.x 系から 4.2.0 にあげるときは

    *= require font-awesome

の代わりに

    @import "font-awesome-sprockets";
    @import "font-awesome";

のように `@import` を使う必要があり、
既に `@import` を使っている場合でも
`@import "font-awesome-sprockets";`
の行の追加が必要でした。

## 詳細

4.1.0 と 4.2.0 の差分の
[Refactoring for use in multiple Ruby envs and upgrading to FontAwesome 4.2.0](https://github.com/FortAwesome/font-awesome-sass/commit/a527acdf693cf0bced797e75f387a8f8e2a9c844 "Refactoring for use in multiple Ruby envs and upgrading to FontAwesome 4.2.0")
を眺めてみると、
FontAwesome を 4.2.0 にあげる以外に
`vendor/assets/` から `assets/` に移動していたり、
それに関係する変更をしていたりするようです。

それから詳しいことはわかりませんが、
`icon-font-path($path)` と `icon-image-path($path)` で
それぞれ
sprockets では `font-path($path)` と `image-path($path)` で、
compass では `font-url($path, true)` と `image-url($path, true)` を
使うようになった影響で `@import "font-awesome-sprockets"` が必要になったようです。
