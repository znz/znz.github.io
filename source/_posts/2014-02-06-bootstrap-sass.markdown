---
layout: post
title: "bootstrap-sass-railsからbootstrap-sassに移行した"
date: 2014-02-06 14:29:27 +0900
comments: true
categories: ruby rails gem
---
今日の `bundle update` は bootstrap の sass の gem を bootstrap 公式のものに移行したのが大きな変更点でした。

<!--more-->

## bootstrap-sass-rails から bootstrap-sass 3.1.0.2

[bootstrap 3.1.0 がリリース](http://blog.getbootstrap.com/2014/01/30/bootstrap-3-1-0-released/)されて少し時間が経ったので、
bootstrap-sass-rails を確認してみたところ、
[DEPRECATION NOTICE](https://github.com/yabawock/bootstrap-sass-rails#deprecation-notice)
に公式サポートされた sass の gem があるので、
そちらに移行を推奨と書かれていたので、移行しました。

bootstrap-sass gem への移行は UPGRADING に書かれているように `@import "twitter/bootstrap";` を `@import "bootstrap";` に書き換えるだけでした。

## bootstrap-sass のカスタマイズ

[Usage](https://github.com/twbs/bootstrap-sass#usage)
によると、一番単純な使い方としては `@import "bootstrap";` だけですが、
[Less variables](http://getbootstrap.com/customize/#less-variables)
を参考にして以下のようにカスタマイズして使うのが良いようです。

Less では変数の接頭辞は `@` ですが、 sass では `$` になることさえわかっていれば、最低限のカスタマイズは出来ると思います。

```
$navbar-default-bg: #312312;
$light-orange: #ff8c00;
$navbar-default-color: $light-orange;

@import "bootstrap";
```

bootstrap-sass-rails からの移行だったので、
bootstrap-custom.css.scss の内容は以下のようにしました。

```
$body-bg:               #fff;
$text-color:            lighten(#000, 20%);

@import "bootstrap";
```

もっと高度なカスタマイズをするなら、
`cp $(bundle show bootstrap-sass)/vendor/assets/stylesheets/bootstrap.scss app/assets/stylesheets/bootstrap-custom.scss`
でコピーしたファイルを
`@import 'bootstrap';`
の代わりに
`@import 'bootstrap-custom';`
で使うことを想定しているようです。
