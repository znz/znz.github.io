---
layout: post
title: "LILO&東海道らぐオフラインミーティング 2017/08/27 に参加しました"
date: 2017-08-27 13:33:33 +0900
comments: true
categories: lilo event linux
---
[LILO&amp;東海道らぐオフラインミーティング 2017/08/27](https://lilo.connpass.com/event/64381/) に参加しました。

今回もアンカンファレンス形式でした。

<!--more-->

## メモ

- 会場はいつもの 4 階でした。
- ハッシュタグ: `#lilo_jp` `#東海道らぐ`
- 登録参加者数 12名
- 自己紹介から
- 最初の発表は発表者の希望により非公開
- 途中からきた人は発表の合間に随時自己紹介

## 自分の発表

- 自分の lilo.linux.or.jp の話
- 発表資料に入れていなかった部分については以下の通り
- OGP を入れたきっかけは [LILO&東海道らぐオフラインミーティング 2017/05/06の記録 - Togetterまとめ](https://togetter.com/li/1107707) で connpass だけ概要と画像が出ていたため
- [Open Graph Debugger - Developers Facebook](https://developers.facebook.com/tools/debug/) (Facebook アカウントがないと使えない?) は OGP がなくても自動検出した内容で埋められるため、本当に反映されるかどうかの確認には使えなかった覚えがあります
- [[clamav-jp 281] ウィルスDB更新の異常について（解決済）](https://ja.osdn.net/projects/clamav-jp/lists/archive/users/2017-August/000280.html)
- [Postfix 2.12 の compatibility_level](http://qiita.com/ttdoda/items/f16422d709e264cbb8a1)
- dokuwiki は [#854592 dokuwiki: Unable to login, missing usr/share/php/Crypt/AES.php](https://bugs.debian.org/854592) で消えていた

## 休憩

## GPD-Pocket に Ubuntu 17.04 をインストールした話

- 東海道らぐ四日市 11/25
- GPD-Pocket でも Ubuntu 17.04 が動いた
- Kernel 4.13RC + Intel Graphics Driver OSS + 蓋開閉
- 色々な条件で試してNGだったが、偶然蓋を開けたら画面がうつった
- ATOM はバニラカーネルの時点でバグがあるらしい

## Fireduck OS

- 東海道LUG有志によるLinuxディストリビューションプロジェクト
- ○○焼き → Fire duck (あひる焼き)
- タブレット向け
- https://github.com/TokaidoLUG/fireduckos
- アーキテクチャ説明
- 悩み事
- journald が起動したプロセスの出力をファイルに書き込むので重い
- OSM のアプリ?
- UEFI32 向けに 64 bit 環境で 32 bit 向けのビルドが必要なので multilib を使った
- https://github.com/TokaidoLUG/meta-intel-mobile
- [Tokaido Linux User Group](https://github.com/TokaidoLUG)
- 欲しいアプリの話
- 資料は https://www.slideshare.net/wata2ki に公開予定

## 休憩

- 懇親会参加者の確認

## つなぎの発表

- オープンソースカンファレンス広島の紹介
- Allwinner タブレットの実演
- firefox を動かして、ネットワークが繋がっていないので about:mozilla とか about:robots とか

## State of the Map 2017 に行ってきたよ

- Open Street Map の国際会議
- 日本で国際会議をやるのは珍しい
- 会津若松市は LibreOffice を使っているので、ついでに話を聞きに行った
- 写真を見ながら色々な話
- Maps With Me というアプリが便利らしい

## 最後の話

プロジェクターに映らなかったので、集まって話をきいていました。

## クロージング

- 今後の予定など
- 会場費は学生以外が 100 円でした。

## 発表した内容

スライドはいつも通り [Rabbit Slide Show](http://slide.rabbit-shocker.org/authors/znz/lilo-20170827/) ([RubyGems](https://rubygems.org/gems/rabbit-slide-znz-lilo-20170827)), [SlideShare](https://slideshare.net/znzjp/lilolinuxorjp-20178), [Speaker Deck](https://speakerdeck.com/u/znz/p/lilo-dot-linux-dot-or-dot-jp-falsehua-2017nian-8yue) にあげています。(ソースは [github](https://github.com/znz/lilo-20170827) にあげています。)

<iframe src="https://slide.rabbit-shocker.org/authors/znz/lilo-20170827/viewer.html"
        width="640" height="524"
        frameborder="0"
        marginwidth="0"
        marginheight="0"
        scrolling="no"
        style="border: 1px solid #ccc; border-width: 1px 1px 0; margin-bottom: 5px"
        allowfullscreen> </iframe>
<div style="margin-bottom: 5px">
  <a href="https://slide.rabbit-shocker.org/authors/znz/lilo-20170827/" title="lilo.linux.or.jp の話 (2017年8月)">lilo.linux.or.jp の話 (2017年8月)</a>
</div>
