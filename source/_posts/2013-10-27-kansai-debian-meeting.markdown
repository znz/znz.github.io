---
layout: post
title: "第 77 回 関西 Debian 勉強会に参加した"
date: 2013-10-27 13:46
comments: true
categories: event debian kansaidebian alsa
---
[第 77 回 関西 Debian 勉強会](https://wiki.debian.org/KansaiDebianMeeting/20131027)
に参加してきました。

前回の「Linuxとサウンドシステム」
の続きの
「ALSAのユーザーランド解説」
と
「git-buildpackage 入門 again」
という話でした。

<!--more-->

## Intro

`jessie` に向けての話で、
デフォルトの `init` が `systemd` に変わるかも、
というのが気になりました。
他に UTF-8 といえば UTF-8 に対応していない namazu のようなものが
どうなるのかというのもちょっと気になりました。

事前課題では、
前回の時に発表者の坂本さんに
VirtualBox の中の wheezy で音が出ない原因を調べてもらって、
`alsamixer` でボリュームが 0 になっていたから、
というのを直してもらっていたので、
すんなり音を出すことができました。

佐々木さんは警告のフラグ (`-Wall -Wextra`) を増やしてビルドしていて、
その警告メッセージを事前課題のところに書いていました。

警告の一部は `if` 文の処理内容が `;` だけで `{}` でくくった方が良いという警告だったのですが、
何もしないままよりも、
後ろにコメントで書いてあるメッセージを `puts` などで出力するようにした方が良いのではないかと思いました。

## ALSA のユーザーランド解説

事前課題の添付ファイルの `pcm_minimal.c` の `device` のところにある
PCM ノードの指定は `default` 以外にも
`aplay -L` (または `arecord -L`) で表示されるものが指定できるという話がありました。

```c
/* playback PCM node */
static char *device = "default";
```

`dmix`, `dsnoop`, `hw`, `plughw` はデフォルトでは隠されていて、
`~/.asoundrc` で以下のように設定をオーバーライドすれば `aplay -L` などで
出てくるようになるという話がありました。
(ALSA の上流や Debian では出ないのがデフォルトで、
Ubuntu ではパッチが当たっていて、
標準で出てくるようになっているという話もありました。)

```text ~/.asoundrc
defaults.namehint.extended on
```

その前に asound の設定ファイルは独自形式で、
書式の説明は `libasound2-doc` パッケージの
`/usr/share/doc/libasound2-doc/html/`
以下にあるという話もありました。
独自形式のため、ライブラリとして設定ファイルを読み込む部分も持っているそうです。

PulseAudio 関係のパッケージを入れた時に
default が dmix/dsnoop から pulse に変わって、
PulseAudio のデーモンはログイン時に起動するという話もありました。

Android ではユーザーランドは asound ではなく tinyalsa で
カーネルランドは ASoC という話もありました。


## git-buildpackage 入門 again

実際に `git-buildpackage` を使ってみるという内容でした。
最初にモバイルルーターなどを使ってみんなネットにつながるようにしてから始まりました。

事前課題で候補にあがったものを実際に `git-buildpackage` でビルドしてみるという話が続きました。
最後は時間切れで次回予告などは片付けをしながらになってしまいました。

最新の `git-buildpackage` だと `git` と同じように `gbp` コマンドとそのサブコマンドになっていて、
`wheezy` だと `gbp-clone` などの `-` 付きのコマンドを実行する必要がありました。

`git-dch` という `git` のコミットログから `debian/changelog` を生成するツールではコミットログの1行目だけとってくるのがデフォルトの動作で、
`--full` というオプションでコミットログ全体を使えるとか、
man page の `META TAGS` にあるような情報を入れられるとか、
そういう話もしていました。

`gbp clone` と `git clone` の違いは、
gbp 用の設定 (`debian/gbp.conf` とか) を見るとか、
upstream ブランチとかの設定をしてくれるということでした。

`pbuilder` を使っていても前処理の `fakeroot debian/rules clean` が `pbuilder` の外で動いて、
そこで依存パッケージが必要なことがあるので `git-buildpackage` の前に
`build-dep` でビルドに必要なパッケージを入れる必要があるという話がありました。

## /dev/snd の ACL

Debian 勉強会ということでホスト OS は Mac OS X なのですが、
サウンド関連や git-buildpackage は
VirtualBox の中の wheezy で試していました。

そのときに `ls -al /dev/snd` でサウンド関連のデバイスファイルに
`+` がついていて ACL がついているということに気がついて、
どこで設定されているのか気になったので帰ってから調べてみました。

```
$ ls -al /dev/snd
合計 0
drwxr-xr-x   3 root root      180 10月 27 22:44 .
drwxr-xr-x  13 root root     3180 10月 27 22:44 ..
drwxr-xr-x   2 root root       60 10月 27 22:44 by-path
crw-rw---T+  1 root audio 116,  5 10月 27 22:44 controlC0
crw-rw---T+  1 root audio 116,  4 10月 27 22:44 pcmC0D0c
crw-rw---T+  1 root audio 116,  3 10月 27 22:44 pcmC0D0p
crw-rw---T+  1 root audio 116,  2 10月 27 22:44 pcmC0D1c
crw-rw---T+  1 root audio 116,  1 10月 27 22:44 seq
crw-rw---T+  1 root audio 116, 33 10月 27 22:44 timer
$ getfacl /dev/snd/seq
getfacl: Removing leading '/' from absolute path names
# file: dev/snd/seq
# owner: root
# group: audio
# flags: --t
user::rw-
user:admin0:rw-
group::rw-
mask::rw-
other::---
```

まず、 `grep -r snd /lib/udev/rules.d` でデバイスを作成している場所を
探してみましたが、 `audio` グループに設定している箇所しか見つかりませんでした。

```text /lib/udev/rules.d/91-permissions.rules
# sound devices
SUBSYSTEM=="sound",                             GROUP="audio",
        OPTIONS+="static_node=snd/seq", OPTIONS+="static_node=snd/timer"
```

次に、ログインしているユーザーの ACL が追加されているということで、
PAM が関係しているだろうという見当をつけて、
`/etc/pam.d` を確認してみたところ、
`common-session` の `pam_ck_connector` が関係してそうだと気付きました。

```text /etc/pam.d/common-session
session	optional			pam_ck_connector.so nox11
```

そこで `dpkg -L consolekit` などで `ConsoleKit` 関連のファイルをみていくと
`/usr/lib/ConsoleKit/run-seat.d/udev-acl.ck -> /lib/udev/udev-acl`
が関係してそうだと気がついたので、
もう一度 udev のルールを確認してみると `70-udev-acl.rules` で
`udev-acl` というタグを設定していました。

```text /lib/udev/rules.d/70-udev-acl.rules
# sound devices
SUBSYSTEM=="sound", TAG+="udev-acl"
```

`70-udev-acl.rules` をみたり `udev-acl` のソースをみると `ACL` を設定しているのは
`udev-acl` で間違いなさそうでした。
ただし、 `70-udev-acl.rules` で
`# systemd replaces udev-acl entirely, skip if active`
と書いてあったので、 `systemd` を使っている環境だと
`udev-acl` は何もしないようです。
