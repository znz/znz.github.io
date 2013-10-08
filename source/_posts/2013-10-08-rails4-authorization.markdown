---
layout: post
title: "rails 4 対応の認可の gem"
date: 2013-10-08 23:04
comments: true
categories: ruby rails authorization pundit
---
rails 4 に対応している認可のライブラリとして
`cancan`
からの移行先候補を選ぶために
`the_role`
と
`pundit`
という gem を試してみました。

<!--more-->

## cancan からの移行

認証は rails 4 でも
`devise`
を使い続ければ良さそうな感じなのですが、
認可の方は
`cancan`
の rails 4 対応がいまいちで
[Ready for Rails 4?](http://ready4rails4.net/gems/cancan)
でも *not ready* のままで
[issues](https://github.com/ryanb/cancan/issues)
も溜まっていることもあり、
rails 4 では他の gem への移行を検討していました。

## 移行先候補

移行先候補として、まず最初に試したのは
[the_role](https://github.com/the-teacher/the_role)
という gem でした。
後述しますが、これは最初から組み込むなら良さそうなのですが、
今回はちょっと目的にあわなかったので諦めました。

次に試したのは
[pundit](https://github.com/elabs/pundit)
という gem で、
これは
[Simple authorization in Ruby on Rails apps — Elabs](http://www.elabs.se/blog/52-simple-authorization-in-ruby-on-rails-apps)
という blog の説明で実装のほぼすべてが理解できるぐらいシンプルな gem でした。
rails 自体の変化にも強そうだと思って、
今回はこれを採用しました。

その後、
検索用キーワードとして使えそうな名前が増えたことで
[User Authorization (Ruby) on pluginGeek](http://www.plugingeek.com/categories/user-authorization-ruby)
という一覧を見つけました。

後で知ったので今回は試していないのですが、
[Authority](https://github.com/nathanl/authority)
は高機能そうなので、高機能なものがほしい場合や
`cancan`
で複雑なことをしていた場合の移行先としては良さそうです。

## `the_role`

最初に試した
[the_role](https://github.com/the-teacher/the_role) 2.1.1
は
「TheRole - Authorization Gem for Ruby on Rails with administrative interface」
と書いてある通り、設定画面も付いているのが特徴です。

認可の設定画面は
`localhost:3000/admin/roles`
で出てくるのですが、
layout に

```haml
= yield :role_sidebar
= yield :role_main
```

を入れていないと真っ白なページで何も内容が出てこないので注意が必要です。
`assets`
もちゃんと設定していないと
`Enable`, `Disable` のところが動かないなど、
使い始めにはまりどころがいくつかありました。

他にも

* 管理画面が `admin/roles` 決めうち (gem の中の `config/routes.rb`)
* `User` というモデル名も決めうち (`Admin::User` などは使えない)
* 1 ユーザーは 1 ロールのみ (例えば、あるユーザーにカレンダーの管理者とお知らせの管理者のロールをつけるということは出来ないので複合権限のロールを別途作る必要がある)

という点など柔軟性はなさそうな感じでした。
このうち、管理画面のパスが決めうちという点が
`rails_admin`
との組み合わせで問題になったので、
今回は採用を見送りました。
`rails_admin`
の方を `admin` 以外のパスに変更して良いのなら
一緒に使えると思います。

使うのに必要な設定は
`rake rails:template LOCATION=/path/to/file.rb`
のように使うことを想定して作っているアプリケーションテンプレートの
https://github.com/znz/rails-app-template/blob/master/the_role.rb
も参考になると思います。

## pundit

こちらも初期設定などの一部は
https://github.com/znz/rails-app-template/blob/master/pundit.rb
で出来るようにアプリケーションテンプレートを作成中です。

認可の仕組みとしては簡単に言うと
`Post` の `update` の認可なら
`PostPolicy.new(current_user, @post).update?`
で判断出来るような `Policy` クラスを用意しましょう、というだけです。

`index` などのようなアクション用として
`@posts = PostPolicy::Scope.new(current_user, Post.scoped).resolve`
のような `Scope` も用意する、ということになっています。

そういうものを用意した上で便利に使えるようにいくつかのヘルパーメソッドなどを
用意したものが `pundit` gem になっています。

つまり仕組み自体は `pundit` gem なしでも使えるので、
細かく制御したい部分では `pundit` gem に関係なく
`Policy` や `Scope` を直接扱うこともできます。

## pundit と rails_admin

というわけで直接扱ったり、
モデルとは関連ない `Policy` を用意したりして、
`rails_admin`
との連携を作成している途中のものを
https://github.com/znz/rails-app-template/blob/master/pundit-with-rails_admin.rb
に公開しています。

## pundit と scaffold

`cancan`
の
`load_and_authorize_resource`
のようなものは
`pundit`
自体には用意されていないので、
scaffold
で生成したコントローラーに埋め込む処理を
https://github.com/znz/rails-app-template/blob/master/pundit-with-scaffold.rb
で公開しています。

## まとめ

今回は初期設定をアプリケーションテンプレートにまとめつつ、
認可用の gem を試しました。

アプリケーションテンプレートはどれもひな形としての利用を想定しているので、
この後さらに変更していくことになると思います。

アプリケーションテンプレート自体も使っていった上で必要に応じて変更していく予定なので、
リンク先はこの記事を書いた時点の内容とは合わなくなっている可能性もあります。

認可用のライブラリとしては権限管理画面付きのものを簡単に組み込みたければ
`the_role`、
シンプルな部品を組み合わせるのが良ければ `pundit`、
高機能なものが欲しければ `authority` を試すのが
良さそうだと思いました。
