---
layout: post
title: "Rubyによるクローラー開発技法 読書会 第2回(兵庫県)に参加しました"
date: 2014-11-01 13:05:51 +0900
comments: true
categories: event amagasakirb ruby
---
[11月1日 Rubyによるクローラー開発技法　読書会　第2回(兵庫県)](http://kokucheese.com/event/index/220392/ "11月1日 Rubyによるクローラー開発技法　読書会　第2回(兵庫県)")
に参加しました。
今回は3,4章でした。

<!--more-->

## メモ

<div style="float:right">
<iframe src="http://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&amp;bc1=000000&amp;IS2=1&amp;bg1=FFFFFF&amp;fc1=000000&amp;lc1=0000FF&amp;t=znz-22&amp;o=9&amp;p=8&amp;l=as4&amp;m=amazon&amp;f=ifr&amp;ref=ss_til&amp;asins=4797380357" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>
</div>

前回参加していなかった人向けに、最初に 1,2 章の概要説明があって、
今回は 3 章からでした。
今回から、誤字・脱字などのフィードバックのため、
Wiki
([2014.11.01 『Ruby によるクローラー開発技法』読書会 第2回](https://github.com/cuzic/amagasakirb/wiki/2014.11.01-Ruby-%E3%81%AB%E3%82%88%E3%82%8B%E3%82%AF%E3%83%AD%E3%83%BC%E3%83%A9%E3%83%BC%E9%96%8B%E7%99%BA%E6%8A%80%E6%B3%95%E8%AA%AD%E6%9B%B8%E4%BC%9A-%E7%AC%AC2%E5%9B%9E "2014.11.01 『Ruby によるクローラー開発技法』読書会 第2回"))
に記録するようになりました。

以下、今回のメモです。

- p.112 Crawl-delay の単位は決まっていない
- p.148 構文解析で正規表現が使われていることは多い?
- p.149 3-1-2 ECU_JP → EUC-JP
- p.151 行頭に `=~` は syntax error で、行頭に `.` は最近の ruby では OK になっている
- Fluent API の話
- p.150 RegExp → Regexp
- posix 文字クラスと Unicode プロパティの話
- Oniguruma と Onigmo の話
- pp.152-153 名前付きキャプチャの説明がローカル変数への割当だけの説明になっているようにみえる
- p.153 `\b` は文字クラスの中か外かで解釈が変わる
- p.154 `\x{7HHHHHHH}` は使えない https://github.com/rurema/doctree/issues/80 参照
- p.154 `\s` には `\v` が含まれる
- p.155 「m」オプションを使うことで改行を無視する → `.` で改行も含むようになる
- p.156 EUC_JP → EUC-JP
- https://github.com/cuzic/amagasakirb/wiki
- p.159 コードの中のコメント部分: UFT8 → UTF-8
- p.162 モンキーパッチではなく直接変更している話
- p.164 `xmlns:"デフォルトの名前空間識別子"` → `xmlns="デフォルトの名前空間識別子"`
- p.167 Webサイトの更新には、Atom配信フォーマットが利用できます。 → Atom出版プロトコルの間違い?
- p.171 Atom 1.0の構造は、RSS 2.0と同様に名前空間の指定が必要になります。 → RSS 1.0?
- close されていないので `open(url).read` よりも `open(url, &:read)` の方が良いのではという話
- 標準添付の RSS ライブラリが便利という話
- Google Feed API
- p.180 の `doc.at('//a').[]('href')` から `doc.xpath("//a/@href").text` などの話
- p.177 Aタグの例の `node.inner_text` の出力例が間違っている
- p.182 `\/` の `\` が不要?
- p.184 スクリーンショットの値段がずれている
- p.192 ここでは nokogiri ではなく REXML を使っている
- p.199, p.205 Marshal.dmp は Marshal.dump なのではないかという話
- mitmproxy の話
