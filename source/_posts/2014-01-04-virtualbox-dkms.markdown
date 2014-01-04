---
layout: post
title: "virtualboxでdkmsによるモジュールのリビルドが動いていなかったので対処した"
date: 2014-01-04 23:47:41 +0900
comments: true
categories: virtualbox dkms ubuntu
---
Ubuntu 12.04.3 LTS の環境で
virtualbox.org から入れた VirtualBox を
virtualbox-4.2 (4.2.20-90983~Ubuntu~precise)
から
virtualbox-4.3 (4.3.6-91406~Ubuntu~precise)
にあげても dkms によるモジュールのリビルドがちゃんと動かないままだったので、
直し方を調べて対処してみました。

<!--more-->

## 状況

カーネルを linux-image-3.8.0-34-generic から
linux-image-3.8.0-35-generic にあげるときに
virtualbox-4.2 (4.2.20-90983~Ubuntu~precise)
から
virtualbox-4.3 (4.3.6-91406~Ubuntu~precise)
にあげました。

以下の作業は linux-image-3.8.0-34-generic で起動中に
再起動後に使われる linux-image-3.8.0-35-generic 用の
VirtualBox のカーネルモジュールをビルドしようとしています。

## エラーの状況

`dkms status` で調べると以下のエラーメッセージが出る状態でした。

```console
  $ sudo dkms status
  Error! Could not locate dkms.conf file.
  File:  does not exist.
```

## 対処その1

`/var/lib/dkms/vboxhost` を退避して
`dpkg-reconfigure` しなおせば良いという情報があったので
試してみました。

この段階での `dkms status` は調べ忘れていたのですが、
後の状況からすると
今起動中のカーネルのモジュールだけビルドされたようです。

```console
  $ sudo mv /var/lib/dkms/vboxhost /tmp/
  $ sudo dpkg-reconfigure virtualbox-4.3
  Stopping VirtualBox kernel modules ...done.
  addgroup: グループ `vboxusers' はシステムグループとしてすでに存在しています。終了します。
  Unknown media type in type 'all/all'
  Unknown media type in type 'all/allfiles'
  Unknown media type in type 'uri/mms'
  Unknown media type in type 'uri/mmst'
  Unknown media type in type 'uri/mmsu'
  Unknown media type in type 'uri/pnm'
  Unknown media type in type 'uri/rtspt'
  Unknown media type in type 'uri/rtspu'
  Stopping VirtualBox kernel modules ...done.
  Uninstalling old VirtualBox DKMS kernel modules ...done.
  Removing old VirtualBox pci kernel module ...done.
  Removing old VirtualBox netadp kernel module ...done.
  Removing old VirtualBox netflt kernel module ...done.
  Removing old VirtualBox kernel module ...done.
  Trying to register the VirtualBox kernel modules using DKMS ...done.
  Starting VirtualBox kernel modules ...done.
```

## 対処その2

purge した古いカーネルのモジュールが残っていたので、
削除しました。
安全のため `/tmp` への移動にしていますが、
いきなり `rm -rf` で削除しても良いと思います。
他の古いバージョンも残っていれば削除してしまっても
大丈夫だと思います。

ねんのため
`/etc/init.d/vboxdrv setup`
を実行してみましたが、
`dpkg-reconfigure`
で既に実行済みだったので関係なかったようです。

この段階で `dkms status` を確認してみると
起動中のカーネルのモジュールだけビルドされていることがわかります。

```console
  $ sudo mv /lib/modules/3.8.0-33-generic/ /tmp/
  $ sudo /etc/init.d/vboxdrv setup
  Stopping VirtualBox kernel modules ...done.
  Uninstalling old VirtualBox DKMS kernel modules ...done.
  Trying to register the VirtualBox kernel modules using DKMS ...done.
  Starting VirtualBox kernel modules ...done.
  $ sudo dkms status
  vboxhost, 4.3.6, 3.8.0-34-generic, x86_64: installed
```

## 対処その3

dkms でのモジュールのビルドは
`/etc/kernel/postinst.d/dkms`
で実行されているので、
カーネルを再インストールすれば確実ということで、
カーネルを `aptitude reinstall` しました。

その結果、正常にビルドされていることが確認できたので、
再起動して VirtualBox の動作確認をしました。

```console
  $ sudo aptitude reinstall linux-image-3.8.0-35-generic
  以下のパッケージが再インストールされます:
    linux-image-3.8.0-35-generic
  0 個のパッケージを更新、 0 個を新たにインストール、 再インストール: 1 個、 0 個を削除予定 、0 個が更新されていない。
  アーカイブ 48.2 M バイト中 0  バイトを取得する必要があります。 展開後に 0  バイトのディス ク領域が新たに消費されます。
  (データベースを読み込んでいます ... 現在 117478 個のファイルとディレクトリがインストールされています。)
  linux-image-3.8.0-35-generic 3.8.0-35.50~precise1 を (.../linux-image-3.8.0-35-generic_3.8.0-35.50~precise1_amd64.deb で) 置換するための準備をしています ...
  Done.
  linux-image-3.8.0-35-generic を展開し、置換しています...
  Examining /etc/kernel/postrm.d .
  run-parts: executing /etc/kernel/postrm.d/initramfs-tools 3.8.0-35-generic /boot/vmlinuz-3.8.0-35-generic
  run-parts: executing /etc/kernel/postrm.d/zz-update-grub 3.8.0-35-generic /boot/vmlinuz-3.8.0-35-generic
  linux-image-3.8.0-35-generic (3.8.0-35.50~precise1) を設定しています ...
  Running depmod.
  update-initramfs: deferring update (hook will be called later)
  Not updating initrd symbolic links since we are being updated/reinstalled
  (3.8.0-35.50~precise1 was configured last, according to dpkg)
  Not updating image symbolic links since we are being updated/reinstalled
  (3.8.0-35.50~precise1 was configured last, according to dpkg)
  Examining /etc/kernel/postinst.d.
  run-parts: executing /etc/kernel/postinst.d/apt-auto-removal 3.8.0-35-generic /boot/vmlinuz-3.8.0-35-generic
  run-parts: executing /etc/kernel/postinst.d/dkms 3.8.0-35-generic /boot/vmlinuz-3.8.0-35-generic
  run-parts: executing /etc/kernel/postinst.d/initramfs-tools 3.8.0-35-generic /boot/vmlinuz-3.8.0-35-generic
  update-initramfs: Generating /boot/initrd.img-3.8.0-35-generic
  run-parts: executing /etc/kernel/postinst.d/pm-utils 3.8.0-35-generic /boot/vmlinuz-3.8.0-35-generic
  run-parts: executing /etc/kernel/postinst.d/update-notifier 3.8.0-35-generic /boot/vmlinuz-3.8.0-35-generic
  run-parts: executing /etc/kernel/postinst.d/zz-update-grub 3.8.0-35-generic /boot/vmlinuz-3.8.0-35-generic
  Generating grub.cfg ...
  Found linux image: /boot/vmlinuz-3.8.0-35-generic
  Found initrd image: /boot/initrd.img-3.8.0-35-generic
  Found linux image: /boot/vmlinuz-3.8.0-34-generic
  Found initrd image: /boot/initrd.img-3.8.0-34-generic
  Found memtest86+ image: /memtest86+.bin
  done
  $ sudo dkms status
  vboxhost, 4.3.6, 3.8.0-34-generic, x86_64: installed
  vboxhost, 4.3.6, 3.8.0-35-generic, x86_64: installed
  $
```

## まとめ

古いバージョンは再起動で消えてしまって確認できないのですが、
多分古い VirtualBox では
`/var/lib/dkms/vboxhost/*/source/dkms.conf`
が存在していなくて、
バージョンを上げても古いバージョンの `source` が残ったままだと
同じエラーが出続けるようなので、
クリーンインストールやそれに近い状態にすると直るということのようです。


