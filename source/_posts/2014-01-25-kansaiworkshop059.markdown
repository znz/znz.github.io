---
layout: post
title: "第59回 Ruby/Rails勉強会＠関西に参加しました"
date: 2014-01-25 13:36:50 +0900
comments: true
categories: event ruby rubykansai
---
[第59回 Ruby/Rails勉強会＠関西](https://github.com/rubykansai/workshops/wiki/Kansaiworkshop059) に参加しました。

一部メモを書いていたので公開しておきます。

<!--more-->

## IT関係の同人誌などの話

### 質疑応答

- Q ReVIEW の本は?
- A 存在を知ったのが後だったので、電子書籍で買っただけ。

- Q いくらぐらい?
- A コピー本などで無料のものもありますが、たとえば500円とか1200円とか。印刷代などの関係で決まります。

## gitlab

### メモ

- [C++テンプレート完全ガイド勉強会 vol.1大阪 : ATND](http://atnd.org/events/47159)
- [Minami.rb](http://qwik.jp/minamirb/)
- [関西Debian勉強会](https://wiki.debian.org/KansaiDebianMeeting)
- [Gitlab は貧者の github enterprise](http://www.slideshare.net/takafumionaka/is-there-anynecessityofusinggithubenterprise)
- [Gitorious](https://gitorious.org/)
- [Gitblit](http://gitblit.com/)
- chef とか puppet とか ansible とかが何か知っている人は会場には少なかった。
- [レガシー依存の悪循環こわい](https://twitter.com/koichiroo/status/425454305944952832/photo/1)
- [GitLab 6.0をインストールする - Markovnikov Laboratory](http://mrk1869.com/blog/gitlab_installation/)
- 「6.2で転けて、6.1で転けて、ここまで設定するのに3日かかった…」と書いてあって大変そうなので、 ansible playbook を使ったという話
- [20分(くらい)で GitLab をインストールする方法 - akishin999の日記](http://d.hatena.ne.jp/akishin999/20130819/1376868748)
- svn からの移行には GUI がある方が良い

## LT1 ダジャレ

- わたしを創ったもの R・C・フェラン 年刊ＳＦ傑作選１
- mecab で分解して置き換えてダジャレ生成
- 変なのもたくさん出てくる
- 評価関数

## Ruby 関西 10 周年

- [関西オープンソース 2004](https://k-of.jp/2004/)
- 2004年11月27日 第0回 Ruby勉強会
- 大前研一: 人間が変わる方法は3つしかない。 １番目は「時間配分を変える。」 ２番目は「住む場所 を変える。」 ３番目は「つきあう人を変える。」 この３つの要素でしか人間は変わらない。 最も無意味なのは「決意を新たにする」ことだ。
- 助け合いの力
- グラミン銀行
- Jim Rohn の名言 : あなたは、あなたのまわりの5人の平均です。
- http://thewriteway.com/2011/01/learning-by-doing-revisited/
- 多様性は善
- ハブとしてのRuby関西

## Ruby初級者向けレッスン

- `codepoints` とは Unicode のだったり、それぞれの encoding のだったり。
- `"Ruby関西".chars.each{|c| puts c}` の `.each` はなくても動くが `$VERBOSE=true` のときに `warning: passing a block to String#chars is deprecated` という警告がでる。
- https://gist.github.com/higaki/8147246
- https://github.com/higaki/learn_ruby_kansai_59

## 帰り際の話

- Windows 版の Ruby で 2.0.0 からエスケープシーケンスに対応しているのが、
  NEWS に書いていないなどソース以外に情報がない。
