---
layout: post
title: "Rails 4.1 で jQuery Raty を使ってみた"
date: 2014-08-11 19:37:53 +0900
comments: true
categories: rails jquery
---
Rails で星を使った評価付けを使いたかったので、
jQuery プラグインを探してみたところ、
[jQuery Raty](http://wbotelhos.com/raty "jQuery Raty")
というのが良さそうだったので使ってみました。

<!--more-->

## 対象バージョン

- jQuery Raty v2.7.0
- Ruby on Rails 4.1.4
- Ruby 2.1.2

## うまくいかなかった方法

[RubyGems.org](http://rubygems.org/ "RubyGems.org")
の方には古いバージョンしかなさそうだったので、
[Rails Assets](https://rails-assets.org/ "Rails Assets")
で最新バージョンを使おうと思い、
`Gemfile` に以下の設定をしたのですが、
`raty` のディレクトリ配置が特殊なのか、
うまくいきませんでした。

```ruby Gemfile
source 'https://rails-assets.org'
gem 'rails-assets-raty'
```

### development 環境では動いた方法

以下のように書くことで development 環境では動いたのですが、
capistrano で deploy した先では画像が表示されていませんでした。

```css app/assets/stylesheets/raty.css.scss
//= require raty/lib/jquery.raty
```

```text app/assets/javascripts/raty.js.coffee.erb
#= require raty/lib/jquery.raty
 $ ->
   $('.raty').raty
     cancel   : true
    cancelOff: '<%= image_path('raty/lib/images/cancel-off.png') %>'
    cancelOn : '<%= image_path('raty/lib/images/cancel-on.png') %>'
    starHalf : '<%= image_path('raty/lib/images/star-half.png') %>'
    starOff  : '<%= image_path('raty/lib/images/star-off.png') %>'
    starOn   : '<%= image_path('raty/lib/images/star-on.png') %>'
    click: (score, event) ->
      raty = $(event.target).parent()
      $(raty.data('field')).val(score)
    score: ->
      $($(this).data('field')).val()
```

## うまくいった方法

`vendor/assets/stylesheets/jquery.raty.css` と
`vendor/assets/javascripts/jquery.raty.js` に
ダウンロードしたファイルをおいて、
`assets` も以下のように書き換えました。
画像も `vendor/assets/images/raty` においてもうまくいかなかったので、
`vendor/assets/images/raty` におきました。

Web フォントは今回は使っていないので、
考慮していません。
`starType` を `i` に変更しない限り使われないはずなので、
Web フォントを配置しなくても問題ないと思います。

```css app/assets/stylesheets/raty.css.scss
//= require jquery.raty
```

```text app/assets/javascripts/raty.js.coffee.erb
#= require jquery.raty
 $ ->
   $('.raty').raty
     cancel   : true
    cancelOff: '<%= image_path('raty/cancel-off.png') %>'
    cancelOn : '<%= image_path('raty/cancel-on.png') %>'
    starHalf : '<%= image_path('raty/star-half.png') %>'
    starOff  : '<%= image_path('raty/star-off.png') %>'
    starOn   : '<%= image_path('raty/star-on.png') %>'
    click: (score, event) ->
      raty = $(event.target).parent()
      $(raty.data('field')).val(score)
    score: ->
      $($(this).data('field')).val()
```

## フォームでの使用例

bootstrap 3.2.0 の `form-horizontal` を使っているので、
slim で以下のように使っています。

```text _form.html.slim
    .form-group
      = f.label :readability, class: "control-label col-md-2"
      .col-md-10
        .raty.form-control data-field='#book_report_readability'
        = f.hidden_field :readability, class: "form-control"
```

## show での使用例

以下のように `raty` の星を使って表示しています。

```ruby app/helpers/raty_helper.rb
module RatyHelper
  def raty_stars(n, max=5)
    (
      image_tag('raty/star-on.png', alt: '') * n +
      image_tag('raty/star-off.png', alt: '') * (max-n)
      ).html_safe
  end
end
```
