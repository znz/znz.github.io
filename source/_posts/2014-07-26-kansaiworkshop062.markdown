---
layout: post
title: "62nd Ruby/Rails勉強会@関西に参加した"
date: 2014-07-26 13:27:09 +0900
comments: true
categories: event ruby rubykansai
---
[62nd Ruby/Rails勉強会@関西](http://rubykansai.doorkeeper.jp/events/13160 "62nd Ruby/Rails勉強会@関西")
に参加しました。

<!--more-->

## スクレイピングの話

後で資料は公開してくれるということで、メモはとらずに話をきいていましたが、
実際にどういうことに使っている話も多くて面白かったです。

資料は
[Rubyで作るクローラー Ruby crawler](http://www.slideshare.net/takurosasaki/ruby-crawler "Rubyで作るクローラー Ruby crawler")
で公開されています。

## Rails Girls Osaka の話

[Rails Girls Osaka 6-7th June 2014](http://railsgirls.com/osaka "Osaka 6-7th June 2014")
の話でした。
写真が多いので資料の公開はないということでした。

今後の予定としては、
[近日開催のイベント](http://railsgirls.jp/events/ "近日開催のイベント")
にはないですが、
[Upcoming Events](http://railsgirls.com/events "Upcoming Events")
には
[RubyHiroba 2014](http://rubyhiroba.org/2014/ "RubyHiroba 2014")
での
[Rails Girls Tokyo 4th](http://rubyhiroba.org/2014/rails-girls.html "Rails Girls Tokyo 4th")
も書いていました。

## Gemfile.local の話

redmine とかでも使っている方法で、
追記せずに何か良い方法はないのかという相談でした。

## るびま (Rubyist Magazine) の話

- http://magazine.rubyist.net/
- 誤植などの指摘は https://github.com/rubima/rubima-support へ。
- [Rubyist Magazine 十周年へのメッセージ](http://goo.gl/KpASY9) 募集中

## GitLab の Git Flow の話

<div style="float:right">
<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&amp;bc1=000000&amp;IS2=1&amp;bg1=FFFFFF&amp;fc1=000000&amp;lc1=0000FF&amp;t=znz-22&amp;o=9&amp;p=8&amp;l=as4&amp;m=amazon&amp;f=ifr&amp;ref=ss_til&amp;asins=477416366X" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</div>

- [GitLab Cookbook](https://gitlab.com/gitlab-org/cookbook-gitlab/blob/master/README.md "GitLab Cookbook") のベースを作った。
- <a href="http://www.amazon.co.jp/gp/product/477416366X/ref=as_li_ss_tl?ie=UTF8&amp;camp=247&amp;creative=7399&amp;creativeASIN=477416366X&amp;linkCode=as2&amp;tag=znz-22">GitHub実践入門 ~Pull Requestによる開発の変革 (WEB+DB PRESS plus)</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=znz-22&amp;l=as2&amp;o=9&amp;a=477416366X" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" /> はおすすめと言っていました。
- [git-flow cheatsheet](http://danielkummer.github.io/git-flow-cheatsheet/index.ja_JP.html "git-flow cheatsheet")
- [Understanding the GitHub Flow · GitHub Guides](https://guides.github.com/introduction/flow/index.html "Understanding the GitHub Flow · GitHub Guides")
- GitLab Flow の提案
- Git Flow について詳細に説明

GitLab Flow は git flow を知っている人向けに大雑把に説明すると

- release ブランチは使わない。
- develop ブランチの代わりに master ブランチを使う。
- master ブランチの代わりに stable ブランチを作る。

ということだと理解しました。

## Ruby 初級者向けレッスン 50回 ブロック

`block.call` の引数に複数渡すのがいいのか、配列でまとめて渡すのが良いのかという話はきいたことがなかったので、どういう話なのかちょっと気になりました。

多重代入について深入りすると大変そうなので、さらっと流したのはありだと思いました。

## @IT Rails4 入門記事をツッコもう

ここはメモを取っていました。

- [開発現場でちゃんと使えるRails 4入門（1）：簡単インストールから始める初心者のためのRuby on Railsチュートリアル (2/3) - ＠IT](http://www.atmarkit.co.jp/ait/articles/1402/28/news047_2.html "開発現場でちゃんと使えるRails 4入門（1）：簡単インストールから始める初心者のためのRuby on Railsチュートリアル (2/3) - ＠IT")
- rbenv 対 RVM
- エディタは Sublime Text, Vim, Emacs が多くて、その他 Atom, Eclipse
- `rails new` の時の `--skip-bundle` と `bundle install --path vendor/bundle` の話
- `bundle exec` の話
- [開発現場でちゃんと使えるRails 4入門（4）：現場で使えるか見極めたいRails 4.1の新機能8選 - ＠IT](http://www.atmarkit.co.jp/ait/articles/1405/16/news024.html "開発現場でちゃんと使えるRails 4入門（4）：現場で使えるか見極めたいRails 4.1の新機能8選 - ＠IT")
