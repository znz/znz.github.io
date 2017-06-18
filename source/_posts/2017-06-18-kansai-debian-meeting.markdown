---
layout: post
title: 'Debian 9 "Stretch" リリースパーティ in 関西に参加した'
date: 2017-06-18 13:24:10 +0900
comments: true
categories: debian event
---
[Debian 9 "Stretch" リリースパーティ in 関西](https://debianjp.connpass.com/event/59443/)に参加しました。

<!--more-->

以下メモです。

## 会場など

## オープニング

- リリースされたらしい。
- リリースノートからのパッケージのバージョンいろいろ
- 変更点いろいろ
- MariaDB → MySQL
- GnuPG
- デバッグシンボル向けの新しいアーカイブ: stretch-debug
- Xorg サーバーは root 権限が不要になった
- sysvinit だと X で問題がおきるらしい?
- upstart はなくなったらしい
- Perl 関連で問題が起きる可能性がある?

## さくらインターネット様からお知らせ

さくらの VPS やさくらのクラウドでは ISO イメージアップロードで使えますという話

## オープニング続き

- https://www.debian.org/News/2017/20170617
- リリースが確認できたので乾杯に移行

## デスクトップ環境の話

- インストーラーの途中でデスクトップ環境が選べるようになった
- リリースノートとインストールガイドは流し読みで良いのでみておくと良い
- UEFI は対応しているが、セキュアブート対応は見送られた
- プロプライエタリなファームウェアが必要なハードウェアの場合はフェームウェア入り非公式イメージを使うのが楽
- カーネルにおけるフリーと Debian のフリーが違うので、そういうもの (非公式イメージ) が存在する
- non-free が Debian 公式ではないのと同様の意味で非公式
- root パスワードを設定しなければ root を無効にして sudo を使うようにできる (以前からそうだった)
- netinst の iso でデスクトップ環境のみチェックして個別のデスクトップ環境を選ばなかった場合は GNOME になる
- 複数入れた場合にどうなるのかは未調査
- というわけで GNOME デスクトップの話
- GNOME 3 は初見だと使い方がわからない
- 今回は gnome-initial-setup パッケージが追加されたので案内が出るかと思ったら出ない?
- gnome-initial-setup パッケージを手動で入れてログアウトしてログインし直すと出る
- 初期設定の後、ヘルプが開く
- gnome-initial-setup で設定されていないと、キーボード設定が英語キーボードになっている
- フォントを入れる
- フォントを削除する
- fonts-droid-fallback が Android でのいわゆる中華フォントなので、完全削除すると良い
- Noto Serif CJK は backports に入るらしい
- ターミナルとか向けには migmix とか ricty とか
- 丸いフォントが好みでない人は fonts-vlgothic を消して IPA フォントを使うと良い
- ツッコミで fontconfig が難しい話
- 右上から開ける設定の他に Tweak Tool
- GNOME 拡張機能は JavaScript と CSS でできている
- 主要なものはパッケージで入れるのが良いのでは
- オススメ: gnome-shell-extension-dashtodock, gnome-shell-extension-top-icons-plus
- 会場から: Alt+F2 r Enter で gnome-shell が再起動する
- uim-toolbar-gtk3-systray が出てこない?
- 起動順序の問題で uim-toolbar-gtk3-systray の後に gnome-shell が起動するので認識されていない
- 回避策1: gnome-shell を再起動
- 回避策2: alternative で /bin/true にしておいて uim-toolbar-gtk3-systray は autostart でユーザーが起動する
- 回避策3: uim を諦める
- お好みで: gnome-shell-extension-move-dock, gnome-shell-extension-remove-dropdown-arrows, gnome-shell-extension-impatience, gnome-shell-extension-suspend-button
- パッケージ以外の拡張機能は GNOME Shell Extensions というサイトから
- ブラウザー拡張の chrome-gnome-shell でアップデートがブラウザー経由でできる
- Dash to Panel と Arc Menu で Windows 風にできる
- デスクトップにアイコンを表示して Nautilus のアイコンサイズを変更するとデスクトップのも一緒に変わる
- 隠し設定で切り離すこともできるらしい
- プロプライエタリなビデオドライバを使うなら contrib と non-free は必須
- backports の話
- software-properties-gtk で (synaptic から) 追加すると /etc/apt/trusted.gpg が壊れるのに昨日気づいた
- Firefox ESR の Accept-Language が en のまま
- 削除して登録し直すとなおる

追加で Debian T シャツの話

## Ryzen の話

- lurdan さん
- Ryzen の話
- [機械学習、データ解析なら 高火力コンピューティング | さくらインターネット](https://www.sakura.ad.jp/koukaryoku/)
- [ニュース解説 - グーグルもGPUクラウドに参入、4社のコスパ比較：ITpro](http://itpro.nikkeibp.co.jp/atcl/column/14/346926/022700857/)
- テラフロップスあたりの月額料金が安い
- 時間貸しなどの時は初期費用はいらないらしい
- Ubuntu インストールしたての状態なので、使うパッケージなどのインストール作業が必要
- [さくらの文教向けソリューション｜さくらインターネット](https://www.sakura.ad.jp/education/)
- Ryzen の話に戻り
- linux kernel 4.10 から対応コードが入っている
- その他の対応も考えると 4.11 以降が望ましい
- Proxmox は Debian のユーザーランドに Ubuntu zesty のカーネルなので、こういう用途の自宅サーバーには Proxmox VE が良いのではないか
- [#Ryzen_SEGV_Battle - Twitter検索](https://twitter.com/search?q=%23Ryzen_SEGV_Battle)

## Stretch リリース

- uwabami さん
- アップグレードの人柱の話
- 何台もあげたが特にはまらなかった
- リリースノート読み
- [第2章 Debian 9 の最新情報](https://www.debian.org/releases/stretch/amd64/release-notes/ch-whats-new.ja.html)
- [第5章 stretch で注意すべき点](https://www.debian.org/releases/stretch/amd64/release-notes/ch-information.ja.html)
- pass がおすすめ
- net-tools パッケージ (ifconfig など) が非推奨
- sl 的なものを設定するのが良いかも
- PIE: カーネルを更新しておかないとセグメンテーションフォルトになる可能性があるので jessie でもちゃんと 8.8 (以降) に更新してから stretch にあげ始める必要がある
- 一番のハマりどころになりそう
- セキュリティサポートの制限
- midori, konqueror などは完全なセキュリティサポートがないので Firefox や Chromium を使いましょう
- node.js はリソース不足で一切対応されない
- php とかコンパイラー対応がなくなった Chromium とか WordPress とか、セキュリティサポートがなくなった例は過去にもある
- [Security Bug Tracker](https://security-tracker.debian.org/tracker/)
- 「旧式の暗号と SSH1 プロトコルは OpenSSH では標準で無効にされています」
- evdev から libinput
- 「Perl での変更がサードパーティ製ソフトウェアを壊す可能性があります」
- カレントディレクトリが `@INC` からなくなる話
- ライブアップグレード
- jessie のまま最新に更新を確認
- apt line 書き換え
- apt-get update
- apt-get upgrade
- apt-get dist-upgrade
- [UnattendedUpgrades - Debian Wiki](https://wiki.debian.org/UnattendedUpgrades)
- 設定ファイルは選択に応じて `*.dpkg-old` とか `*.dpkg-dist` ができる
- apt autoremove
- reboot
- sysvinit にしていたので systemd に移行
- sysvinit がなくなったので sudo reboot は進むがコンソールに帰ってこなくなるので、処理が進んだ段階でブチっと切れて進む
- atig は bundle し直しで動いた
- znc も何か直したら動いた
- bitlbee は動いていなかった
- さくらインターネットさんで借りている VPS の stretch への upgrade の Live 実演終了

## LT

ここから LT タイム。

## 最新ハードウェアへのインストール

- 初めて Jessie をインストールしたときに起きたエラーについて
- 「ブートローダーのインストールに失敗しました。」
- 原因: GRUB が NVMe に対応していない
- 解決策1: NVMe 規格の SSD を買わない (おすすめは SATA)
- 解決策2: NVMe に対応しているブートローダーを使う
- NVMe に対応しているブートローダー: rEFInd

「ブートローダーのインストールに失敗しました。」というメッセージは見覚えがあったので、インストーラーで出たメッセージだとすぐにわかったけど、わからなかった人もいたようで、どんな状況で見たのか思い出そうとしたけど、思い出せなかったので、仮想環境か何かで特殊なことをしていて出ただけで困らなかったのかもしれない、と思った。

## yabuki さんの話

自分の準備中で聞けず。

## Debian での OpenSSH の TCP wrappers サポート

なぜか HDMI を接続しても反応しなかったので、PDF にして、さとうさんの PC を借りて発表しました。

発表資料はいつも通り[github](https://github.com/znz/openssh-on-debian9), [Rabbit Slide Show](https://slide.rabbit-shocker.org/authors/znz/openssh-on-debian9/), [slideshare](https://www.slideshare.net/znzjp/stretchopensshtcp-wrappers), [Speaker Deck](https://speakerdeck.com/znz/stretchdefalseopensshfalsetcp-wrapperssapoto), [RubyGems](https://rubygems.org/gems/rabbit-slide-znz-openssh-on-debian9) にあげています。

<iframe src="https://slide.rabbit-shocker.org/authors/znz/openssh-on-debian9/viewer.html"
        width="640" height="524"
        frameborder="0"
        marginwidth="0"
        marginheight="0"
        scrolling="no"
        style="border: 1px solid #ccc; border-width: 1px 1px 0; margin-bottom: 5px"
        allowfullscreen> </iframe>
<div style="margin-bottom: 5px">
  <a href="https://slide.rabbit-shocker.org/authors/znz/openssh-on-debian9/" title="stretchでのOpenSSHのTCP wrappersサポート">stretchでのOpenSSHのTCP wrappersサポート</a>
</div>

## T シャツの話

欲しい人は OSC 京都や勉強会で、または @nogajun さんに直接連絡

## 告知

- 次回は 7月はなしで、代わりに[オープンソースカンファレンス2017 Kyoto](https://www.ospn.jp/osc2017-kyoto/)の8月5日で。
- [KOF](https://k-of.jp/) (今年のサイトはまだない)
- [姫路IT系勉強会](https://histudy.connpass.com/)の8月が今回と同じさくらインターネットさんが会場

## 感想

リリースは twitter などではリリースされたっぽい感じでしたが、たぶん最後のアナウンスっぽい[リリースアナウンスのメール](https://lists.debian.org/debian-announce/2017/msg00003.html)が「Sat, 17 Jun 2017 20:22:36 -1000」つまり日本時間だと「2017-06-18 15:22:36 +0900」で、少なくともリリースパーティー中にはリリースされていたようです。

すでにあげた人の話では、特に大きなトラブルもなさそうなので、安心してあげられそうです。
