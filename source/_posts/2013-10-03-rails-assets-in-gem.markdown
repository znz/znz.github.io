---
layout: post
title: "rails で assets を gem に入れる時の配置"
date: 2013-10-03 10:30
comments: true
categories: ruby rails gem
---
昨日の記事で
[jquery_mobile_rails](https://rubygems.org/gems/jquery_mobile_rails) 1.3.2
で画像ファイルが
`rake assets:precompile`
で処理されないという話を書きましたが、
[jquery-ui-rails](http://rubygems.org/gems/jquery-ui-rails)
gem
などは Gemfile に足すだけで特に require などをしなくても
画像が処理されていたので違いを調べてみました。

<!--more-->

結論を先に書くと、
`jquery-ui-rails`
は
`app/assets/images/`
に画像ファイルを置いていたから処理されていて、
`jquery_mobile_rails`
は
`vendor/assets/images/`
に画像ファイルを置いていたから、
というのが原因でした。

自作の gem で
`app/assets/images/`
と
`vendor/assets/images/`
に画像を置いて rails 4.0.0 の
`rake assets:precompile`
で違いがあることを確認しています。

`jquery_mobile_rails` の issues を確認すると
[In production, path to images is wrong](https://github.com/tscolari/jquery-mobile-rails/issues/16)
という同じ問題に困っている話があったので、
コメントを付けておきました。

ちなみに
いくつか存在する jQuery Mobile の assets の中から
`jquery_mobile_rails`
を選んだ理由は新しいバージョンへの対応が一番早そうにみえたからです。
