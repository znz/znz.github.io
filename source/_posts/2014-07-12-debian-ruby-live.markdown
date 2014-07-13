---
layout: post
title: "Debian 7 (wheezy) の RubyLive をカスタマイズ中"
date: 2014-07-12 10:08:26 +0900
comments: true
categories: debian live ruby rails
---
今年の
[KOF 2014：関西オープンフォーラム2014](https://k-of.jp/2014/ "KOF 2014：関西オープンフォーラム2014")
に向けて
[RubyLive](https://github.com/znz/rubylive "RubyLive")
を作成しています。
fork 元の [no6v 版](https://github.com/no6v/rubylive) との違いをまとめていきます。

<!--more-->

## カスタマイズの基本

README から今回に関係する部分を引用しておくと以下のようになっています。

- config/hooks/
  - インストールの最後の方で実行するフックスクリプトを置くディレクトリ。
  - 拡張子を .chroot にして実行権限を付けておく。
- config/includes.chroot/
  - このディレクトリを root に見立てて LiveCD 環境にコピーしたいファイルを置くディレクトリ。
- config/package-lists/
  - 特別にインストールしたいパッケージのリストを置くディレクトリ。
  - 拡張子を .list.chroot にしてパッケージ名を列挙する。

## 壁紙などの変更

`resources.yml` で設定されたファイルは `url` から自動ダウンロードして `path` に置くようになっています。

`size` と `sha256sum` をチェックするだけで違っていても削除はしないようなので、
ダウンロードに失敗した時は`path` のファイルは手動で削除する必要がありました。

`path` を変更した時も古いファイルが残ってしまうので、削除する必要がありました。

## dconf の設定

`config/includes.chroot/etc/skel/.config/dconf/user`
に設定変更後のバイナリが置かれていて、
これはひどいと思ったので、
`config/includes.chroot/etc/skel/.gnomerc`
で `gsettings set` を使って設定するようにしました。

### 壁紙の変更

起動後の `dconf-editor` で選択肢を確認しつつ、
`gsettings set org.gnome.desktop.background picture-options centered`
にしたり、
`gsettings set org.gnome.desktop.background picture-uri 'file:///usr/share/images/desktop-base/RubyKaigi2014-commonLogo.svg`
にしたりしました。

2014-07-13 追記:
生成後のイメージに CC-BY 3.0 の説明がないのは良くないと思って、
`config/includes.chroot/etc/skel/README.txt`
に説明を追加することにしました。

### デスクトップのアイコン

以前は `gnome-panel` (上のバーのところ) に起動用のアイコンを追加していたようですが、
`gsettings set` で設定しようとすると
`org.gnome.gnome-panel.layout object-id-list` の他に
`org.gnome.gnome-panel.layout.objects.object-0` や
`org.gnome.gnome-panel.layout.objects.object-0.instance-config` などの
複数設定が必要で管理の手間もかかりそうだったので、
`gsettings set org.gnome.desktop.background show-desktop-icons true`
でデスクトップのアイコンが見えるように変更しました。

### スクリーンセーバーの停止

`gsettings set org.gnome.desktop.screensaver idle-activation-enabled false`
で止めました。

## chm の変更

[Rubyリファレンスマニュアル chm版リミックス](http://ruby.morphball.net/refm-remix.html "Rubyリファレンスマニュアル chm版リミックス")
の標準テーマのRuby 2.1.0向け chm に差し替えました。
zip ファイルなので、先ほどの `.gnomerc` でデスクトップに展開するようにしました。

xCHM v. 1.20 で背景画像や色とかがつかないようなので、サイズが小さい標準を選びました。

## パッケージ変更

`jfbterm` の代わりに `fbterm` にしたり、
`ruby-build` でビルドに必要なパッケージを追加したり、
`config/includes.chroot/etc/iceweasel/profile/prefs.js` の代わりに `iceweasel-l10n-ja` を追加したり、
`bash-completion` などを追加したりしました。

## ruby-build で ruby 2.1.2 のインストール

[[ReVIEW Tips] DockerでRe:VIEW - Qiita](http://qiita.com/takahashim/items/406421d515ef1d4f1189 "[ReVIEW Tips] DockerでRe:VIEW - Qiita")
を参考にして rbenv は使わずに ruby-build だけ使って `/usr/local` に ruby 2.1.2 をインストールしました。

## localepurge

locale の設定は live-config で起動時にやっていて hook の中で
`DEBIAN_FRONTEND=noninteractive dpkg-reconfigure localepurge`
としても起動後と違って ja locale の設定がなかったので、
起動後や設定した後に `debconf-show localepurge` で確認した値を使って、
`localepurge/nopurge` が `NEEDSCONFIGFIRST` のままなら
`echo localepurge localepurge/nopurge string "ja, ja_JP.UTF-8" | debconf-set-selections`
で設定することにしました。

これで約 200MB ぐらい小さくなりました。
