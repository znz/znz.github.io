---
layout: post
title: "nadokaさんの0.8.6をリリースした関連の話"
date: 2013-11-29 23:30
comments: true
categories: community github ruby nadoka ansible
---
[nadoka さんの 0.8.6 をリリース](http://mla.n-z.jp/?ruby-list:49704)
したので、
その関連の話を書いてみようと思います。

<!--more-->

## なぜ続けているか

短い答えとしては、自分が使っているからというのが一番大きな理由です。

bot というか plugin もいくつか作って、
サーバーの管理などにも便利に使っていて、
わざわざ他の IRC proxy 的なソフトに乗り換えて
プラグインを書き直すよりは、
一応動いているものをメンテナンスし続けた方が楽というのが
理由になっています。

## subversion repository について

最初は
[atdot.net の nadoka さん](http://www.atdot.net/nadoka/nadoka.ja.html)
のところに書いてある
`http://www.atdot.net/svn/nadoka/trunk`
にあったものが、
ささださんのサーバー管理の都合なのか、
rubyforge に移行して今に至ります。

途中から
([HowToRelease](https://github.com/nadoka/nadoka/wiki/HowToRelease)
の記録によると 0.7.7 から)
は github メインに移行しています。

その後、しばらく放置していたかどうだったのか忘れましたが、
最近のリリースでは github での変更をリリースのタイミングで
rubyforge の svn にも git-svn を使ってマージしていました。

このマージも今回で最後になります。

その作業をするときに
`http://rubyforge.org/`
をみてみると
`RubyForge Could Not Connect to Database: `
というエラーになっていて、
[hsbt さん](https://twitter.com/hsbt/status/406423900432506881)
に
[RubyForgeは5月15日で終了予定](https://twitter.com/evanphx/status/399552820380053505)
という話を教えてもらいました。

古いサーバーで svn co して、
そのまま使っている場合でも使い続けられるように続けていましたが、
rubyforge 自体が終わるということで、
そういうサーバーでは、
そのまま最後の svn up をして使い続けるか、
git に移行する必要がありそうです。

## git repository について

github が主流になっていたこともあり、
pull request とかしやすくなることを期待して
移行しました。

pull request が来た件数も 0 ではないので、
そのあたりは活発ではないプロジェクトとしては
うまくいっているのではないでしょうか。

github への移行方法として、
最初は
[tailor](http://darcs.net/RelatedSoftware/Tailor)
を検討したのですが、
既にあまり使われていなくて、
ちょっと試した感じでもうまくいかなかったので、
git-svn で移行しました。

今となっては変換専用ソフトはほぼ使えるものはなく、
変換先のソフト (今回は git) のプラグイン的なものを使って
変換元のソフト (今回は svn) の repository から取り出す、
という方法しかないようです。

## CloudCore VPS

[開発者支援制度 - CloudCore VPS](http://www.cloudcore.jp/vps/develop/)
でサーバーを借りてみて、
テスト用の IRC サーバーを動かしています。

接続用の設定は
[nadokarc-example](https://github.com/nadoka/nadokarc-example)
にあるので、適当に試したい時に使えると思います。

途中の経路の問題 (モバイルでモバイルルーターの接続が切れたとか?)
で、サーバーから応答がなくなったときに
nadoka さんの再接続がうまくいかないのを調査するのに使いたいと
思っているのですが、
切れた状況を再現する部分の作り込みがまだ出来ていないです。

ircd の設定は
[ansible-ircd](https://github.com/nadoka/ansible-ircd)
のように ansible でやってみました。
SSL の証明書の問題などがあるので、
すべての情報を公開できるわけじゃないというのが難しいところです。

## まとめ的なもの

長い間続いているといろいろあるものです。

小規模なプロジェクトなので、
いつも場当たり的な対応でなんとかなっていますが、
そういうのも良いんじゃないでしょうか。

とりあえず自分が使っている限りはリリースも続くと思いますし、
新しいバージョンの ruby への対応も続けていけると思います。
