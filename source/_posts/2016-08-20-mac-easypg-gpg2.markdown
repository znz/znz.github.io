---
layout: post
title: "OS X の Emacs で EasyPG が gpg2 で Opening input file: Decryption failed, になったので対処した"
date: 2016-08-20 13:23:22 +0900
comments: true
categories: emacs gpg
---
OS X の Homebrew で入れている gnupg が更新されて gpg コマンドではなく gpg1 コマンドしか入らなくなって、 gpg コマンドは gnupg2 で入れるようになった影響で、 Emacs 上の EasyPG で `*.gpg` ファイルを開くときに `Opening input file: Decryption failed,` で開けなくなったので、その対処をしました。

<!--more-->

## 対象バージョン

OS 自体以外は Homebrew で入れたバージョンです。

- OS X : Yosemite (10.10.5)
- Emacs.app : 24.5.1
- gnupg : 1.4.21
- gnupg2 : 2.0.30_2
- gpg-agent : 2.0.30_1
- pinentry-mac : 0.9.4

## 現象

`*.gpg` ファイルを開くと今までは minibuffer でパスフレーズをきいてきていたのに、 gpg コマンドが gnupg 1 系から gnupg 2 系に変わったら、 `Opening input file: Decryption failed,` というエラー (`*Messages*` バッファには `epa-file--find-file-not-found-function: Opening input file: Decryption failed,` と出ていた) で開けなくなりました。

## 対処方法案

`(setq epg-gpg-program "gpg1")` で古い gnupg を使い続けるという案も考えましたが、今後のことを考えると新しいバージョンを使った方が良いだろうと思ってやめました。

[A. GnuPG および EasyPG アシスタントの設定](http://www.bookshelf.jp/cgi-bin/goto.cgi?file=auth-ja&node=GnuPG+and+EasyPG+Assistant+Configuration) によると gpg2 は gpg-agent との組み合わせが必須のようだったので、 gpg-agent を使うことにしました。

## 対処方法

端末上で `gpg -c hoge.txt` や `gpg -c hoge.txt.gpg` を試してみると `pinentry-curses` が使われているとわかったので、 `brew install pinentry-mac` で pinentry-mac をインストールして、 `Caveats` に出てきたように `~/.gnupg/gpg-agent.conf` を作成して `pinentry-program /usr/local/bin/pinentry-mac` という設定を入れたところ、 Emacs 上でも `*.gpg` ファイルを開くときに pinentry-mac でパスフレーズをきかれるようになって、開けるようになりました。

パスフレーズを入力できるようにするだけなら pinentry の設定だけで gpg-agent の起動は必要ありませんでした。
パスフレーズを毎回入力する必要があるというのは今までと同じ使い勝手なので、 gpg-agent の起動までは追求しませんでした。

```
==> Caveats
You can now set this as your pinentry program like

~/.gnupg/gpg-agent.conf
    pinentry-program /usr/local/bin/pinentry-mac

.app bundles were installed.
Run `brew linkapps pinentry-mac` to symlink these to /Applications.
```

## 将来の対処方法案

gpg-agent は 2.1.5 から `--allow-emacs-pinentry` というオプションが追加されていて、 http://unix.stackexchange.com/questions/55638/can-emacs-use-gpg-agent-in-a-terminal-at-all に書かれているように https://elpa.gnu.org/packages/pinentry.html を使って、 Emacs 上でパスフレーズを入力できるようになるようなので、またそのように設定を変更するかもしれません。
