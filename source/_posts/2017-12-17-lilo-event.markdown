---
layout: post
title: "LILO 20周年記念ミートアップに参加しました"
date: 2017-12-17 23:06:45 +0900
comments: true
categories: lilo event linux
---
[LILO 20周年記念ミートアップ](https://lilo.connpass.com/event/73932/)に参加しました。
いくつか事前に発表されていたものもありましたが、他はいつも通りアンカンファレンス形式でした。

<!--more-->

以下、メモです。

## 会場

いつもより遅い14時からでした。

## オープニング

- 自己紹介案: 名前, いつから Linux を使っているのか?, 初めて使った Linux ディストリビューション, 最近の Linux の使い方
- 会場費とか会計の話とか (結局プールされているお金があるので、今回は参加費は無料だった)
- ハッシュタグは `#lilo_jp`

## 自己紹介など

- 名前, いつから Linux を使っているのか?, 初めて使った Linux ディストリビューション, 最近の Linux の使い方
- 好きなディストリビューション: Debian: 7, Gentoo: 1, Vine: 1, Rasbian: 2, Plamo: 2, Ubuntu: 1, CentOS: 1
- UNIX: Minix: 1, OS9: 1
- 好きな言語: AWK: 4, Python: 2, JavaScript: 1, bash: 3, 日本語: 1, C: 3, BASIC09: 1, Perl: 2, Ruby: 1, PHP: 1
- libc5 のバージョンアップではまったという話が数人
- 初めて使ったのは Slackware という人が多かった感じがする

## Linuxコミュニティ20年の振返りと学んだ事(もうちょと付け足し版)

- GPD Pocket で HDMI がうまく出なかったので Windows で発表
- id にこだわりがあるので、取れなかったサービスはあまり利用しない
- 1997-06-22 から (ML に投稿があって名前が決定した日)
- 運営者は特にいなくて、その時々でやっている人が違う
- linux.or.jp ドメイン管理者の JLA との窓口やさくらインターネットとの窓口はやっている
- LinuxMaMa というパッチを集めたサイトがあった
- IP マスカレードは最初はここにあって、本体に取り込まれた
- Linux JF, Linux JM
- linux-users ML の方が LinuxMaMa より後から知った
- UNIX が十万円以上する頃に Linux は CD-ROM 代だけで売っていた
- 営業さんはすごいと感じた話
- コミュニティはいろんな人がいる

- k-of.jp/2017 の時からの付け足しはお世話になったサイトとお店
- 最近は野良パッチというのはあまり見かけない
- 今は github があるが、昔は SCM が無料というのはなかった

- Google 翻訳などで翻訳のモチベーションが減っているかも
- 英語ができる人しか生き残っていない?
- 仕様書がない?
- ウォーターフォールではない, リーンに近い
- 開発プロセスも勉強になる
- コアはきれいだが周辺のデバイスドライバのソースはまちまち

- Windows が嫌いというより (シャットダウン時に時間がかかる) Windows Update が嫌い

## LILO(7)

- LILO の説明を man page にした
- LILO(8) だとブートローダーとかぶってしまう
- LILO のサイトの複数のページから日付が付いているものを集めた
- LibreOffice Calc にまとめて、重複などは TSV を AWK で処理
- イベント開催回数は 100 回以上 (141回ぐらい?)

## lilo.linux.or.jp の話

- いつもの更新情報
- GitHub のプライベートリポジトリを使わせてもらっていたのを GitLab.com に変更した
- Web コンテンツが非公開なのは内容のライセンスというよりもコミットログが適当なので公開したくないということの方が大きいようでした
- 発表資料などの置き場所として resources を作った
- デフォルトのライセンスも決めた

スライドはいつも通り、
- [Rabbit Slide Show](https://slide.rabbit-shocker.org/authors/znz/lilo-20171217/)
- [SlideShare](https://slideshare.net/znzjp/lilolinuxorjp-201712)
- [Speaker Deck](https://speakerdeck.com/u/znz/p/lilo-dot-linux-dot-or-dot-jp-falsehua-2017nian-12yue)
- [RubyGems.org](https://rubygems.org/gems/rabbit-slide-znz-lilo-20171217)
- [GitHub.com](https://github.com/znz/lilo-20171217)
で公開しています。

<iframe src="https://slide.rabbit-shocker.org/authors/znz/lilo-20171217/viewer.html"
        width="640" height="524"
        frameborder="0"
        marginwidth="0"
        marginheight="0"
        scrolling="no"
        style="border: 1px solid #ccc; border-width: 1px 1px 0; margin-bottom: 5px"
        allowfullscreen> </iframe>
<div style="margin-bottom: 5px">
  <a href="https://slide.rabbit-shocker.org/authors/znz/lilo-20171217/" title="lilo.linux.or.jp の話 (2017年12月)">lilo.linux.or.jp の話 (2017年12月)</a>
</div>

## ライセンス変更の話

- Ruby のリファレンスマニュアルは RWiki の頃に仮の適当なライセンス + 変更手順を決めていたので、今は CC にできた
- Ruby は 1.9.3 で GPL とのデュアルライセンスから 2条項 BSDL に切り替えた
- tdiary は GPL2 から GPL2+ に変わった
  - highlight の著作権者に入っているが、特に個別連絡はなかったので詳細はよくわからない
  - 日本の著作権法では異議申し立てがなければ問題はないはずという意見があった

スライドはいつも通り、
- [Rabbit Slide Show](https://slide.rabbit-shocker.org/authors/znz/change-license/)
- [SlideShare](https://www.slideshare.net/znzjp/ss-84299220)
- [Speaker Deck](https://speakerdeck.com/znz/raisensubian-geng-falsehua) (We had issues processing your talk になっていて見えない?)
- [RubyGems.org](https://rubygems.org/gems/rabbit-slide-znz-change-license)
- [GitHub.com](https://github.com/znz/change-license)
で公開しています。

<iframe src="https://slide.rabbit-shocker.org/authors/znz/change-license/viewer.html"
        width="640" height="524"
        frameborder="0"
        marginwidth="0"
        marginheight="0"
        scrolling="no"
        style="border: 1px solid #ccc; border-width: 1px 1px 0; margin-bottom: 5px"
        allowfullscreen> </iframe>
<div style="margin-bottom: 5px">
  <a href="https://slide.rabbit-shocker.org/authors/znz/change-license/" title="ライセンス変更の話">ライセンス変更の話</a>
</div>

## LibreOffice や各地の IT コミュニティ運営について

- LILO や KOF などのイベント関連とか
- OpenOffice.org や LibreOffice などのソフトウェア関係とか
- ノウハウの文書は少ない, [アート・オブ・コミュニティ](https://www.oreilly.co.jp/books/9784873114958/) とか
- コミュニティの目的によっていろいろ違う
- LILO は ML で反論があると意思決定が難しい
- connpass に移行して楽になった
- やり方は様々
- ミッション大事、最初のメンバーで文化が決まる
- 人のトラブルとか悩ましい
- CoC の話
- DroidKaigi は毎年スタッフが半分入れ替わるらしい
- 若い人がいない問題: 諦める or 取りに行く
- 既存のところに入るより新しく立ち上げた方が活躍しやすい問題
- 最初の慣なれるための場として活用してもらう?
- OSC 広島だとトップを学生にしてみたことがある
- LibreOffice コミュニティの話は省略

## 実数ってナンだ?

- 人工知能や機械学習をきっかけにして数学を勉強し直した
- 自然数はここでは1以上 (0 を含む場合もある)
- ペアノの公理
- 自然数→整数→有理数
- 有理数, 無理数は有比数, 無比数の方がわかりやすかったのに
- デデキントの有理数の切断
- 有理数→無理数
- 無限の和で近似する話
- 無限の話

## Processing でなんとなく

- Processing : アート向けのプログラミング環境
- setup と draw
- デモ
- ランダムに贈りあうシミュレーション
- https://gigazine.net/news/20170711-random-people-give-money-to-random-other-people/

## qemu-debootstrap で別アーキテクチャ環境が手軽に使えて便利

- arm64 (aarch64) のノート PC Pinebook を買った
- chroot と qemu-user-static で別アーキテクチャが使える
- Korea Community Day 2018 の紹介 (OSC のようなものらしい)

## おっさんの集中力について

- 適度に運動する
- 寝ることが重要
- 食事のデザインも大事
- マインドフルネスも大事
- スピリチュアル系はやめた方が良い
- 注意力は限りがある
- Task warrior, Time warrior
- anki

## 今後の予定

- [OSC 2018 Osaka](https://www.ospn.jp/osc2018-osaka/) の翌日の 2018-01-28(日) にまた関西Debian勉強会と合同で勉強会をやるらしい
