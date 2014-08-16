---
layout: post
title: "octopress をアップデートした"
date: 2014-08-16 22:26:34 +0900
comments: true
categories: octopress
---
[Updating Octopress](http://octopress.org/docs/updating/ "Updating Octopress") を参考にして octopress をアップデートしてみました。

<!--more-->

## 更新をマージ

`git pull octopress master` でマージしました。
conflict したらがんばって修正してから `git commit` します。

久しぶりに更新すると `_config.yml` がひっかかりやすいと思いますが、変更は少ないのでマージしやすいと思います。

## gem の更新

`bundle install` か `bundle update` で更新します。

`Gemfile.lock` が `.gitignore` に入って octopress master からは消されてしまったので、
`bundle update` で動かなくなったときに戻せるように、
`git add -f Gemfile.lock` で自分のレポジトリでは追加しておくのも良いかもしれません。

## テンプレートのソースの更新

`rake update_source` で更新します。
カスタマイズしていると戻ってしまうので、がんばって再適用します。

zenback 対応の追加などの `_config.yml` の `default_asides` などでは対応しきれないカスタマイズが戻ってしまうので、
変更し直しました。

## テンプレートのスタイルの更新

`rake update_style` は特に何も変化がありませんでした。

## `*.old` の削除

テンプレートの更新でバックアップとして作成される `source.old/` と `sass.old/` は更新後には不要なので削除します。

## Build Warning の対処

更新後に

     Build Warning: Layout 'nil' requested in blog/categories/octopress/atom.xml does not exist.

のような警告が出るようになってそのうち対処されると思っていたら、
[How to fix Octopress build warning Layout nil requested - Bravo Kernel](http://www.bravo-kernel.com/blog/2014/08/how-to-fix-octopress-build-warning-layout-nil-requested/ "How to fix Octopress build warning Layout nil requested - Bravo Kernel")
に書いてあるように

     layout: nil

を

     layout: null

にするという修正で対応が入ったのですが、
`source/_includes/custom/category_feed.xml`
は `source/_includes/custom/` 以下なので自分で修正する必要がありました。
