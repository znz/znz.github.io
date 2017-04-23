---
layout: post
title: "第 122 回関西 Debian 勉強会に参加しました"
date: 2017-04-23 13:30:00 +0900
comments: true
categories: debian event
---
[第 122 回関西 Debian 勉強会](https://debianjp.connpass.com/event/54424/ "第 122 回関西 Debian 勉強会")に参加しました。

<!--more-->

以下メモです。

## 会場など

早めに出発したら時間があったので、駅前ビルの金券屋で切符を買って、少し安く移動できました。

しばらく前に阪急三番街の KIDDY LAND で stretch のぬいぐるみが 2,3 個あったのを見かけて、次にみたときには最後の 1 個になったいたので、買っておいたのを持って行きました。

## オープニング

- 前回の話から会場候補地の話とか
- https://wiki.debian.org/KansaiDebianMeeting/DraftMemo に古いメモがある
- [Open Source Summit Japan 2017](http://events.linuxfoundation.jp/events/open-source-summit-japan "Open Source Summit Japan 2017") というのがあるらしい
- [Status on the stretch release](https://lists.debian.org/debian-devel-announce/2017/04/msg00008.html "Status on the stretch release")
- [Statement concerning the arrest of Dmitry Bogatov](https://www.debian.org/News/2017/20170417 "Statement concerning the arrest of Dmitry Bogatov")
- [Statement regarding Dmitry Bogatov | The Tor Blog](https://blog.torproject.org/blog/statement-regarding-dmitry-bogatov "Statement regarding Dmitry Bogatov | The Tor Blog")
- 事前課題
- maven とか make とか

## 休憩

## CMake でビルド

- Windows 版と Linux 版の両対応が動機
- Visual Studio や Eclipse のプロジェクト出力も可能
- https://github.com/yosukesan/kansai_debian に今回のサンプルを用意
- `distclean` 相当がないので build ディレクトリを作る方が良い
- `cd 000.hello; mkdir build; cmake ../ -DCMAKE_INSTALL_PREFIX=.; make; make install`
- `CMakeCache.txt` が `configure.log` 相当
- `CMakeCache.txt` を編集することも可能
- `cmake ..` ではなく `cmake ../CMakeLists.txt` としてしまうと build ディレクトリではなくソースディレクトリにファイルが作られてしまうので注意
- ライブラリをリンクする例: 失敗する例が `001_NG.link_library` で成功する例が `001_OK.link_library`
- 自作ライブラリのビルドとリンク
- `CMakeCache.txt` に入る変数と入らない変数がある
- 設定するのに `FORCE` オプションが必要なものとなくても良いものがあってハマった
- echo しても空なのに、内部的には変数がある
- 変数の上書きに癖があってハマった
- Windows でマルチスレッドかどうか、デバッグかリリースかなどでリンクするライブラリが違うのが自動でできなかった
- Visual Studio でもリンクするライブラリの組み合わせ問題ははまることがあるらしい
- Dependency Walker
- デバッグビルドの DLL が混ざっていてバグっていた話
- https://ninja-build.org/manual.html

## その後

時間が余ったので、次回の予定などの話をしていました。
