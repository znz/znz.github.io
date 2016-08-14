---
layout: post
title: "LILO&東海道らぐオフラインミーティング 2016/08/14"
date: 2016-08-14 13:00:00 +0900
comments: true
categories: lilo event linux
---
[LILO&amp;東海道らぐオフラインミーティング 2016/08/14 ブックマークする](http://lilo.connpass.com/event/37410/ "LILO&amp;東海道らぐオフラインミーティング 2016/08/14 ブックマークする") に参加しました。

今回もアンカンファレンス形式でした。

[Doorkeeper料金体系の変更について](https://www.doorkeeper.jp/news/2016/7/25/change-in-pricing "Doorkeeper料金体系の変更について")でアナウンスされたように Doorkeeper が有料化されることに伴い、申し込みは connpass に移行しました。

<!--more-->

## メモ

今回のメモです。

- いつものように鍵担当の人が遅れていたが、代理で開けていた
- ハッシュタグ: `#lilo_jp` `#東海道らぐ`
- 登録参加者数 14名
- 自己紹介から
- 最初は自分の発表「lilo.linux.or.jp の話」
- 「OpenStreetMap で地図を作ろう!」坂ノ下さん
- Map Compare というサイトで Google Map と OSM の平安神宮を比較すると、OSMの方が詳しい
- JOSM というクライアントで実演
- ノード、ウェイ、エリアででできている
- 誰でも書き込めるし消せるので、悪質なユーザーへの対処は日々行っている
- http://wiki.openstreetmap.org/wiki/JA:Map_Features
- 鳥居のタグの話
- どのようにタグをつけるのかは議論しながら決まっている
- 「IoTハウス」山内さん
- http://www.pepolinux.com https://twitter.com/kujiranodanna
- ラズパイで IoT ハウス
- Tocos, IRKit
- リセッタブルヒューズ
- 休憩
- 「お前が持っているLPICってどんなものよ? LPI 304受験報告記」中野さん
- 事前に公開されていた発表資料: https://bitbucket.org/itsango/lilo20160814
- 有意性の期限がある
- メリット: Linux が使える客観的な証拠になる
- デメリット: 高い
- 304 の参考書: https://amazon.jp/dp/4844380540
- TOEIC みたいに何か統計処理された採点方式らしい
- 「Windows 10 タブレットに Ubuntu 16.04 を色々入れてみた 2016 年度版」Kapper さん
- http://www.slideshare.net/kapper1224/windows10ubuntu16042016install-ubuntu1604-on-windows10-tablet-63862255
- Wubi for Ubuntu 16.04 が公式にタブレット対応
- 「Yocto を使った Linux Distro の作り方とハマり方」山口さん
- https://github.com/watatuki
- https://www.yoctoproject.org/
- たとえていえば Gentoo をクロスビルドにしたようなもの
- layer を組み合わせて構成
- recipe はソースと 1対1 対応
- 複数の layer の組み合わせが問題でビルドが通らなくなることがある
- Android でおなじみの repo でいい感じできた
- 例: https://github.com/watatuki/agl-jetson-tk1
- 休憩
- 「TUI作業で便利なソフト2題」島田さん
- opencocon の紹介
- build server
- このごろあった悩み: ファイルツリーを駆け回るのがめんどい
- 解決法：TUI ファイラー
- mc (Midnight Commander)
- あんまり慣れてない
- Ctrl キー等を多用する
- F1-F12 を使わなければならない
- tmux とキーバインドが干渉しやすい
- 他に選択肢がないか?
- FDclone
- このごろあった悩み: git コマンドをいちいち叩くのがめんどい
- tig
- 「Docker話」左川さん
- vagrant で docker を試した話
- box ファイルは VirtualBox で普通にインストールして不要なものを削除して作成した。
- 会場から packer がおすすめという話
- さくらインターネットの Arukas はまだ誰も使ったことがない
- 今回の会場費は余剰金があるので無料になった。
- 「sedの話」田川さん
- sed はプログラミング言語
- デバッグオプション `-d` のある sed
- sdb というデバッガをネットワーク経由で接続
- 「せっかくのプレゼン資料なんだから Git で管理しよう Slides を公開しよう」中野さん
- 事前に公開されていた資料: https://bitbucket.org/itsango/vcsforslides
- http://progit-ja.github.io/
- [OSSホスティングサービスの比較](https://ja.wikipedia.org/wiki/OSS%E3%83%9B%E3%82%B9%E3%83%86%E3%82%A3%E3%83%B3%E3%82%B0%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9%E3%81%AE%E6%AF%94%E8%BC%83 "OSSホスティングサービスの比較")
- 次は冬休みの予定 (その前に k-of.jp にも参加予定)

## 発表した内容

lilo.linux.or.jp のサーバーの前回の発表以降の話をしました。

内容は大きく分けると 二要素認証は進捗なし、 letsencrypt の証明書は certbot に変わっても順調に使えている話、 ufw でアタックが多いポートをログに残さないようにした話でした。

スライドはいつも通り [Rabbit Slide Show](http://slide.rabbit-shocker.org/authors/znz/lilo-20160814/) ([RubyGems](https://rubygems.org/gems/rabbit-slide-znz-lilo-20160814)), [SlideShare](http://www.slideshare.net/znzjp/lilo-20160814), [Speaker Deck](https://speakerdeck.com/znz/lilo-dot-linux-dot-or-dot-jp-falsehua) にあげています。(ソースは [github](https://github.com/znz/lilo-20160814) にあげています。)
しかし、2016-08-14現在 slide.rabbit-shocker.org (Rabbit Slide Show) には反映されていないようなので、 SlideShare か Speaker Deck でみてください。

<!--
<iframe src="http://slide.rabbit-shocker.org/authors/znz/lilo-20160814/viewer.html"
        width="640" height="524"
        frameborder="0"
        marginwidth="0"
        marginheight="0"
        scrolling="no"
        style="border: 1px solid #ccc; border-width: 1px 1px 0; margin-bottom: 5px"
        allowfullscreen> </iframe>
<div style="margin-bottom: 5px">
  <a href="http://slide.rabbit-shocker.org/authors/znz/lilo-20160814/" title="lilo.linux.or.jp の話">lilo.linux.or.jp の話</a>
</div>
-->
