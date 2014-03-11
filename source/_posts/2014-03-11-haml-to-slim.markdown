---
layout: post
title: "hamlからslimへの移行でひっかかったこと"
date: 2014-03-11 22:39:11 +0900
comments: true
categories: ruby rails haml slim zsh
---
haml より slim の方が速いらしいという話を良く見かけるので、
haml から少しずつ slim に移行しようとして、
haml2slim で変換してみたらひっかかりまくりました。

<!--more-->

## インデントがない

haml は生成される HTML がインデントされていてみやすいのですが、
slim は不要なスペースは入れずにコンパクトな HTML が生成されるようで、
単純に比較できなくて困りましたが、
https://github.com/slim-template/slim#default-options
で設定できるようだったので、以下のように設定しました。

```ruby config/initializers/slim.rb
if Rails.env.development?
  Slim::Engine.set_default_options pretty: true
end
```

## 1.9 hash style

https://github.com/slim-template/haml2slim/pull/23
に pull request がでているように
haml で属性を `{foo: bar}` のように書く形式には対応していないようです。

## string interpolation がおかしい

https://github.com/slim-template/haml2slim/issues/7
などで報告されているように `#{ ... }` という書き方の変換で問題があるようです。
`%tag(foo="#{bar}")` が `tag(foo="# bar ")` になっていて気づきました。

## HTML コメントの中でタグにならない

```haml
    /[if lt IE 9]
      %script{src: url_for(html5shiv)}
```

のように書いていたのが

```text
    /![if lt IE 9]
      script src: url_for(html5shiv)
```

のように変換されてさっぱりダメだったので、

```text
    /[if lt IE 9]
      script(src="#{url_for(html5shiv)}")
```

または

```text
    /[if lt IE 9]
      script(src=url_for(html5shiv))
```

のように書き換えました。
HTML コメント `/!` の中だとタグが使えない (そのまま文字列としてコメントの中に入る) ようなので、
IE コンディショナルコメント `/[...]` を使う必要があるようです。

## エラーメッセージがわかりにくい

書き換え途中で、属性値をくくる `""` を忘れると
エラーメッセージが
`syntax error, unexpected ';'`
としかでないので原因がわかりにくくて困りました。

他のところでも ruby のエラーメッセージそのままなので、
閉じ忘れ系は原因の行と離れたところでエラーになって、
行番号もあまり当てにならなくて不便です。

## 真偽値属性の属性値が省略できない

haml だと `%div(data-pjax-container)` で `<div data-pjax-container>` に変換できるのに、
slim だと `div(data-pjax-container)` が `<div data-pjax-container="">` になってしまって、
`=""` が余計です。

[Render boolean attributes without value in html5](https://github.com/slim-template/slim/issues/480)
によると
`:format` を `:html` にすれば良いということで

```ruby config/initializers/slim.rb
Slim::Engine.set_default_options format: :html
```

と設定してみても slim 2.0.2 だと変化がありませんでした。

まだリリースされていないバージョンを使う必要があるようで、
`Gemfile` で `gem 'slim', github: 'slim-template/slim'` と指定して
試そうとしたのですが、
rails 3.2.17 との組み合わせは tilt の要求バージョンの解決が
出来なくて使えませんでした。

実験のため slim 専用の Gemfile を用意して以下のように試したところ、
slim と temple の両方をまだリリースされていないバージョンにすると
使えることが確認できました。

```console
% mkdir /tmp/slim-test
% cd /tmp/slim-test
% bundle init
% echo 'gem "slim", github: "slim-template/slim"' >> Gemfile
% echo 'gem "temple", github: "judofyr/temple"' >> Gemfile
% bundle
% echo 'div(ng-app)' | bundle exec slimrb
<div ng-app=""></div>
% echo 'div(ng-app)' | bundle exec slimrb -o format=:html
<div ng-app></div>
```

今のところ、 `rack-pjax` の使用は止めていて、
 `=""` がつく挙動で直接困ることはなさそうなので、
この問題は保留することにしました。

## javascript や css の埋め込み

```haml
    :javascript
      window.current_user_id = #{current_user.id.to_i}
```

のように書いていたら、

```text
    javascript:
      | window.current_user_id = #{current_user.id.to_i}
```

と変換されて `| ` がそのまま `script` の中身に付いてしまったので、
削除しました。

```text
    javascript:
      window.current_user_id = #{current_user.id.to_i}
```

`:css` から `css:` の変換でも同様でした。
CSS の場合は行頭が記号だと `| ` がついていないこともありました。

## 空白の有無の問題

```haml
            = link_to home_path, class: 'brand' do
              = image_tag("brand.png", alt: "", size: "35x35")
              = base_title
```

がそのまま slim になっていて、基本的には問題なかったのですが、
haml と違って slim だとタグとタイトルの間に空白が入らなくなってしまって、
見た目が変わってしまっていたので、
`'` だけの行を追加して空白が入るようにしました。

```text
            = link_to home_path, class: 'brand' do
              = image_tag("brand.png", alt: "", size: "35x35")
              '
              = base_title
```

## end が抜けるバグで SyntaxError

`syntax error, unexpected keyword_ensure, expecting keyword_end`
になるので原因を調べてみたところ、
インデントの中の最初がコメントだと、
そのブロックの `end` が抜けてしまうようです。

```console
% echo $'- if true\n  / comment' | bundle exec slimrb -e
<% if true

 %>
% echo $'- if true\n  |' | bundle exec slimrb -e
<% if true

end %>
% echo $'- articles.each do |a|\n  / comment' | bundle exec slimrb -e
<% articles.each do |a|

 %>
% echo $'- articles.each do |a|\n  | text' | bundle exec slimrb -e
<% articles.each do |a|
 %>text<%
%><% end %>
% echo $'- articles.each do |a|\n  / comment\n/ more comment' | bundle exec slimrb -e
<% articles.each do |a|


 %>
% echo $'- articles.each do |a|\n  / comment\n| text\n/ more comment' | bundle exec slimrb -e
<% articles.each do |a|

 %>text<%
%><%
%>
% echo $'- articles.each do |a|\n  / comment\n| text\n' | bundle exec slimrb -e
<% articles.each do |a|

 %>text<%
%>
```

この件は https://github.com/slim-template/slim/issues/485 で報告してみました。

## ファイル名の指定が面倒

これは slim や haml2slim の問題ではないのですが、
zsh で久しぶりに compctl を使って

    compctl -g '*.haml' + -g '*(-/)' haml2slim

と設定しましたが、ちゃんと compsys を使った方が良いと思い直して、

    compdef '_files -g "*.haml"' haml2slim

と設定しました。

## 感想

シンプルとか入力が楽とかいう主観的なものは個人的には同意できなくて利点とは思えないので、
速いという客観的な理由で slim への移行をしているのですが、
haml2slim の出来が悪かったり、
空白の扱いの違いで同じようなソースからでも最終的な見た目が違うものが出来たりして、
移行は思ったより大変な印象を受けました。

はまりどころさえわかっていれば、そこを注意して haml2slim での変換結果を修正していけば良いので、途中からはどんどん変換していけるようになりました。
(まだ一部しか終わっていませんが。)

それなりの数の変換をしてみた結果、最初から slim で書くようにすれば haml と大きな違いはなさそうにも感じました。

`end` 抜けなどの問題が起きたことから、
haml と比べていろんな書き方で使われていなさそうということで、
まだ安定してなさそうと感じました。

また属性値なしの `<div ng-app>` のような出力に対応しているリリースがまだないなど、
融通が利かない点があって問題が起きそうな可能性がありそうだと感じました。
