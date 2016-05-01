---
layout: post
title: "LILO&東海道らぐオフラインミーティング 2016/05/01 に参加しました"
date: 2016-05-01 13:00:00 +0900
comments: true
categories: lilo event linux
---
[LILO&amp;東海道らぐオフラインミーティング 2016/05/01](https://lilo.doorkeeper.jp/events/42910 "LILO&amp;東海道らぐオフラインミーティング 2016/05/01")
に参加しました。

今回もアンカンファレンス形式でした。

<!--more-->

## メモ

今回のメモです。

- いつものように鍵担当の人が遅れていた
- ハッシュタグ: `#lilo_jp` `#東海道らぐ`
- 登録参加者数 13名
- 自己紹介から
- はしもとさん : 東海道らぐともうひとつの東海道
- 東海道らぐ 今年で 5 周年目突入
- インプットメソッドの話
- https://github.com/tkd53
- Genji も方向性が違うので続けるという話
- 自分の発表
- 休憩
- Kapperさん : シンガポールFossasia2016に初参加してみた
- あひる焼き先進国
- シンガポール英語とインド英語
- マニアックな話が多かった
- アンカンファレンス形式は海外ではメジャー
- 3月はチケット代が高い
- 京橋ひよわさん : 自宅サーバのトラブルを楽しもう
- http://hiyowa.com/
- `SSLCertificateChainFile` がなくなっていたはまった話で nginx は単純に cat で中間証明書を結合すればいけるが apache はダメらしい http://serverfault.com/questions/588986/sslcertificatechainfile-deprecation-warning-on-apache-2-4-8
- owncloud はエラー表示が親切
- こんどうさん : やりなおし方について
- デュアルブートを諦めれば楽
- fdisk /mbr とか fixmbr とか
- EaseUS Todo Backup で D2D Backup
- Windows 10 が勝手にアップグレードする話
- UEFI だと ESP パーティションをマウントして Linux のブート情報を消すだけ
- おくのさん : 格安モバイルノート PC で Linux
- hp Stream 11-d000 の 2 代目モデル
- hp Stream 11-r000
- Wi-Fi がカニから Intel
- Linux Mint 17.3 は Wi-Fi が新しすぎて対応していなかったのでアップグレードが必要だった
- Lubuntu 16.04 だと問題なく動いた
- DELL New Inspiron 11 3000 シリーズの方が良さそうだったというオチ
- 休憩
- しまださん : このごろの状況
- opencocon v9i
- Libretto L1 の CD ブート対応
- ビルド時間 3 時間の半分は WebKit
- opencocon v10
- Linux 4.4
- uClibc から musl libc に変更
- musl libc を採用しているディストリ
- 標準採用 : Alpine Linux
- 選択可能 : Gentoo, Buildroot, OpenEmbedded, などなど
- ext4 に正式対応
- grub-legacy も Arch Linux のパッチで対応
- Dynabook AZ + Linux 3.18 〜 を公式サポートする唯一のディストリビューションになる予定
- AZ はシンクライアント用途にしてはちょっと高速すぎる?
- デスクトップでもいいぐらい
- v10 から名前に thinclient が入って opencocon thinclient v10 になる
- Allwinner タブレットの話
- さとうさん
- `sleep 300; poweroff` を実行してから開始
- タイムラプス
- 赤外線カメラと通常のカメラを並べてくっつけて同時に撮影
- raspistill で撮影
- mencoder で jpg から動画に変換
- 動画表示デモ
- ききょうさん : openSUSE で bot を作ろう
- 鹿焼き
- BotBuilder/Connector/Directly
- LILO&東海道らぐの次回は 8 月 (お盆) ぐらいの予定

## 発表した内容

このオフラインミーティングのネタのためも兼ねて、 lilo.linux.or.jp のサーバーを wheezy から jessie にあげたので、その話をしました。

内容は大きく分けると jessie にあげた話、二要素認証を導入した話、 letsencrypt の証明書を導入した話でした。

スライドはいつも通り [Rabbit Slide Show](http://slide.rabbit-shocker.org/authors/znz/lilo-20160501/) ([RubyGems](https://rubygems.org/gems/rabbit-slide-znz-lilo-20160501)), [SlideShare](http://www.slideshare.net/znzjp/lilo-20160501), [Speaker Deck](https://speakerdeck.com/znz/lilo-dot-linux-dot-or-dot-jp-from-wheezy-to-jessie) にあげています。

<iframe src="http://slide.rabbit-shocker.org/authors/znz/lilo-20160501/viewer.html"
        width="640" height="524"
        frameborder="0"
        marginwidth="0"
        marginheight="0"
        scrolling="no"
        style="border: 1px solid #ccc; border-width: 1px 1px 0; margin-bottom: 5px"
        allowfullscreen> </iframe>
<div style="margin-bottom: 5px">
  <a href="http://slide.rabbit-shocker.org/authors/znz/lilo-20160501/" title="lilo.linux.or.jp を wheezy から jessie にあげた話">lilo.linux.or.jp を wheezy から jessie にあげた話</a>
</div>
