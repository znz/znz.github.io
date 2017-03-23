---
layout: post
title: "pinentry-emacsを使ってみた"
date: 2017-03-23 20:30:13 +0900
comments: true
categories: emacs gpg
---
`gpg-agent` で `--allow-emacs-pinentry` が使えない gnupg2 の環境で、
https://github.com/ecraven/pinentry-emacs を使ってみました。

<!--more-->

## 動作確認環境

- macOS Sierra 10.12.3
- GNU Emacs 25.1.1
- gnupg2 2.0.30_3 (Homebrew で入れたもの)

## 注意事項

[README.org の末尾](https://github.com/ecraven/pinentry-emacs/blob/7c384a65eaaa37d38dbbb4e4ca89a094b498d811/README.org#emacs)にも
「This is probably totally insecure, and your passphrase may be leaked! Use at your own risk!」と書いてありますが、
`read-passwd` した文字列をそのまま `emacsclient` に返して、シェルスクリプトでそのまま扱っているので、
全く安全ではありません。

他の `pinentry` が使える時は避けることをオススメします。

## 別の方法

`gnupg` が 2.1.5 以降なら `--allow-emacs-pinentry` というオプションがあるので、
https://elpa.gnu.org/packages/pinentry.html と組み合わせて使うことをオススメします。
(`elpa` の説明によると `pinentry` も 0.9.5 以上が必要のようです。)
この方法は別途試してまた記事を書く予定です。

今回の環境では、
[Homebrew の gnupg2](https://github.com/Homebrew/homebrew-core/blob/328a89b492b600686be41b6b69b93d7c88fb8b89/Formula/gnupg2.rb)
が 2.0.30 で 2.1.5 未満なので使えませんでした。

macOS 上なら Emacs 上で入力するよりも
[OS X の Emacs で EasyPG が gpg2 で Opening input file: Decryption failed, になったので対処した](/blog/2016-08-20-mac-easypg-gpg2.html)記事に書いた
`pinentry-mac` を使うのがオススメです。

## 設定

### gpg-agent の設定

`~/.gnupg/gpg-agent.conf` がなければ作成して、以下の設定を入れておきます。

```
pinentry-program /path/to/github.com/ecraven/pinentry-emacs/pinentry-emacs
```

### emacs の設定

README.org に書いてあるように `~/.emacs.d/init.el` に

```
(defun pinentry-emacs (desc prompt ok error)
  (let ((str (read-passwd (concat (replace-regexp-in-string "%22" "\"" (replace-regexp-in-string "%0A" "\n" desc)) prompt ": "))))
    str))
```

を追加しておきます。

`emacsclient` コマンドで `pinentry-emacs` 関数を呼んでいるので、
`(server-start)` も (README.org には書いていませんが) 必要です。

## 動作確認

Emacs の中で `*.gpg` ファイルを開いてみたり、
端末上でパスフレーズの必要な `gpg` コマンドを実行してみたりして、
`(server-start)` した Emacs の mini buffer でパスフレーズがきかれるのを確認します。

## fallback 動作

`emacsclient` の呼び出しが `sed` に [pipe されている](https://github.com/ecraven/pinentry-emacs/blob/7c384a65eaaa37d38dbbb4e4ca89a094b498d811/pinentry-emacs#L24) ので、
[Fix fallback when emacsclient failed](https://github.com/ecraven/pinentry-emacs/pull/5)
のパッチをあてないと fallback してくれないようです。
(取り込まれたらブロク記事を書こうと思っていたのですが、取り込まれないようなのでもう書くことにしました。)

さらに別途 `pinentry-emacs` ディレクトリにパスを通しておくか、
`pinentry-emacs` ファイルの先頭で `PATH=$PATH:$(dirname "$0")` などとしてパスを通さないと
`lukspinentry` の実行に失敗して fallback してくれません。

さらに、端末上で `gpg` コマンドを実行したとしても、
`pinentry` プログラムは `gpg-agent` から起動される
(プログラム間の関係は http://miniconf.debian.or.jp/assets/files/gnupg-now.html の GnuPG programs (2) 参照)
ので、
`tty` が「`not a tty`」を返すらしく、
`pinentry-curses --ttyname not a tty`
が実行されていて fallback もうまくいきません。

なので、
fallback 動作は期待せず、
パッチもあてず、
`pinentry-emacs` を使うなら
必ず Emacs は起動しておく、
という運用が良さそうです。

## まとめ

`pinentry-emacs` を試してみましたが、
制限事項も多く、
修正も期待できないため、
別のもっと良い `pinentry` が使える場合は、
他のものを使うことをオススメします。

他の手段がない時の最終手段としては、知っておいても良いのではないかと思いました。

ちなみに自分の環境では、
`pinentry-mac` に戻してしまっていて、
`pinentry-emacs` は使っていません。
