---
layout: post
title: "ブラウザのみで使えるバーコードリーダーを作った"
date: 2013-10-20 19:27
comments: true
categories: html5 javascript
---
最近の HTML5 の getUserMedia API でカメラの画像をブラウザ上で処理できるようになったということで、
専用のバーコードリーダー機器を買ったり、
モバイル機器で専用のアプリを入れたりしなくても、
ノートPCの内蔵のカメラでも使えるバーコードリーダーを作ってみました。

<!--more-->

## 使い方

[Web Barcode Reader](http://web-barcode-reader.herokuapp.com/)
を開いて、
既にPC上に画像がある場合はその画像を選択してください。
選択するだけでバーコードの認識が始まります。
iOS や Android などではファイル選択かカメラで撮影かを選べます。

[BarcodeReader](https://github.com/EddieLa/BarcodeReader)
のライブラリを使って
JavaScript のみで認識していて、
バーコードの認識は初回はちょっと時間がかかるようなので、
ゆっくり待ってください。

Google Chrome などではファイル選択の横に
「ブラウザの中でカメラ画像を取得」
ボタンが出るので、
そのボタンを押した後、
カメラへのアクセスの許可を求められるので、
「許可」を押して許可してください。

そしてカメラに入るようにバーコードをうつして、
画像をクリックすると認識が始まります。
(クリックできる範囲が広い方が良いかと思って画像のクリックにしています。)

向きは逆さまなどでも良いようですが、
画質が悪いと認識できないことが多いようなので、
しっかりピントを合わせてカメラいっぱいにうつすと認識しやすいようです。

必要な数だけ繰り返した後、
不要になったら
「カメラを閉じる」
ボタンを押せば
ブラウザからカメラを使うのを止めます。

最後に下のテキストエリアに認識できたバーコードがたまっているので、
コピーして自由に使えます。

## 仕組み

画像ファイルから認識する方は
HTML5 の File API
を使ってローカルファイルを読み込んでいます。

iOS や Android で撮影が選べる機能は
`<input type="file" accept="image/*;capture=camera">`
のように
`accept`
属性を付けるだけで使えます。

画像は
`img`
要素に読み込んで、
読み終わったら
`canvas`
要素にコピーして
`canvas`
要素から画像データを取得しています。

最終的にその画像データは、
Web Workers
という HTML5 の機能を使ってバックグラウンドで
[BarcodeReader](https://github.com/EddieLa/BarcodeReader)
の
`DecoderWorker.js`
を動かして、
バーコードの認識をしています。

### カメラを扱う仕組み

まず
`getUserMedia`
でカメラを開いて
video
要素に設定します。

よくある使用例では
`video`
要素には
`autoplay`
属性を付けることが多いようですが、
今回は `video` 要素の `play()` を呼ぶようにしました。

`getUserMedia`
は一度変数に保存して、
その function を呼ぶという使い方は出来ないようだったので、
以下のように `call` で呼ぶようにしました。

```coffeescript
  getUserMedia =
    navigator.getUserMedia ||
    navigator.webkitGetUserMedia ||
    navigator.mozGetUserMedia ||
    navigator.msGetUserMedia

  getUserMedia.call(navigator, {audio:false,video:true,toString:->"video"}, onStream, onError)
```

`onStream`
の引数にわたってきた
`stream`
は保存しておいて後で使いました。

カメラを閉じるには
`video` 要素で再生を止めるのではなく、
`stream` の方の `stop()` を呼ぶ必要がありました。

一度 `stream` を `stop()` してしまうと、
再度再生するには、
また許可を求める必要があったので、
毎回閉じるのではなく、
連続読み込み用に閉じるのは別ボタンの機能にしました。
