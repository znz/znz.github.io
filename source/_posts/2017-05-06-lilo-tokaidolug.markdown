---
layout: post
title: " LILO&東海道らぐオフラインミーティング 2017/05/06"
date: 2017-05-06 13:17:50 +0900
comments: true
categories: lilo event linux
---
[LILO&amp;東海道らぐオフラインミーティング 2017/05/06](https://lilo.connpass.com/event/55003/ "LILO&amp;東海道らぐオフラインミーティング 2017/05/06") に参加しました。

今回もアンカンファレンス形式でした。

<!--more-->

## メモ

今回のメモです。

- 会場はいつもの 4 階でした。
- ハッシュタグ: `#lilo_jp` `#東海道らぐ`
- 登録参加者数 16名
- 自己紹介から

## 自分の発表

- 資料にある話
- 資料にないけど DNS CAA の話
- CAA レコードに書いて spam が増えなかったかという話
- 公開しているアドレスなので変化はわからず
- .travis.yml に書いた時には明らかに増えた
- 質問があったので、ちょっと etckeeper の話 (etckeeper vcs とか)

## こんどうさん

- ESP パーティションの話
- ESP 内の grub データを消せば Windows のみになる
- BIOS/MBR から UEFI の過渡期の話
- openSUSE の話
- openSUSE Asia Summit 招聘中
- LibO もほぼ併催確定
- Cinnamon を入れた
- Linux Mint 17.3 with NVIDIA Driver は GPU 切り替えがログアウトだけで可能
- 18.1 では切り替え NG (プロプラドライバとの相性も悪い)
- M17N: ibus の設定
- SUSE のデフォルトは `/` が btrfs で `/home` が XFS

## 5分休憩

## としひささん

- スマートウォッチ
- WSD-F20 と Pebble
- 必要な理由とか要件とか
- 特徴とか比較とか
- 個人的には Pebble の方が良い
- WSD-F20 は GPS ロガーにはならない
- Android Wear 2.0 はアプリが少ない

- LILO の歴史 https://lilo.linux.or.jp/history/
- 今年で 20 年
- インストールからサポートする場、九州でやっているから関西でも、というのが発足理由
- Samba の話で 150 人ぐらいきたことがあるのがピーク
- 普通は多くて50人ぐらい
- 普通は10から20人ぐらい
- タコを育てようという文化
- JF がすばらしい
- Linux は初心者を大切にするのが良い
- https://twitter.com/xoxyuxu/status/860737579657732098 JFは素晴らしい(ドキュメントがあることは素晴らしい) タコを育てよ(判らないことがある環境が悪い,という考え方)
- みんないろんなものを持ってきていた
- NAIST でやっていた奈良時代
- 場所が先端
- デスクトップマシンを持ってくるのが普通
- Doom をやりたくて Linux を入れた人もいる
- Enlightment は昔からかっこいい
- 昔は重かったが時代が変わって今は軽い
- Tizen にのっているので今も開発は活発
- 第3回のアンケート結果
- 2002年より後の歴史は分散していてまとまっていない
- Wiki にある https://lilo.linux.or.jp/wiki/history
- トップから辿れる一覧ページもある https://lilo.linux.or.jp/event/
- LMS だけ別ページ https://lilo.linux.or.jp/event/lms/

## さかのしたさん

- 「Google map 印刷 制約」で検索
- https://www.google.co.jp/intl/ja/permissions/geoguidelines.html
- Google Map は使用に制限がある
- Open Street Map
- facebook のチェックインなどに使われている
- overpass turbo
- f4map - 延暦寺
- [OSM Access Map](https://www.netfort.gr.jp/~saka/accessmap/) というのを作った
- 道路のみの画像を保存できる
- 道路の区別は実データを元にご意見募集中
- ゼンリンが地図グッズを売っている http://www.zenrin.co.jp/goods/matimati/item/
- SUZURI でグッズを売る?
- 5月20日(土) 【西国街道#04】摂津富田の街並みと寺社巡り https://countries-romantic.connpass.com/event/56126/

## やまうちさん

- 実践IOTハウス 古いPCで
- Tocos 無線モジュール
- Raspberry Pi が遅いので古い PC に置き換え
- Linux Beans
- TOCOS TWE-Lite と ToCoStick で簡易照度センサー(100均電卓)
- 太陽電池部分の電圧をとっている
- デモはうまく動かず

## 休憩

- 懇親会参加予定は 10 名
- 会場費は 100 円

## もりわかさん

- https://developers.redhat.com/
- 登録すると 1 年間無料で開発用途に使える
- 1 年後に更新すればずっと使える
- シェルスクリプト10行ぐらい書くよねという話
- JBoss などもある
- ナレッジベースやドキュメントにもアクセスできる
- 質の高いドキュメント
- 実マシン1台という制限があるが、VM の制限はないのですごいマシンを用意して 100 VM とかでも OK
- 会社で開発者がそれぞれ登録して手元のマシンはそれで動かすと言うのもあり
- バグかどうか困ったらサポートを買うのが良い

## のがたさん

- [第462回　韓国開催，Korea Community Day参加レポート 〜Ubuntu KRのみなさんと交流してきました：Ubuntu Weekly Recipe｜gihyo.jp … 技術評論社](http://gihyo.jp/admin/serial/01/ubuntu-recipe/0462)
- 記事とは違う話を
- http://cogniti.github.io/nimf/ja/ (ibus などの層のもの) の開発者の人と Google 翻訳を使って話をした
- Google 翻訳は韓国語と日本語の翻訳はスラングなどを使わずきれいに話せば精度良く翻訳できるので、突っ込んだ話もできた
- Samsung と Tizen
- 言語が違うだけでやっていることは同じだと思った話

## ぉゅぅさん

- Ext4 filesystem ではまった話
- 32 bit OS だと 16TiB まで
- ブロックアロケータまで調べた
- `posix_fallocate`
- http://www.ujiya.net/linux/

## 今後の予定

- 次回はたぶん 8 月

## 発表した内容

スライドはいつも通り [Rabbit Slide Show](http://slide.rabbit-shocker.org/authors/znz/lilo-20170506/) ([RubyGems](https://rubygems.org/gems/rabbit-slide-znz-lilo-20170506)), [SlideShare](https://slideshare.net/znzjp/lilolinuxorjp-20175), [Speaker Deck](https://speakerdeck.com/u/znz/p/lilo-dot-linux-dot-or-dot-jp-falsehua-2017nian-5yue) にあげています。(ソースは [github](https://github.com/znz/lilo-20170506) にあげています。)

<iframe src="https://slide.rabbit-shocker.org/authors/znz/lilo-20170506/viewer.html"
        width="640" height="524"
        frameborder="0"
        marginwidth="0"
        marginheight="0"
        scrolling="no"
        style="border: 1px solid #ccc; border-width: 1px 1px 0; margin-bottom: 5px"
        allowfullscreen> </iframe>
<div style="margin-bottom: 5px">
  <a href="https://slide.rabbit-shocker.org/authors/znz/lilo-20170506/" title="lilo.linux.or.jp の話 (2017年5月)">lilo.linux.or.jp の話 (2017年5月)</a>
</div>

## Togetter まとめ

[LILO&amp;東海道らぐオフラインミーティング 2017/05/06の記録 - Togetterまとめ](https://togetter.com/li/1107707)にまとめられているようです。
