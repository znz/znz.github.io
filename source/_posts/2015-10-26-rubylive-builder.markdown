---
layout: post
title: "RubyLive ISO イメージを更新した"
date: 2015-10-26 21:44:33 +0900
comments: true
categories: live debian vagrant
---
[RubyLiveを仮想環境で作成](http://blog.n-z.jp/blog/2014-07-13-build-rubylive-on-vm.html "RubyLiveを仮想環境で作成")したものを更新しました。

<!--more-->

## 動作確認環境

- VirtualBox 5.0.6
- Vagrant 1.7.4

## ビルド環境の更新

`git clone https://github.com/znz/rubylive-builder` で取得していた `rubylive-builder` 環境の `Vagrantfile` の `config.vm.box` を `"ffuenf/debian-7.6.0-amd64"` から `"ffuenf/debian-8.2.0-amd64"` に[更新しました](https://github.com/znz/rubylive-builder/commit/289f2079dad86bde424aa810f0e6b28302607ccb)。

## ビルド内容の更新

### ruby のバージョンの更新

`config/hooks/10-ruby-rails.chroot` でインストールしている ruby を 2.1.4 から 2.2.3 に[更新しました](https://github.com/znz/rubylive/commit/475e077f9bdb066628415ad602fefdde2608c57e)。

### 壁紙の変更

壁紙を RubyKaigi 2014 のロゴから関西 Ruby 会議 06 のロゴに[変更しました](https://github.com/znz/rubylive/commit/f7f4136a39fd1ac4da8121204b941b12eac3e65e)。

変更しても `config/includes.chroot/usr/share/images/desktop-base/rubykaigi2014-mark.svg` にダウンロードしたファイルは残ったままで、そのまま ISO イメージを作成しなおすと古い画像も ISO イメージの中に入ってしまうので、手動で削除しておく必要があります。
(`rake distclean` でも消えません。)

試行錯誤した後には特に `config/includes.chroot` 以下に不要なファイルが残っていないか注意する必要があります。

### jessie への更新 (失敗)

ビルド環境を jessie にあげても作成される ISO は wheezy のままだったので、以下の変更で jessie にしようとしたのですが、失敗したので結局 wheezy のままにしました。

変更点は以下の通りです。

```
diff --git a/auto/config b/auto/config
index d44fc2d..3dc3665 100755
--- a/auto/config
+++ b/auto/config
@@ -2,6 +2,7 @@

 lb config noauto \
        --architectures "amd64" \
+       --distribution "jessie" \
        --bootloader "grub2" \
        --templates "templates" \
        --bootappend-live "quiet locales=ja_JP.UTF-8 timezone=Asia/Tokyo utc=no keyboard-layouts=jp
diff --git a/config/package-lists/30-japanese.list.chroot b/config/package-lists/30-japanese.list.ch
index 602270e..304d5f7 100644
--- a/config/package-lists/30-japanese.list.chroot
+++ b/config/package-lists/30-japanese.list.chroot
@@ -3,8 +3,8 @@ fbterm
 lv
 manpages-ja
 nkf
-ttf-mona
-ttf-monapo
+fonts-mona
+fonts-monapo
 fonts-ipafont-gothic
 fonts-ipafont-mincho
 fonts-vlgothic
diff --git a/config/package-lists/50-editors.list.chroot b/config/package-lists/50-editors.list.chro
index 8468dec..cb2197a 100644
--- a/config/package-lists/50-editors.list.chroot
+++ b/config/package-lists/50-editors.list.chroot
@@ -1,4 +1,4 @@
-emacs23
+emacs24
 vim
 vim-gtk
 gedit
```

失敗した部分は以下の通りです。

```
[2015-10-26 13:27:55] lb chroot_live-packages
dpkg: dependency problems prevent removal of systemd:
 libpam-systemd:amd64 depends on systemd (= 215-17+deb8u2).

dpkg: error processing package systemd (--purge):
 dependency problems - not removing
dpkg: dependency problems prevent removal of systemd-sysv:
 init depends on systemd-sysv | sysvinit-core | upstart; however:
  Package systemd-sysv is to be removed.
  Package sysvinit-core is not installed.
  Package upstart is not installed.
 libpam-systemd:amd64 depends on systemd-shim (>= 8-2) | systemd-sysv; however:
  Package systemd-shim is not installed.
  Package systemd-sysv is to be removed.

dpkg: error processing package systemd-sysv (--purge):
 dependency problems - not removing
Errors were encountered while processing:
 systemd
 systemd-sysv
P: Begin unmounting filesystems...
P: Saving caches...
Reading package lists...
Building dependency tree...
Reading state information...
rake aborted!
Command failed with status (1): [sudo lb build...]
```

## ISO のビルド

`vagrant up` して `vagrant ssh` で入った後、`/vagrant/rubylive.sh` で作成できます。
2 回目以降は `~/rubylive` で `time rake APT_HTTP_PROXY=http://localhost:3142` を直接実行しても良いと思います。
