---
layout: post
title: "【5/28大阪】データ分析用次世代データフォーマットApache Arrow勉強会に参加しました"
date: 2017-05-28 10:00:00 +0900
comments: true
categories: event
---
[【5/28大阪】データ分析用次世代データフォーマットApache Arrow勉強会](https://classmethod.connpass.com/event/56478/ "【5/28大阪】データ分析用次世代データフォーマットApache Arrow勉強会")に参加しました。

<!--more-->

以下、メモです。

## 会場

早めに到着していたのですが、1階の入り口があいていなくて、案内してもらうまではいれませんでした。

## ハッシュタグ

`#osaka_arrow`

## 会場アンケート

- 普段使ってる言語
- データ分析をしているか
- 使っているならツールは?

使ってる言語としては Python や Ruby や C# が多い?
データ分析をしていない人も多かったけど、している人はデータ分析には Python や R が多い?

## スライドなど

- 今回のメインのスライドは (まだ?) 公開されていないっぽい? (https://slide.rabbit-shocker.org/ にはなかった)
- Arrow については [RubyもApache Arrowでデータ処理言語の仲間入り](https://slide.rabbit-shocker.org/authors/kou/data-science-rb/) を使って説明 (Apache Arrow とは何なのかがわかるので、一読をオススメします。)
- 今回は Apache Arrow がメインなので Ruby 関連のところは飛ばしていた。
- 以下の関連資料の URL は `#osaka_arrow` でツイートしてから飛ばしつつ説明していました。
- [Next-generation Python Big Data Tools, powered by Apache Arrow](https://www.slideshare.net/wesm/nextgeneration-python-big-data-tools-powered-by-apache-arrow)
- [Memory Interoperability in Analytics and Machine Learning](https://www.slideshare.net/wesm/memory-interoperability-in-analytics-and-machine-learning)
- `#osaka_arrow` でWesさん(Arrow のメイン開発者?)のblogの翻訳をしている方が以下の翻訳の URL をツイートしていました。
- [（翻訳）2017年の展望: pandas, Arrow, Feather, Parquet, Spark, Ibis](http://qiita.com/tamagawa-ryuji/items/deb3f63ed4c7c8065e81)
- [（翻訳）毎秒10GBでArrowからpandasへ](http://qiita.com/tamagawa-ryuji/items/9ba22061fc78907a5826)

## メモ

- Feather は R と Python の間だけ用の Arrow のようなもの
- 作っている人が同じで Feather での知見が Arrow に生かされている
- Parquet は保存用でデータサイズを小さくすることを重視
- Parquet は無圧縮もできて、それだとサイズが大きくなることがある
- Arrow や Parquet は特定の列だけ読むとかもできるので、サイズが同じでも処理効率がよくなることがある
- 多次元配列 (テンソル) は中身が同じ型で、そういう用途向けに最適化されている

- Wes McKinney さん: pandas を作った人でその知見が Arrow にも生かされている
- Hadoop のディストリビューター

- SlideShare で apache arrow で検索すると色々資料がみつかる

- zero-copy が大事
- in memory が前提としてある
- メモリレイアウトや record batch とかもその関連
- メモリに収まるような record batch のサイズ指定は API で手動設定
- Arrow は基本的に read only
- 元データはアプリケーション次第
- IPC: 同じマシンなら mmap とか

- PySpark だと JVM と Python とのやりとりが重い
- 個別にチューニングするのは無駄なので Arrow でみんなで共通のチューニングをする
- Ruby のオブジェクトにすると変換すると負けなので、高速に処理したい場合は Arrow の世界で演算も済ませる必要がある

- streaming もある

- 開発に参加しようという話
- https://issues.apache.org/jira/browse/ARROW/
- https://red-data-tools.github.io/

- 質問タイム
- fluentd で message pack をパースしてルーティングの情報だけ読んでまた message pack にシリアライズして、だと読んでない部分のパースとシリアライズが無駄なので、そこを arrow で置き換えると改善できないかなあという話
- Red Data Tools の名前の由来: Ruby に限定したくなかったので redmine などで使われている red と PyData などの Data を組み合わせて、 red data だけだと絶滅危惧種などの意味とぶつかるので、何が良いか悩んで最終的に tools をつけた。
- データベースとの絡み

## まとめ

基本的には開発者としての参加をしやすくするための勉強会でした。
Apache Arrow 自体でデータ分析ができるようになるわけではなく、ツールを作るための共通基盤という感じでした。
