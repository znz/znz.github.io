---
layout: post
title: "LILO&東海道らぐオフラインミーティング 2016/01/16 に参加しました"
date: 2016-01-16 13:00:34 +0900
comments: true
categories: lilo event linux
---
[LILO&amp;東海道らぐオフラインミーティング 2016/01/16 - LILO | Doorkeeper](https://lilo.doorkeeper.jp/events/36903 "LILO&amp;東海道らぐオフラインミーティング 2016/01/16 - LILO | Doorkeeper")
に参加しました。

今回もアンカンファレンス形式でした。

<!--more-->

## メモ

今回のメモです。

- ハッシュタグ: `#lilo_jp` `#東海道らぐ`
- 登録参加者数 10名
- 自己紹介から
- Chromecast を持ってきている人がいてプロジェクターにさしていた。
- Kapper さん : ARM Linux、Android、RaspberryPi で Windows とアプリをX86エミュで動かそう
- ARM は x86 に比べて浮動小数点演算が遅い
- ExaGear Desktop という商用のエミュレータもあるらしい
- qemu より速い
- DOSBox, Bochs は遅いが移植性が高い
- qemu-user-static と schroot で Wine などを動かしている話
- Debian は mutlilib をサポートしているが multi-binary はサポートしていないので `apt-get *:i386` で i386 のバイナリで上書きされてしまわないように schroot で環境を分けているという話
- プレゼンに使った環境は Ubuntu を追加で入れた Chromebook
- `uname` は `armv7l`
- Crouton
- 派生して Windows 上で仮想環境の話
- Virtual PC, Hyper-V, VirtualBox, VMware Player
- 山内さん : Raspberry Pi で遊んだ後は Piface を買ってホームエレクトロニクスとホームセキュリティで実用しよう
- Tocos 無線DIO
- TWE-Lite DIP (トワイライト・ディップ)
- [epicon](https://osdn.jp/projects/pepolinux/wiki/epicon) でシリアル通信
- 休憩
- 榎さん : LibreOffice の最近の動向と Debian での LibreOffice パッケージについて
- LibreOffice Online (LOOL)
- ownCloud の編集画面で LOOL を使うデモがある
- VirtualBox のイメージが用意されているので簡単に試せる
- LibreOffice Viewer for Android
- Advisory Board
- ヨーロッパでは行政中心に導入が進行中
- 日本での活動
- UI の翻訳率は高い
- Help などは追いついていない
- イベント
- HackFest
- LibreOffice Conference 2015 への日本からの参加者は 3 名
- 各言語のコミュニティ : LibreItalia (非営利団体), ベトナムコミュニティ
- [ヨーロッパでのLibreOffice活用は移行期から安定期に、アジアも活発な動き](http://itpro.nikkeibp.co.jp/atcl/column/15/102800252/102800003/ "ヨーロッパでのLibreOffice活用は移行期から安定期に、アジアも活発な動き")
- LibreOffice mini Conference 2016 in Japan
- 現時点での参加者が 13 名に増えている話
- Debian での LibreOffice パッケージ
- 遅れてきた人の自己紹介
- Shimada Hirofumi さん : opencocon
- 新ビルドサーバ
- Allwinner タブレットの OS を作ってみる (途中)
- linux-sunxi コミュニティ
- http://linux-sunxi.org/Identification_guide
- http://linux-sunxi.org/GPL_Violations
- Android 上だと型番などの情報がわからないので、バラしてプロセッサなどを確認
- http://linux-sunxi.org/Format_Q8
- OpenEmbedded + meta-sunxi
- https://github.com/linux-sunxi/meta-sunxi
- Device Tree, FEX
- sunxi-tools
- microUSB で転送
- 現状はカーネルが起動するところまで
- デモ
- まとめ
- 上周りはどうするのかが問題
- 選択肢のひとつ : http://plasma-mobile.org/technology/
- 休憩
- 矢吹さん : Sphinx + VOoM
- [VOoM : Vim two-pane outliner](http://www.vim.org/scripts/script.php?script_id=2657 "VOoM : Vim two-pane outliner")
- [VOoM: two-pane text outliner](http://vim-voom.github.io/ "VOoM: two-pane text outliner")
- [Sphinx-Users.jp — Python製ドキュメンテーションビルダー、Sphinxの日本ユーザ会](http://sphinx-users.jp/ "Sphinx-Users.jp — Python製ドキュメンテーションビルダー、Sphinxの日本ユーザ会")
- <a rel="nofollow" href="http://www.amazon.co.jp/gp/product/4478490279/ref=as_li_ss_tl?ie=UTF8&amp;camp=247&amp;creative=7399&amp;creativeASIN=4478490279&amp;linkCode=as2&amp;tag=znz-22">考える技術・書く技術―問題解決力を伸ばすピラミッド原則</a><img src="http://ir-jp.amazon-adsystem.com/e/ir?t=znz-22&amp;l=as2&amp;o=9&amp;a=4478490279" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />
- https://packages.debian.org/sid/vim-voom
- すがさん : [システム奮闘記](http://www.geocities.jp/sugachan1973/doc/funtouki.html "システム奮闘記") 進捗
- LAN の減衰、表皮効果
- さとうさん : Raspberry Pi と魚眼レンズのカメラなどの実演
- 丸市 展之さん : OSM 3D マップの話
- 榎さん : 最後にちょっとだけ ownCloud + LibreOffice Online のデモ
