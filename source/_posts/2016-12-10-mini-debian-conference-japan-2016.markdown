---
layout: post
title: "Mini Debian Conference Japan 2016に参加して発表してきた"
date: 2016-12-10 10:30:00 +0900
comments: true
categories: debian event letsencrypt
---
[Mini Debian Conference Japan 2016](http://miniconf.debian.or.jp/ "Mini Debian Conference Japan 2016")
に参加して、発表してきました。

<!--more-->

## 会場

前日に東京に移動していたので、余裕がありましたが、
東京駅から近いので、当日移動でも頑張れば間に合いそうな場所でした。

ネットワークはゲスト用の無線があって、接続すると Web 画面が出てきてそこでユーザーとパスワードを入れて認証するという方式でした。

## セッション

2トラックなので、どちらを聞きに行くのか悩む必要がありました。
さらに午後からは LibreOffice Kaigi 2016.12 も併催なので、さらに悩みました。

## オープニング

- DebConf を日本でやりたいので、カンファレンス開催のノウハウをためたい
- 諸注意で Windows の画面が出てきてブーイング
- 結局映らなかったので口頭で
- 自動販売機は使用禁止とか
- 休憩中の予定説明とか
- さっき出てきた DebConf の[写真](https://twitter.com/yasulab/status/807399231107584001)のような集合写真撮影があるよとか
- 機材の関係で最初のセッションは部屋を入れ替え

## Open Build Service in Debian

- Open Build Service のアーキテクチャの説明
- フロントエンドは Rails
- https://goo.gl/OSBNqv
- https://goo.gl/2rNPMx
- デモはディスクフルで終了

## 昼食及びPGP/GPGキーサインパーティ

パスポートを机の上に準備していたのに持ってくるのを忘れてしまっていて、
ID が運転免許証しかなかったので、日本人とだけにしておきました。

## OSS license 101

- ライセンスは一部の権利を許諾するもの
- 著作権の他に特許、商標、契約も関係する
- 商標の例: "Firefox" と Iceweasel
- 契約の例: Red Hat エンタープライズ契約書
- 「5.2 検査。」という項目がある
- ライセンスを選ぶ
- 目的に合わせて
- 万能のライセンスはない
- ライセンスを独自に作るのはよくない
- OSS ライセンスは well-tested library
- 独自ライセンスは使うときに吟味が必要になるし、互換性も問題になる
- コードを書きたい人は既存のライセンスを使ってコードを書いていた方が生産性が高い
- Proprietary license vs OSS license
- Default deny vs Default allow
- Whitelist vs Blacklist
- The Open Source Definition (Annotated) https://opensource.org/osd-annotated/
- DFSG-free (OSS), OSI-Certified and fake-OSS
- Well-known OSS license
- どのライセンスが良いか?
- 目的と利用方法によって変わる
- Copyleft vs Permissive https://www.gnu.org/licenses/copyleft.ja.html
- patent-free or not
- Domain-specific
- OFL,CC,GFDL, etc.
- 残りの時間はライセンスがらみの雑談
- Zstd https://github.com/facebook/zstd
- BSD-3-clause license However, its "PATENTS" file says
- Zfs (GPL vs CDDL) by Canonical, Ltd.
- GPL: Linux "T-800" issue
- 第三者はソースコードを請求できない
- 「Linux で稼働しているターミネーターを掴まえたとしても、そのバイナリの所有権を得たわけではないので、ソースコードは請求できない。」 https://twitter.com/elim/status/807442658830336001

## Go言語で書かれたソフトウェアをDebianパッケージにする方法

- https://twitter.com/tSU_RooT
- GPG ID: 63A6 000E
- peco の Debian パッケージを入れた人
- dh-make-golang
- 佐々木さんは自分用パッケージを作ったが、メンテナンスするプログラミング言語を増やしたくなかったので公式にはあげなかったらしい
- 公式に入れるとメリットが多い
- 公式に入ったっときのデメリットはパッケージメンテナがアップデートに追随してくれないことがある
- パッケージに限らない問題
- 下準備
- sid の環境を用意
- リポジトリの確認
- ソースコードからビルドできるか
- ライセンスが付属しているか
- go get するだけでビルドできるか
- 依存ライブラリがすでにパッケージになっているか
- 依存ライブラリも同じチェック
- ライセンスがない場合: issue でお願いする
- 複雑なビルド手順が必要な場合: debian/rules で頑張る必要がある、今回は対象外
- 依存ライブラリが多い場合
- ライセンス確認
- 例えば、サンプルに Gopher くんの画像がついていたら debian/copyright に明記する必要あり
- pkg-go.alioth.debian.org
- パッケージの命名規則がある
- fork したリポジトリも別パッケージで問題ない
- Go 1.6 で正式導入された vendor ディレクトリの扱いはまだ完全には決まってないっぽい
- peco (v0.4.2) での実例
- 依存パッケージの話
- `apt showsrc golang-go-flags-dev 2>/dev/null | grep Homepage` で upstream を確認
- 古いパッケージで依存なしで消えていたものを復活させた
- lintian の警告を消す
- debian/copyright を書く
- debian/changelog を直す (ITP の番号を埋める、UNRELEASED を unstable に)
- debian/control の README から自動で生成された説明文を直す
- ライブラリパッケージすべてに行う
- バイナリパッケージは man ページも用意する
- 今回話せなかったこと
- 参考資料
- 質疑応答
- pristine-tar と git-buildpackage の話

## Certbotで無料TLSサーバー

Certbotで無料TLSサーバーというタイトルで発表しました。

<iframe src="https://slide.rabbit-shocker.org/authors/znz/debian-certbot/viewer.html"
        width="640" height="524"
        frameborder="0"
        marginwidth="0"
        marginheight="0"
        scrolling="no"
        style="border: 1px solid #ccc; border-width: 1px 1px 0; margin-bottom: 5px"
        allowfullscreen> </iframe>
<div style="margin-bottom: 5px">
  <a href="https://slide.rabbit-shocker.org/authors/znz/debian-certbot/" title="Certbotで無料TLSサーバー">Certbotで無料TLSサーバー</a>
</div>

https://github.com/sorah/acmesmith というクライアントもあるらしい。

https://github.com/dokku/dokku-letsencrypt で使っている `simp_le` は 開発が止まっている ( https://github.com/kuba/simp_le/issues/114 )。

## 休憩及び集合写真撮影

集合写真を撮影して、おやつ休憩がありました。

## FOSS バーチャルシンガー 徴音梅林 と LINNE プラットホーム

- 英語なので頑張って聞いていました。
- http://projectmeilin.github.io/ja/

## 最近のGnuPG

- 二ヶ国語でプレゼン
- メモリ不足でプレゼンツールがうまく動かないので佐々木さんのマシンに切り替え
- Jessie は gnupg パッケージは 1.4 (新しいバージョンは gnupg2 パッケージ)
- Stretch は gnupg パッケージは 2.1 (古いバージョンは gnupg1 パッケージ)
- GnuPG 2.1?
- 公開鍵のフォーマットが KBX に変わった。(昔の形式もサポート)
- プライベート鍵は gpg-agent が管理するようになった。
- gpg, gpg-agent, pinentry, scdaemon, dirmngr, (gpgsm, ssh)
- おすすめの使い方
- gpg-agent を ssh-agent として使う
- Token を使う
- Curve25519 を使う (Ed25519/X25519 is more secure, key is small, fast)
- キーサインパーティー
- WKD: Web key directory
- ToFU: Trust On First Use
- g13 + dm-crypt
- 質疑応答
- RSA 鍵からの移行
- サブキーの追加よりも新規に作るのがおすすめ
- gnuk の話
- curve25519 サポートしている
- 楕円曲線暗号は輸出入の規制にひっかかることがある

## 休憩

2つの部屋をくっつけて広くなった。

## DebConf 2018 台湾 参加表明準備とステータスの更新

- 英語なので twitter の `#debianjp` を参考にして頑張って聞いていました。

## クロージング

- アンケート: https://goo.gl/BsPrgA

## 懇親会

- LT 大会をやっていました。
- 全体的にマイクの通りが悪いのか、英語に限らず話が聞き取りにくかったです。
- Unicode の [漢字構成記述文字列 (Ideographic Description Sequence; IDS)](https://ja.wikipedia.org/wiki/%E6%BC%A2%E5%AD%97%E8%A8%98%E8%BF%B0%E8%A8%80%E8%AA%9E#.E6.BC.A2.E5.AD.97.E6.A7.8B.E6.88.90.E8.A8.98.E8.BF.B0.E6.96.87.E5.AD.97.E5.88.97_.28IDS.29 "漢字構成記述文字列 (Ideographic Description Sequence; IDS)") ですごい漢字を出しているのとか自作 OS の話とかが印象に残りました。
- 2回あった自動販売機の話もなかなか面白かったです。 https://twitter.com/OrientalHistory/status/807530627293593600 https://twitter.com/OrientalHistory/status/807535513779322880
- セッション中は結局 LibreOffice Kaigi 2016.12 の方はいけなかったが、 LibreOffice 側にいた人とも話ができてよかった。
- 昔の RubyKaigi でセッションがのびたのに別の部屋の次のセッションは始まってしまって、両方見たかった人が最初の方を見逃すということがあって、その後、別の部屋とも同期をとるようになったということがあったのを思い出したのですが、別イベントなので、そこまで同期を取る必要はないにしても、せめてセッションの開始終了予定時刻が同じくらいの時間になっていれば、相互に行き来が発生しやすかったのではないかと思いました。
- この話は LibreOffice 側の人にしたのですが、もともとイベントの企画は別々に始まっていて、たまたま会場と日付が一緒だったから合同にしたという流れだったようで、そこまで考えていなかったということのようでした。

## まとめ

k-of.jp で発表者募集を見て応募するまで参加する予定は全くなかったので、
前回会ったのがいつだったか忘れるぐらい久しぶりに会った人もいて、
全体としては楽しかったので、
参加して良かったと思いました。
