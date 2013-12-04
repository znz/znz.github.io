---
layout: post
title: "Rails 4.0.2のi18nで出るようになったdeprecated警告の対策方法"
date: 2013-12-04 10:59
comments: true
categories: ruby rails
---
Ruby on Rails のセキュリティアップデートがあって、
4.0.2 にあげたら i18n gem も 0.6.5 から 0.6.9 にあがって
`locale` の設定変更をしているところで
`[deprecated] I18n.enforce_available_locales will default to true in the future. If you really want to skip validation of your locale you can set I18n.enforce_available_locales = false to avoid this message.`
という警告が出るようになりました。

<!--more-->

## 警告の出るタイミング

[deprecated - Rails 4.0.2 I18n validation deprecation warning - Stack Overflow](http://stackoverflow.com/questions/20361428/rails-4-0-2-i18n-validation-deprecation-warning)
経由で
[Add I18n.locale_available? and enforce available locales](https://github.com/svenfuchs/i18n/commit/3b6e56e06fd70f6e4507996b017238505e66608c9)
のコミットから入った変更ということで、
コミットログをみてみると、

- `I18n.config.default_locale=`
- `I18n.config.locale=`
- `I18n.translate`
- `I18n.localize`
- `I18n.transliterate`

を呼んだときに影響するようです。
つまり `rails new` で作っただけだと `config/application.rb` の
`config.i18n.default_locale = :de` がコメントアウトされていて、
警告は出ません。

## 日本語のみで使う場合

日本語のみで使うのなら、
将来のデフォルトの
`I18n.enforce_available_locales = true`
にしてしまってから、
普通に日本語をデフォルトにする
`config.i18n.default_locale = :ja`
を呼べば良いと思います。

```ruby config/application.rb
    I18n.enforce_available_locales = true
    config.i18n.default_locale = :ja
```

ただし、先にちゃんと日本語の locale ファイルを作っておかないと
`I18n::InvalidLocale`
という例外が発生して、
`rake` などで
`:ja is not a valid locale`
と言われてしまいます。

## 今まで通りの挙動にする場合

`I18n.enforce_available_locales = false`
にすれば今まで通りの挙動になり、
存在しない `locale` を設定しても例外は発生しません。

## 今のデフォルト

今は
`I18n.enforce_available_locales = nil`
がデフォルトになっていて、
`nil` だと警告がでる、
ということのようです。
