---
layout: post
title: "Amagasakirb特別編 須藤さん関連プロダクツ勉強会に参加した"
date: 2014-06-28 13:16:24 +0900
comments: true
categories: event amagasakirb
---
[6月28日 Amagasakirb特別編 須藤さん関連プロダクツ勉強会(兵庫県)](http://kokucheese.com/event/index/186111/ "6月28日 Amagasakirb特別編 須藤さん関連プロダクツ勉強会(兵庫県)")
に参加しました。

<!--more-->

## 自己紹介

まず最初に、使っている須藤さん関連プロダクツの話を中心に自己紹介していました。
イベントのページには書いていませんでしたが、
[るりまサーチ](http://docs.ruby-lang.org/ja/search/ "るりまサーチ")
も須藤さん関連プロダクツでした。

## メモ

その後は須藤さん関連プロダクツの話をいろいろしていたので、
その中からのメモです。
話としては groonga 関連と ActiveLdap 関連が多かったです。

### groonga 関連

- [数万の電子書籍から目的のページを一瞬で見つけ出す、Honyomi - ブログのおんがえし](http://ongaeshi.hatenablog.com/entry/honyomi-init "数万の電子書籍から目的のページを一瞬で見つけ出す、Honyomi - ブログのおんがえし")
- [Milkode - 行指向のソースコード検索エンジン](http://milkode.ongaeshi.me/ "Milkode - 行指向のソースコード検索エンジン")
- [PatentField | 無料特許検索](http://patentfield.com/%E3%83%A1%E3%82%A4%E3%83%B3%E3%83%9A%E3%83%BC%E3%82%B8 "PatentField | 無料特許検索")

### ActiveLdap 関連

- 値が1個しかなくても常に配列を返す使い方も出来る
- 使い方が Net-LDAP より直感的

### groonga と競合する?プロダクト

- solr
- lucene
- xapian

### mroonga 関連メモ

他の話はメモしきれなかったので、
テーブルの情報取得に

- `show create table test;`
- `desc test;`

の2種類の方法を使っていたことぐらいはメモしておきたいと思いました。

### ChupaText

- チュパカブラが名前の由来
- groonga と組み合わせれば Hyper Estraier のようなものが作れるかも

### Jekyll のコンテンツの翻訳の話

- [Jekyllで複数言語のコンテンツを継続してメンテナンスする方法 - ククログ(2014-04-23)](http://www.clear-code.com/blog/2014/4/23.html "Jekyllで複数言語のコンテンツを継続してメンテナンスする方法 - ククログ(2014-04-23)")
- [www.ruby-lang.org - News Post Translation Status](https://www.ruby-lang.org/admin/translation-status/ "www.ruby-lang.org - News Post Translation Status")

### 須藤さん関連

- http://slide.rabbit-shocker.org/authors/kou/
- https://github.com/kou
- https://github.com/clear-code/sezemi-2014-readable-code

### 最後に須藤さんの過去のプレゼンから

- http://slide.rabbit-shocker.org/authors/kou/devmi-2013/

## rabbiter 問題

自己紹介の時に rabbit を使ったので、せっかくなので、
rabbiter を使おうとしたのですが、今回も事前に動かせる状態までは
もっていけませんでした。

しかし、
Homebrew で rabbiter がインストールできなかったり、動かなかったりしたのが、
須藤さんの助言で解決しました。

まず `libffi` は `PKG_CONFIG_PATH` で `libffi.pc` の場所を指定すれば通りました。

```
    export PKG_CONFIG_PATH=/usr/local/Cellar/libffi/3.0.13/lib/pkgconfig
    gem install rabbiter
```

この件は `/usr/local/lib/pkgconfig` に symlink がないのが問題なので、
[Homebrew 側の問題の修正](https://github.com/Homebrew/homebrew/pull/30516)
を pull request してみました。


次に `rabbiter --filter twitter` のように起動しても
`(rabbiter:99703): GLib-GIO-ERROR **: No GSettings schemas are installed on the system`
ですぐに落ちてしまうという問題が起きました。

これは須藤さんに調べてもらって、環境変数 `GSETTINGS_SCHEMA_DIR` で `schemas` のディレクトリを指定すれば良いということで、

```
    export GSETTINGS_SCHEMA_DIR=/usr/local/Cellar/gsettings-desktop-schemas/3.12.2/share/glib-2.0/schemas
    rabbiter --filter 'twitter'
```

のように起動して解決しました。

こちらは対処方法がわからなかったので、
[gsettings-desktop-schemas crashes under current configure options](https://github.com/Homebrew/homebrew/issues/26455)
という同様の現象が報告されている issue に回避策をコメントしておきました。

その後も以下のエラーで動いていませんでしたが、これは
`~/.rabbit/twitter-oauth.yaml`
が古いままで、アプリ連携の許可を取り消してしまっていたためで、
ファイルを削除して許可し直せば直りました。

```
    % rabbiter --filter 'twitter'
    [エラー]
    [twitter] invalid status code: 401.
```


## 自分のスライド

最後に自分の自己紹介の時に使ったスライドを載せておきます。
`ActiveLdap` と書いていますが、
`ActiveSambaLdap` の間違いでした。

<iframe src="http://slide.rabbit-shocker.org/authors/znz/amagasakirb-201406/viewer.html"
        width="640" height="524"
        frameborder="0"
        marginwidth="0"
        marginheight="0"
        scrolling="no"
        style="border: 1px solid #ccc; border-width: 1px 1px 0; margin-bottom: 5px"
        allowfullscreen> </iframe>
<div style="margin-bottom: 5px">
  <a href="http://slide.rabbit-shocker.org/authors/znz/amagasakirb-201406/" title="普段使っている須藤さん関連プロダクツ">普段使っている須藤さん関連プロダクツ</a>
</div>
