---
layout: post
title: "mastodonにpull requestを送った話と開発環境構築の話"
date: 2017-04-16 22:43:41 +0900
comments: true
categories: mastodon rails vagrant
---
最近流行っている mastodon に pull request を送ったので、
その環境構築などの話です。

<!--more-->

## 環境

- macOS Sierra 10.12.4
- VirtualBox 5.1.18
- Vagrant 1.9.3
- vagrant-hostsupdater 1.0.2
- https://github.com/tootsuite/mastodon の master

## 起動まで

[Development with Vagrant](https://github.com/tootsuite/mastodon#development-with-vagrant) からリンクされている
[Vagrant guide](https://github.com/tootsuite/documentation/blob/master/Running-Mastodon/Vagrant-guide.md) を参考にして、
環境を構築しました。

初回起動 (`vagrant up`) 時は `vagrant-hostsupdater` での `sudo` に続けて実行されるので気づかなかったのですが、
2 回目に `vagrant up` した時に `/etc/exports` の変更のためにも `sudo` が実行されているのに気づきました。

    git clone https://github.com/tootsuite/mastodon
    cd mastodon
    vagrant plugin install vagrant-hostsupdater
    vagrant up

## 初期アカウント設定

初期アカウントはメールアドレスが `admin@mastodon.dev` でパスワードが `mastodonadmin` と書いてあるのですが、入れなかったので、

    cd /vagrant
	rails c
	User.all

で確認してみると、メールアドレスが `admin@localhost:3000` になっていました。
そこで、そのまま `rails c` の中で、

    u=User.first
	u.email="admin@mastodon.dev"
	u.save!

で修正しました。

こんな感じで何か引っかかった時は rails の知識がないと辛そうです。

## いろいろ動作確認

80 番ポートから 3000 番ポートへのポートフォワーディングは
Vagrantfile で設定しているので、
ホスト側から `http://mastodon.dev` は見えるのですが、
`vagrant ssh` で入ったゲスト側では `curl http://mastodon.dev` ではなく
`curl http://localhost:3000` や `curl http://mastodon.dev:3000` などのように
ポート番号をつける必要がありました。

## ストリーミング API

Vagrantfile を見ればわかるのですが、
rails server が `rails s -d -b 0.0.0.0` で動いているところにポートフォワーディングしているだけなので、
streaming API は使えませんでした。

最初、実装されていないのかと勘違いしてしまったのですが、
実装されていると聞いたので、よくみてみると
`streaming/index.js` で rails 外のところに実装されていました。

`npm run start` で起動すればゲストの中なら 4000 番ポートで使えるようになったので、
ストリーミング API を使いたい場合は一工夫必要そうです。

## メール

メールは `http://mastodon.dev/letter_opener` に溜まっていました。
mailcatcher と違って、再起動しても残っていました。

## 環境の更新

最新の状態にするために master の変更に追随する必要がありますが、
`vagrant ssh` で入った中で `cd /vagrant` した状態で `git pull` するとパーミッションの関係でうまくいかないようだったので、
ホスト側で `git pull` する方が安全なようです。

`yarn install` などでファイルが書き換わっていると、その変更を元に戻しておく必要もあるかもしれません。

`git pull` した後は、Gemfile なども書き換わっていた時は

    vagrant ssh
    cd /vagrant
    bundle install
    yarn install
    rails db:migrate
    rails assets:precompile
    pkill -f puma
    export $(cat ".env.vagrant" | xargs)
    rails s -d -b 0.0.0.0

のような感じで `bundle install` と起動している rails server の停止と Vagrantfile の `$start` の処理をすると良さそうです。

パーミッションの問題で `yarn install` がうまくいかなかった時は `node_modules` を削除すると良さそうです。
ゲスト側だとうまくいかなかったら、ホスト側で消すなどの工夫が必要そうです。

## pull requests

`vagrant up` する前に Vagrantfile を確認していたところ、
`PATH` にカレントディレクトリを追加していたので、
[Remove current directory from PATH](https://github.com/tootsuite/mastodon/pull/1779)
で削除する pull request を送りました。

そして、管理画面をみていたところ、title が並んでいてなんだこれ、と思ったので、
[翻訳の更新の pull request](https://github.com/tootsuite/mastodon/pull/1785)
を送りました。

`I18n.t` の最後の単語は翻訳がないときのデフォルトとしても使われるので、
そのことも考慮した単語を選んだ方が良さそうに思いましたが、
yaml ファイルをみていると title にしたい気持ちもわからなくはなかったので、
悩ましいところです。

## 翻訳もれ?

テスト環境なので短いパスワードでもいいかと思って、テストアカウントを登録する時に短いパスワードを入れてみたところ、
`translation missing: ja.activerecord.errors.models.user.attributes.password.too_short`
と出てきて調べてみると `config/locales/doorkeeper.fr.yml` にだけ `too_short` の翻訳があって何かおかしいと思って、
よく調べてみると、
rails-i18n gem に翻訳が入っている、 rails デフォルトのエラーメッセージだとわかったので、
[rails-i18n gem の追加リクエスト](https://github.com/tootsuite/mastodon/issues/1790) を出しました。

## i18n-tasks

前回の翻訳の更新は目視で比較して追加したのですが、
rails-i18n gem について調べているときに
i18n-tasks gem というのが入っていると気づいたので、
次はそれを使ってみました。

`i18n-tasks health` でチェックできるのですが、デフォルトだと全言語が対象で、出過ぎなので、
`i18n-tasks health -l ja` で日本語だけに絞って表示しました。

そして `i18n-tasks add-missing -l ja` で `config/locales/ja.yml` に英語のまま追加され、
`git diff` で何が追加されたか確認して翻訳していきました。

`i18n-tasks find '*.reset_password'` や `i18n-tasks find admin.accounts.reset_password` のようにして、どこで使われているのか確認して、
実際に表示させて確認しつつ翻訳しました。

affected_accounts は one と other で[複数形化](https://railsguides.jp/i18n.html#%E8%A4%87%E6%95%B0%E5%BD%A2%E5%8C%96)していたのですが、
日本語にすると同じだと思ったので、
一段階浅くして共通の翻訳を使うように変更しました。

`i1n-tasks add-missing -l ja` をした時に引用符がちょっとへんこうされてしまったのですが、
それもそのまま変更点として含めて、
[Add missing Japanese translations](https://github.com/tootsuite/mastodon/pull/1923)
として pull request を送りました。

`i18n-tasks unused -l ja` は本当に消して良いかどうかが不安だったので、
手をつけていません。

## まとめ

vagrant で簡単に mastodon の開発環境を構築できました。
ただしストリーミング API はそのままだと対応していないので注意が必要そうです。
