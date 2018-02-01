---
layout: post
title: "XQuartzでX転送がうまくいかなかったので調べた"
date: 2018-02-01 23:13:13 +0900
comments: true
categories: osx homebrew ssh
---
XQuartz で openssh の X 転送がうまくいかなかったので、調べてみました。

<!--more-->

## 環境

- macOS High Sierra 10.13.3, macOS Sierra 10.12.6
- XQuartz 2.7.11

## インストール方法

XQuartz は `brew cask install xquartz` で入れました。
手動でダウンロードしてインストールした時もあまり違いはなさそうです。

## 初回の問題

初回インストール時はインストーラーの最後で再起動を促されますが、再起動した後でもダメでした。

そもそも、再起動していないと ssh 経由でない状態でも X が起動しないので、今回は無関係でした。

## X 転送の許可の問題

接続先の `/etc/ssh/sshd_config` は `X11Forwarding yes` になっていて、ちゃんと許可されていました。

## xauth の問題

色々悩んだ末、 `ssh -Xv` のように `-v` をつけてデバッグログを表示させて、よくみてみると、

    debug1: No xauth program.
    Warning: untrusted X11 forwarding setup failed: xauth key data not generated

と出ていて xauth の問題だとわかりました。

なぜか `/opt/X11/bin` にパスが通っていなかったのが原因らしく、 `~/.ssh/config` に

    XAuthLocation /opt/X11/bin/xauth

と設定して X サーバー側 (XQuartz 側) の問題は解決しました。

これとは別に X クライアント側 (ssh で接続した先) でも xauth がない環境があったので、 `sudo apt install xauth` でインストールして解決しました。

## PATH の問題

動作確認によく使っている `xeyes` などもローカル環境で起動しなかったので、普通は `/opt/X11/bin` にパスを通しておいた方が良さそうです。
普段使っている iTerm2 + zsh ではなく Terminal.app + bash だと問題なくパスが通っていたので、パスが通らないのは、自分の設定の問題の可能性もたかそうでした。

`/etc/paths.d/40-XQuartz` に `/opt/X11/bin` と書かれていて、普通はこれでパスが通るようです。

## まとめ

X 転送ができなかった環境で原因と対策を調べてみました。

結局、ログが重要ということで、原因追求にはサーバー側のログを見たり `ssh` の `-v` の数を増やしてログをしっかりみたりするしかないようです。
