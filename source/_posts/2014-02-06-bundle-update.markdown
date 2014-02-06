---
layout: post
title: "bundle updateの記録"
date: 2014-02-06 14:29:27 +0900
comments: true
categories: ruby rails gem
---
今日は bootstrap の sass の gem を bootstrap 公式のものに移行したのが大きな変更点でした。

<!--more-->

## bootstrap-sass-rails から bootstrap-sass 3.1.0.2

[bootstrap 3.1.0 がリリース](http://blog.getbootstrap.com/2014/01/30/bootstrap-3-1-0-released/)されて少し時間が経ったので、
bootstrap-sass-rails を確認してみたところ、
[DEPRECATION NOTICE](https://github.com/yabawock/bootstrap-sass-rails#deprecation-notice)
に公式サポートされた sass の gem があるということで、
そちらに移行するように書かれていたので、移行しました。

bootstrap-sass gem への移行は UPGRADING に書かれているように `@import "twitter/bootstrap";` を `@import "bootstrap";` に書き換えるだけでした。
