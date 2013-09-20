---
layout: post
title: "Ubuntuでの対応が悪いと感じた話"
date: 2013-09-20 19:43
comments: true
categories: ubuntu bts community
---
このエントリは愚痴が含まれます。
そういうのが嫌な人は読まない方が良いですよ。
警告しましたからね。

<!--more-->

結論としては、こういういろいろなことが積み重なって、
最近は ubuntu にはあまり積極的には貢献をしないようになった
という話です。

コミュニティ運営のアンチパターンの例として使えるのかもしれません。

最初の4項目は Ubuntu の Launchpad での話で、
後の2項目は ubuntu-jp での話です。

もちろん、対応が悪かったものだけ集めているので、
他の多数のバグの対応は良かったことが多いです。
全体的に Ubuntu のバグ対応は
Debian に比べると遅いという気はしますが。

## 最近の例 (bind9)

最初は Ubuntu 12.04 LTS の `bind9` の
`/etc/bind/db.root` が更新されない話です。

どういう話かというと
D-ROOT-SERVERS.NET に IPv6 アドレスが追加されて、
その後 IPv4 アドレスの変更があったのに `db.root` が
更新されないという話です。

ちょっと探して見つかっただけでも4個見つかります。
4個目のは3個目の重複という印がついてます。

* [db.root file outdated](https://launchpad.net/ubuntu/+source/bind9/+bug/1023600)
* [Bind db.root outdated, backport only the db.root file, for LTS important to keep db.root updated](https://bugs.launchpad.net/ubuntu/+source/bind9/+bug/1160464)
* [D.ROOT-SERVERS.NET changing January 3rd 2013](https://bugs.launchpad.net/ubuntu/+source/bind9/+bug/1090593)
* [d-root ipv4 has changed. Update db.root to show this](https://bugs.launchpad.net/ubuntu/+source/bind9/+bug/1097050)

IPv6 アドレスが追加された時点では重要ではないということで、
次の更新があれば一緒に更新するという反応だったようですが、
2013年1月3日に IPv4 アドレスが変更されて、
その後、移行期間として6ヶ月あったのですが、
それが終わった後も更新されないままです。

事前に見つけていた上2個のbugは移行期間中につつてみたりしたのですが、
全く反応がありませんでした。

そして未だに更新されていません。
これはひどいと思います。

## quick-fix?

[Version mismatch of link in Code of Conduct](https://bugs.launchpad.net/ubuntu-website-content/+bug/619920)
で CoC の説明のバージョンが間違っていると指摘した時の話です。

一度 Confirmed になったのに Invalid になったというのが
嫌な印象を受けました。

## dbskkd-cdb を rebuild するだけの簡単なお仕事

[wrong reply via netkit-inetd](https://bugs.launchpad.net/ubuntu/+source/dbskkd-cdb/+bug/58355)
で dbskkd-cdb を rebuild するだけで直るという報告をしたつもりだったのですが、
今見直してみると、こちらの反応がなかったのが悪かったようにも見えました。
追加情報を求められた時にはもう使ってなかったとはいえ、ちょっと反省。

## 気付くのが遅すぎた

[FreezeException request-- Sync with Debian unstable](https://bugs.launchpad.net/ubuntu/+source/inspircd/+bug/978206)
で inspircd が古すぎて upstream の
セキュリティアップデートも出ないようなバージョンなので、
まずいんじゃないかと伝えてみたつもりだったんですが、
結局古いまま入ってしまいました。

古いまま入るぐらいなら消せば良かったのに、と思いましたが、
英語で書けなかった
(そもそも最初の説明も書けなかったのでリンクですませた)
ので、
自分で使うのは inspircd を止めて ngircd に乗り換えました。

## 何もやってない (ように見える) 話

「#ubuntu-jp IRCミーティングの議事録」というのが
ubuntu-jp の ML に流れるのですが、
アクションアイテムの冒頭が毎回同じで、
そこだけみると
(というかいつも最初の数行しかみてないのですが)
いつも何もしていないように見えるので、
順番を変えた方が良いんじゃないか、
というようなことを IRC とかで言ってみたことが
あるのですが、何も変わりませんでした。

## 査読とは?

アクションアイテムをもうちょっと下まで見た時に
`* [ ] 誰か査読して！`
と書いてあったので、
読んでみて返信してみたのですが、
次の週もそのままだったので、
あれは査読に含まれないのかと思ってがっかりしました。
