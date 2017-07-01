---
layout: post
title: "Debian/Ubuntuで暗号化 LVM を使いつつ自動起動する"
date: 2017-07-01 14:00:00 +0900
comments: true
categories: debian ubuntu linux
---
さくらの VPS 環境でも ssh などの秘密鍵を置くなら、ディスクの暗号化は使いたいので、リリースされたばかりの Debian 9 の ISO をアップロードしてインストールして、暗号化されていない `/boot` に鍵ファイルを置いて自動起動を設定しました。

自動起動を設定するということはセキュリティ的には弱くなりますが、そこはホストを信用するということにしています。

自動起動設定時に `/etc/crypttab` の設定ミスで起動しなくなるということがあったので、そういう時の直し方も含めてまとめてみました。

<!--more-->

## 確認環境

- Debian GNU/Linux 9.0 (stretch)

Ubuntu でも debian-installer ベースのインストーラーを使った場合は同じだと思います。
(Live 環境が起動するデスクトップ版のインストーラーの場合は暗号化 LVM でのインストールができるかどうか確認していないのでわかりません。)

他のバージョンの Debian でも luks 対応の cryptsetup があれば同じだと思います。

## インストール

普通に netinst の iso でインストールします。

ただし途中の「ディスクのパーティショニング」で「ガイド - ディスク全体を使い、暗号化 LVM をセットアップする」を選んだ場合が対象です。
パーティションは `/` (と `/boot`) だけを想定しています。
暗号化のパスフレーズは鍵ファイル設定前の起動時と、鍵ファイルの追加時などしか使わないので、長くて強いものにしておくと良いと思います。

VirtualBox などの仮想環境で試す時は、暗号化前のランダムなデータで上書きでデータ用のパーティション全体に書き込みが発生するので、可変サイズのディスクではなく固定サイズのディスクにしておくと良いかもしれません。

## 鍵ファイル作成

まず、鍵ファイルを作成します。
内容作成前に root しか読み書きできないようにするために、touch して chmod しておきます。
次に urandom から読んだランダムデータを鍵ファイルに書き込みます。
前回設定したときは count=1 で 1024 バイトにしていましたが、今回は 4096 バイトにしてみました。
最後に root はパーミッションだと書き込み禁止できないので、誤操作防止 (削除やヒストリーから dd を再実行してしまうなど) のために chattr で ext2 の immutable 属性をつけておきます。(確認は `sudo lsattr /boot/keyfile`)

    sudo touch /boot/keyfile
    sudo chmod 400 /boot/keyfile
    sudo dd if=/dev/urandom of=/boot/keyfile bs=1024 count=4
	sudo chattr +i /boot/keyfile

## 情報確認

- `/etc/fstab`: `/dev/mapper/HOSTNAME--vg-root` が `/` に、 `/dev/mapper/HOSTNAME--vg_swap_1` がスワップパーティションに設定されています (`HOSTNAME` はインストーラーで設定したホスト名)
- `/etc/crypttab`: `vda5_crypt UUID=... none luks` で UUID で指定された `/dev/vda5` の暗号化が解除された状態が `/dev/mapper/vda5_crypt` として見えるということがわかります
- `lsblk`: ツリー上にみえます (`lsblk -f` だと UUID も表示されました)
- `ls -l /dev/disk/by-uuid`: uuid とデバイスの対応を確認できます (これで確認できる vda1 のパスをあとで使います)
- `sudo cryptsetup luksDump /dev/vda5`: luks の情報が表示できます (最初は Key Slot 0 だけ ENABLED で 1 から 7 は DISABLED になっています)

## 鍵追加

`cryptsetup luksAddKey` で鍵を追加します。
ここで最初に設定したパスフレーズが必要です。
なぜか `Key slot 0 unlocked.` が2回でましたが、特に問題はなさそうです。

    $ sudo cryptsetup -v luksAddKey /dev/vda5 /boot/keyfile
    Enter any passphrase:
	Key slot 0 unlocked.
	Key slot 0 unlocked.
    Command successful.

## 鍵削除

違うファイルを登録してしまったり、同じ鍵を複数回追加してしまったりしたときには `cryptsetup luksRemoveKey` で削除できます。
この場合はその鍵自身で unlock されるようなので、パスフレーズは不要でした。

    $ sudo cryptsetup -v luksRemoveKey /dev/vda5 /boot/keyfile
	Key slot 1 unlocked.
	Key slot 1 selected for deletion.
	Command successful.

特定の Key Slot を DISABLED に戻したいときは `cryptsetup luksKillSlot` が使えます。
この場合はパスフレーズが必要でした。

    $ sudo cryptsetup -v luksKillSlot /dev/vda5 2
	Key slot 2 selected for deletion.
    Enter any remaining passphrase:
	Key slot 0 unlocked.
	Command successful.

## 自動起動設定

この段階ではまだ暗号化解除に使える鍵が増えただけで、再起動してもパスフレーズを要求されるままです。

`/etc/crypttab` を以下のように書き換えます。

    vda5_crypt UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /dev/disk/by-uuid/yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy:/keyfile luks,keyscript=/lib/cryptsetup/scripts/passdev

xxx... の方の UUID は `/dev/vda5` の UUID なので、そのまま書き換えません。
第3項目の `none` を `/boot` パーティションのデバイスを UUID を使って指定したパス + `:` + `/boot` パーティション内での `keyfile` へのパスに書き換えます。
第4項目の `luks` は `luks,keyscript=/lib/cryptsetup/scripts/passdev` に書き換えます。
`passdev` は `cryptsetup` パッケージで用意されているファイルなので、そのまま書きます。

## initramfs 更新

書き換えてもまだブートプロセスに反映されていないので、再起動してもパスフレーズを要求されるままなので、
最後に initramfs を更新します。

    sudo update-initramfs -u

これで再起動すると自動起動するようになります。

keyscript のパスが間違っていると以下のように WARNING が出るので、再起動する前に気づくことができますが、
keyfile の指定は間違っていても何も出ないので注意する必要があります。

    $ sudo update-initramfs -u
	update-initramfs: Generating /boot/initrd.img-4.9.0-3-amd64
	cryptsetup: WARNING: target vda5_crypt has an invalid keyscript, skipped
	cryptsetup: WARNING: target vda5_crypt has an invalid keyscript, skipped

## 起動失敗した場合

`/etc/crypttab` の設定をミスして起動しなくなった場合、 netinst の ISO からレスキューモードで起動すればパスフレーズでマウントできます。
そしてルートファイルシステムとして `/dev/HOSTNAME-vg/root` (`HOSTNAME` はインストーラーで設定したホスト名) を選び、 `/boot` パーティションもマウントしてシェルを起動します。

シェルは `/bin/sh -i` なので使いにくければ `bash` を起動して、`/etc/crypttab` を修正して `update-initramfs -u` で反映させます。
そして exit で抜けて再起動します。

レスキュー環境での修正が難しそうなら、 `none` と `luks` だけに戻して、パスフレーズを使う通常起動にしてから直すという方法もあります。

## 最後に

知らないところで暗号化が解除できてしまうのは、コンソール接続が毎回必要になることとのトレードオフですが、鍵ファイルでも解約時に `chattr -i /boot/keyfile; shred --remove /boot/keyfile` でディスク全体の削除に似た効果を期待できます。ただし [Man page of SHRED](https://linuxjm.osdn.jp/html/GNU_coreutils/man1/shred.1.html)の警告に書いてあるように上書きを期待しているので、 ext2 になっている `/boot` はファイルシステム的には大丈夫だとしても、その下のブロックデバイスで上書きされていない可能性は残りそうです。

## まとめ

暗号化 LVM を使うことで macOS の FileVault や Windows BitLocker のように簡単にディスクほぼ全体 (`/boot` を除く) を暗号化できました。
そして、再起動したい時に常にコンソールに接続できるとは限らない環境向けに鍵ファイルで自動起動の設定もできました。

トレードオフもちゃんと考えた上で設定すれば、安全な環境が簡単に作れると思いました。
