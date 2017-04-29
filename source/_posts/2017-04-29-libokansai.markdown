---
layout: post
title: "第14回関西LibreOffice勉強会に参加しました"
date: 2017-04-29 13:00:00 +0900
comments: true
categories: event libokansai
---
[第14回関西LibreOffice勉強会](https://connpass.com/event/53960/ "第14回関西LibreOffice勉強会")に参加しました。

<!--more-->

以下、メモです。

## 会場など

- 地図ソフトで三国駅から歩くルートも表示されたので、早めに出発して時間もあったので、歩いてみました。(帰りは懇親会には参加しなかったので、十三駅まで歩いてちょっと梅田に出てから帰りました。)
- セキュリティがかかって閉まっていてオフィスフロアに入れなくて、しばらく待っていました。
- 普通の土曜日なら開いているのに、祝日だから閉まっていたようです。
- 電源やネットワークはなし
- 飲食は可能

## 自己紹介

自分はたぶん初参加でした。

久しぶりに参加という人も何人かいて、長く続いている勉強会という感じを受けました。

## 書式の自由が社会を変える―LibreOfficeとPandocができること

- 発表資料: https://github.com/sky-y/libreoffice-kansai-14-pandoc
- Pandoc の日本語ドキュメントは古くなってしまっているので、英語が読めるなら本家の英語を推奨

- この発表の概要
- Pandoc の概要
- Pandoc をインストールする
- Pandoc でドキュメントを変換する (LibreOffice Writer 文書を中心に)
- 「書式の自由」について

- 対応フォーマットが多い
- ODT などは入力にも対応
- Markdown とは

- Pandoc でできないこと
- スプレッドシートは扱えない
- 簡単な表は対応している
- LibreOffice Impress には対応してない
- LaTeX Beamer/HTML プレゼンには変換可能

- Pandoc を使う心得
- 過剰な期待をし過ぎないこと
- Pandoc は万能でないし、文書仕様の全てを満たしているわけではない
- 補助的に使うのがベスト

- Pandoc の実装は Haskell

- 補足: Markdownと標準仕様
- RFC で Media Typeにて「Markdownであること」と「方言の名前」を明示する方法を定めた
- (RFC でうまくいかなかったというと Cookie を思い出した)

- 質疑応答
- 文字コードは UTF-8 で

- Pandoc をインストールする
- wkhtmltopdf の wk は WebKit らしい
- 動作確認
- `echo "http://localhost" | pandoc -f markdown_github -t html` のように `markdown_github` だと自動リンクがある
- `echo "**Hello**" | pandoc -f markdown -t html5 -o hello.pdf`

- おまけ: Pandocで作れるスライド
- 今回は「reveal.js」形式に変換
- LaTeX Beamer など他のプレゼン形式にも変換できる

- Pandoc でドキュメントを変換する
- https://github.com/sky-y/libreoffice-kansai-14-pandoc の sample で変換を試す
- Windows の start コマンド、GNOME の gnome-open や macOS の open に相当

- テンプレート
- `pandoc --print-default-data-file reference.odt > reference.odt`
- 左の `reference.odt` は pandoc 内部のテンプレートディレクトリの中のファイル名を指定している
- 右は出力ファイル名なので紛らわしいが別物
- スタイルで 源ノ角ゴシック (げんの かくごしっく) に変えるとか

- 画像に関するノウハウ
- 96dpi よりも 300dpi の方が良いという指摘あり https://twitter.com/nogajun/status/858193042309861377

- 「書式の自由」について

- 質疑応答

## 休憩

10分間休憩

## Office文書を手打ちハックする ～FlatODF活用のすすめ～

- JO3EMC さん
- 文書の自動生成をやりたい
- 文書を細部まで思い通りにコントロールしたい
- アプローチ例1: LibreOffice 以外のツール・言語を利用する
- アプローチ例2: LibreOffice 関連のツールを利用する
- FlatODF
- 単一の圧縮されていない XML ファイル
- 弱点
- ファイルサイズが大きくなる
- 圧縮されていないので
- あとで ODF に変換すれば良い
- 画像やオブジェクトの埋め込みは少し面倒
- Base64
- fodt で保存してテキストエディタで書き換えて開き直すデモ
- UTF-8 以外には対応していないらしい
- LibreOffice を使って他フォーマットへ変換・印刷
- `soffice --headless --convert-to pdf *.fodt`
- `soffice -p *.fodt`
- 活用例
- FlatODF の構造の概略
- 質疑応答
- zip された中の content.xml をいじる方法も FlatODF をいじる方法もそれぞれ長所や短所があるのでいろんな方法があるのは良いんじゃないかという話

## 休憩

- 時間がおしているので5分間休憩
- 休憩前に懇親会参加者確認

## LibreOffice Online環境の構築

- LibreOffice Online (LOOL)
- LOOL (ろーる)
- CentOS 7.3.1611 で環境構築
- LibreOffice の make に時間がかかる (一晩?)
- 依存をいろいろ入れる
- 日本語フォントも別途入れる必要あり
- LibreOffice Online のコンパイル (5〜10分ぐらい?)
- LOOL はファイルのインプットの GUI がない
- Nextcloud と連携
- サーバー側でレンダリングして画面を転送しているので、サーバー側にフォントが必要
- https://librepc.jp/

## LT

- 矢吹さんの話
- 会場アンケート
- スプレッドシートを使ったことがある人
- SQL を使ったことがある人
- 領収書を集計するのが面倒だった話
- バベルの塔
- ピボットテーブル = GROUP BY を知るのに時間がかかった話
- 相手の文化を知る必要がある
- 用語集の必要性
- 抽象度の上げ下げ

## ディスカッション

- 来年 LibreOffice 6 になる
- デザインを一新するのでデザイナーを募集している
- [New branding for LibreOffice 6.0 - LibreOffice Design Team](https://design.blog.documentfoundation.org/2017/04/21/new-branding-libreoffice-6-0/)
- [BRANDING FOR LIBREOFFICE 6.0](http://opensourcedesign.net/jobs/jobs/2017-04-20-branding-for-libreoffice-60)

- [Please participate in a survey about table styles - LibreOffice Design Team](https://design.blog.documentfoundation.org/2017/04/27/table-styles-survey/)
- 表スタイルというのが追加された
- デザインのアンケートを実施中

- [LibreOffice Calcのスプレッドシートの変更点をgit diffで見られるようにする - ククログ(2017-04-24)](http://www.clear-code.com/blog/2017/4/24.html "LibreOffice Calcのスプレッドシートの変更点をgit diffで見られるようにする - ククログ(2017-04-24)")

- 本?

- HackFest

- バグハンティング・セッション

- 源ノ明朝
- IPA フォントはデザインなどが古い
- 源ノ角ゴシック 源ノ明朝 と Noto はパッケージの違い
- CJK にも対応しているのが良い
- デザインも今風
- LibreOffice 5.2 だと縦書きが変? 5.3 だと太字しか出ない?

- テストの話
- 普通のテスターが簡単に使える Selenium のような自動化がないのがつらい
- xautomation ?

- デザインチームなどの話

- ドキュメントの翻訳の話
- OmegaT は odt に直接対応している

## ふりかえり

- 抽象度の話がよかった
- LibreOffice を直接触る話はなかった
- 前処理は好き勝手にできるというのは良い
- 表現とデータをわけると嬉しいと思ってくれる人が増えると嬉しい
