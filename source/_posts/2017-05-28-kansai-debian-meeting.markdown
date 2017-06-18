---
layout: post
title: "第 123 回関西 Debian 勉強会に参加しました"
date: 2017-05-28 13:50:00 +0900
comments: true
categories: debian event
---
[第 123 回関西 Debian 勉強会](https://debianjp.connpass.com/event/57323/ "第 123 回関西 Debian 勉強会")に参加しました。

<!--more-->

以下メモです。

## 会場など

Apache Arrow の勉強会の後、一緒にランチに行くのに参加していたら、ちょっと遅くなってしまって、 13:40 すぎに到着しました。

## オープニング

- 事前課題は必須項目になっていなかったため、 connpass での参加登録時に出ていなかったので、その場で発表して資料が更新されていた。
- 初参加の人がいた
- 初めて作った人とか長い間使っていない人とか
- easypg は表示は復号されているがメモリダンプをしても平文はないらしい
- 期限切れと revoke の違い: 期限切れはのばせるが、 revoke は取り消せない

ruby の svn のコミットに必要という話は https://bugs.ruby-lang.org/projects/ruby/wiki/CommitterHowto https://bugs.ruby-lang.org/projects/ruby/wiki/CommitterHowtoJa あたりに書いてあるのですが、今見ると `ssh-keygen -b 2048 -f ruby_key` になってて古い感じ。英語の方は `~/.ssh/id_rsa or ~/.ssh/id_dsa` と書いてあるのでもっと古い感じが。

### 途中で矢吹さんの話

- カジュアルに使うのも良いけど、暗号関連は色々気にしている人もいるので気をつけましょう (色々なポリシーがあるのを尊重しましょう) という話とか
- cypherpunk : 暗号の民主化
- FST-01 を入手した (まだ秘密鍵は入れていない) ので回覧

## GnuPG にまつわるアレコレ - GnuPG2 とか Keybase とか Yubikey とか

- key id は昔は 8 文字だったけど conflict が作れるという話になって、今は 16 文字 (以上?) が普通らしい
- (勉強会中に聞きそびれてしまったけど、GPG の key id が fingerprint の末尾っていうのは、みんなどうやって知るんだろう?)
- (自分は昔の key sign party で fingerprint しか書いてないけど、どうやって fetch するの? と聞いたときに教えてもらったけど。)
- GnuPG とは?
- OpenPGP: RFC になっている
- 暗号化によって盗聴を防ぐ (機密性)
- 署名によって改ざんを検知可能にする (完全性)
- 例: secure-apt, DKIM
- 信頼の輪 (Web of trust) によって本人を同定
- 過去の資料の参照情報紹介
- [第 101 回 関西 Debian 勉強会](https://wiki.debian.org/KansaiDebianMeeting/20150823 "第 101 回 関西 Debian 勉強会")の「wiki:Subkeys」が気になった
- Windows の GUI の GPG クライアントで良いものがない
- mac は GPG Tools というので標準の Mail.app で GPG 対応できるらしい
- Web メールで困る話
- Thunderbird は Enigmail
- 鍵の選び方の話
- ED25519 は gnupg 1.4 系 (今は oldstable ブランチ) で復号できない
- 今の所は互換性のための RSA4096 と速い ED25519 を併用するのが良いかもしれない
- GnuPG 2 - Stretch での変更点
- gnupg (1.4 → 2.1), gnupg1 (なし → 1.4), gnupg2 (2.0 → drop)
- gnupg 2.1 は初回実行時に自動的に移行が実行されて、2.1 用のファイル (`pubring.kbx` とか `private-keys-v1.d/` とか) が作成される
- 古いバージョン用のファイル (`pubring.gpg` とか `secring.gpg` とか) はそのまま残るので古いバージョンの共存も可能 (だがそれぞれのバージョンのファイルしか更新されないので、併用すると情報がずれていくはず)
- 2.0 がどっちのファイルを使うのかは検証してないので不明
- プログラムの分離具合が整理された ([Mini Debian Conference Japan 2016](http://miniconf.debian.or.jp/) の[g新部さんのスライド](http://miniconf.debian.or.jp/assets/files/gnupg-now.html) の図が非常にわかりやすい (GnuPG programs (2) の図))
- proxy 設定が gpg.conf から dirmngr.conf に変わった
- use-gpg-agent が obsolete (常に有効で単純に無視される)
- 大量の ssh 接続に使うと scdaemon がエントロピーが足りないという話
- haveged (はべーじでぃー(と言っていた)) とか rng-tools とか
- disable-scdaemon をつけておくと無駄に scdaemon が起動しなくなる (gpg-agent から scdaemon が起動して scdaemon 対応のデバイスがなければ、すぐに gpg-agent に返ってくるらしいが、起動するだけ無駄という話)
- gpg-agent を ssh-agent として使うのは簡単
- GNOME で gnome keyring daemon は止めやすい
- mate や xfce では ssh-agent を止めにくい
- ssh-agent を入れないときは `.xsessionrc` で `SSH_AUTH_SOCK` を自分で設定する必要がある
- mate や xfce では `.xsessionrc` で設定しても、さらに上書きされるらしい
- YubiKey https://yubikey.yubion.com/ https://www.cloudgate.jp/yubikey.html
- ED25519 には対応していない
- RSA 4096 で復号で 0.5 秒ぐらい
- (後でし
- Qt5 での GUI ツールとコマンドラインツールがある
- keybase.io というサービスがあるらしいが秘密鍵もアップロードできるらしく?
- アカウントを作ってみた https://keybase.io/uwabami が消すかも。
- https://github.com/keybase/client には大量の Issues が。

## クロージング

- 今後の予定
- 来月は17日に stretch がリリース予定なので、25日から18日に予定を変更してリリースパーティの可能性が高そう

## よくわからないままだった点

- 懇親会への移動中などにある程度話しましたが
- `gpg --version` で出てくる「公開鍵: RSA, ELG, DSA, ECDH, ECDSA, EDDSA」の「EDDSA」(ECDSA ではなく)と「ED25519」は同じ?
- Curve25519 と ED25519 の関係
- [エドワーズ曲線デジタル署名アルゴリズム](https://ja.wikipedia.org/wiki/%E3%82%A8%E3%83%89%E3%83%AF%E3%83%BC%E3%82%BA%E6%9B%B2%E7%B7%9A%E3%83%87%E3%82%B8%E3%82%BF%E3%83%AB%E7%BD%B2%E5%90%8D%E3%82%A2%E3%83%AB%E3%82%B4%E3%83%AA%E3%82%BA%E3%83%A0 "エドワーズ曲線デジタル署名アルゴリズム") に多少説明がある
