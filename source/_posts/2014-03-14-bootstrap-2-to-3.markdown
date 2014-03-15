---
layout: post
title: "bootstrap 2.3.2 から bootstrap 3.1.1 への移行"
date: 2014-03-14 23:00:00 +0900
comments: true
categories: ruby rails bootstrap
---
Rails 3.2.17 を使っている rails アプリで
bootstrap 2.3.2 から bootstrap 3.1.1 に移行している途中なのですが、
どういうところに対処が必要だったのか、
メモをまとめてみました。

<!--more-->

## Gemfile 変更

bootstrap 2 系のときは twitter-bootstrap-rails gem を使っていたのですが、
bootstrap 3 系対応がリリースされていないこともあって、
bootstrap 側から公式にリリースされている bootstrap-sass gem に移行しました。

bootstrap-sass は rails_admin gem で使われていて、
バージョンが rails_admin によって制限されてしまうので、
公式の less の gem があれば良かったのですが、
見つけられなかったので、
自分しか使っていない rails_admin を rails 4 に上げるまでの間、
一時的に rails_admin を外して、
新しい bootstrap-sass を rails 3.2.17 と一緒に使いました。

```ruby Gemfile
gem 'less-rails'
gem 'twitter-bootstrap-rails'
```

を https://github.com/twbs/bootstrap-sass に書いてあるバージョン指定付きで

```ruby Gemfile
gem 'sass-rails', '>= 3.2'
gem 'bootstrap-sass', '~> 3.1.1'
```

に書き換えて、 `bundle update` で反映しました。

## twitter-bootstrap-rails のヘルパーメソッド削除

twitter-bootstrap-rails で定義されているヘルパーメソッドを使っていたら、
削除するなり代替メソッドを定義して置き換えるなり、
一時的にコメントアウトするなりしてエラーにならないようにしておきます。

今回は `glyph` と `add_breadcrumb` がひっかかりました。

## bootstrap_and_overrides.css.less 削除

app/assets/stylesheets/bootstrap_and_overrides.css.less を削除して、
app/assets/stylesheets/app.css.scss のようなファイルに移行しました。

```css app.css.scss
@import "bootstrap";

// その他の bootstrap_and_overrides.css.less から移行した内容
```

## キャッシュ削除

less を消したはずなのにエラーにならなくておかしいと思っていたら、
キャッシュの影響だったので、
development 環境でもキャッシュを有効にしている場合は削除します。

`bundle exec rake tmp:clear` で消したり、
`rake` 経由だと遅いと思ったら
`rm -rf tmp/cache` のようにばっさり削除したりすると良いと思います。

## .navbar

bootstrap 2 を使っていた頃は
`.navbar` と `.nav` の違いがよくわかっていなかったのですが、
`.navbar` は上 (または下) のバーのことで、
`.nav` はその中にある `ul` のリンクのことでした。

`.nav` は他にも `.nav-tabs` のような使い方もするというのを先に知っていれば
迷わなかったと思います。

http://getbootstrap.com/components/#nav
(古い方: http://getbootstrap.com/2.3.2/components.html#navs )
の説明では navbar の上なのですが、
サイトの作成順として画面の上から作っていっていたので、
迷ってしまっていたようです。

`.navbar` の中の `.nav` は `.navbar-nav` に変わっている、
`.navbar-inner` は `.container` か `.container-fluid` に置き換えなど、
変更点が多いので、
`.navbar` の中身は全面的に見直すのが良さそうです。

## .brand から .navbar-brand

`.brand` から `.navbar-brand` に変わっていました。

中に画像を入れているとずれてしまうので、
scss に以下のように書いて調整しました。

```css
.navbar-brand {
  padding-top: 8px;
  img {
    height: 34px;
    width: 34px;
  }
}
```

8 + 34 + 8 ということで高さ 50px の `.navbar` の真ん中になることを期待しています。

## .container (固定幅) か .container-fluid (横幅いっぱい)

grid system の配置を使うなら `.row` は `.container` か `.container-fluid` の中に置く必要があります。

違いは実際に window の横幅を変えてみればわかるのですが、
`.container` だと
http://getbootstrap.com/css/#grid
のように 768px 992px 1200px のところで急に配置が変わって、
その間は左右の余白が増減するだけのようです。

`.container-fluid` だと普通のサイトと同じように出来るだけ横幅いっぱいになるように中身の横幅が変わります。

bootstrap 2 ではレスポンシブにしたいときは `.container-fluid` で固定幅の時は `.container` だったので、
`.container-fluid` しか使っていなかったのですが、
今はレスポンシブな中で使い分けが出来るようになっているようです。

http://getbootstrap.com/examples/navbar-fixed-top/
の例で `Project name` が左端によっていないのも `.container` を使っているからです。
ブラウザーのデベロッパーツールで `.container-fluid` に変えて横幅を変えてみれば
違いがわかると思います。

## `.row-fluid .span*` から `.row .col-xs-*`

`.row` と `.row-fluid` の区別はなくなって、
`.row` に統一されています。

`.span*` の置き換えは `.col-md-*` と説明されていることが多いようですが、
`.col-md-` が `@media (min-width: 992px)` になっているなど、
小さい画面サイズ用のスタイルを大きい画面用のスタイルで上書きするようになっているので、
画面サイズに関わらず同じ分割をしたいのなら `.col-xs-` だけ指定すれば良さそうです。

## btn

`btn` だけでは色や枠 (border) が変わらなくなったので、
`btn-default` を足す必要がありました。

サイズ変更の `btn-mini` などがなくなって `btn-xs` などに変更する必要がありました。

## list-unstyled

`ul.unstyled` は `ul.list-unstyled` に変更する必要がありました。

## font-awesome-sass

アイコンは glyphicons ではなく font-awesome を使っていたので、
[font-awesome-sass](https://github.com/FortAwesome/font-awesome-sass)
に乗り換えました。

mixin や variables などがなければ関係なさそうですが、
[bootstrap-sass の Usage](https://github.com/twbs/bootstrap-sass#usage)
では `//= require` より `@import` が推奨されていたので、

```css
@import "bootstrap";
@import "font-awesome";
```

のように `@import` にしてみました。

`<i class="icon-sort"></i>` が
`<i class="fa fa-sort"></i>` に変わるような感じで、
`icon-` を `fa-` に置き換えるだけではなく、
`fa ` の追加が必要でした。

`glyph` ヘルパーメソッドを使っていた場合は、
`icon` ヘルパーメソッドに書き換えると良いのですが、
複数引数の扱いが違うのと、
`_` を `-` に置き換える処理がなくなっているのに注意が必要です。

たとえば `glyph(:sort_up)` は `icon('sort-up')` のように書き換えました。

`glyph(:lock, :white)` のような複数指定には対応していないようなので、

```haml
    = icon('spinner fa-spin'.freeze)
    -# or
    = icon(:spinner, nil, class: 'fa-spin')
```

のように 2 個目以降は無理矢理埋め込むしかなさそうです。

それぞれのヘルパーメソッドのソースは以下のようになっています。

- [glyph](https://github.com/seyhunak/twitter-bootstrap-rails/blob/40afc477f6a3813ef82cf5821602c9cf2422efc2/app/helpers/glyph_helper.rb)
- [icon](https://github.com/FortAwesome/font-awesome-sass/blob/23ca05e75e85ec84afecca9f62e7f01d1fb9628b/lib/font_awesome/sass/rails/helpers.rb)

`icon` は第2引数に文字列を指定すると間にスペースを入れてくれるので、
今までは

```css
i[class^="icon-"]:after {
  content: " "
}
```

のようにスタイルシートで空白を足していたのですが、

```ruby
icon(:pencil, t('.edit', default: :'helpers.links.edit'))
```

のように使うのも良さそうです。

今回は書き換えの手間を減らすため、以下のように今までと同じようにスタイルシートで空白を入れるようにしました。
何のスタイルが影響しているのか調べていないのですが、半角スペースだとうまくいかなかったので、 NBSP を直接埋め込みました。

```css
i.fa:after {
  content: " "; // nbsp
}
```

いくつかのアイコンの名前は変更が必要でした。
使っていて影響があったのは以下のアイコンでした。

- 変更前 変更後
- comment-alt comment-o
- folder_close folder
- picture picture-o
- remove times
- signin sign-in
- signout sign-out
- star-empty star-o
- trash trash-o

## pagination

kaminari を使っているので、
`app/views/kaminari/_paginator.html.*`
の
`nav.pagination ul`
という構造になっている部分を
`ul.pagination`
に書き換えるだけでした。

## label

`btn` と同様に `label` 単独ではなく `label-default` と組み合わせるようになりました。

選択されているものに `label-inverse` を使っていたのになくなってしまったので、
`label-primary` に変えました。

意味的には `.nav` に変えた方が良さそうなので、
一通り一時的な対処をした後にちゃんと変更しようと思っています。

## muted

`.muted` は `.text-muted` に変更しました。

## 参考サイト: Bootstrap3移行ガイド

基本的に本家のドキュメントを参照していたのですが、
ここまで変更してから 2 から 3 への移行のドキュメントを探して、
[Bootstrap3移行ガイド](http://bootstrap.s1.adexd.net/)
をみつけて見てみたのですが、全体としては非常に参考になるのですが、
所々気になる点がありました。
連絡先が見つけられなかったので、ここにメモしておきます。

[グリッドシステム（Grid system）](http://bootstrap.s1.adexd.net/css.php#grid)
の【Bootstrap2.xとの変更箇所】の説明で `.row` と使い分けると書いているのは
間違いだと思います。

スクリーンリーダー用のクラスを
[すべての要素の表示を隠す](http://bootstrap.s1.adexd.net/css.php#screen-reader)
と説明しているなど、たまに間違いがあるようです。

## 現状まとめ

layouts といくつかのコントローラーに対応する views を変更すれば
ある程度自分のアプリで使っているものは網羅できるので、
そこからは移行に必要な変更点はわかってきて速くなっていく感じでした。

先日書いた haml から slim への移行も同時にやっていて、
views 全体を見直す良い機会になっています。

一部で jquery-ui を使っていて、
機能的には問題ないのですが、
見た目が bootstrap 3 となじまないので、
後で対処が必要そうでした。
