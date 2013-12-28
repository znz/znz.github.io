---
layout: post
title: "Ubuntu Server 12.04.3 LTS をインストールした"
date: 2013-09-15 20:02
comments: true
categories: osx ubuntu
---
自宅サーバーのシステムディスク (sda) が壊れたので、 HDD を入れ替えて、せっかくなのでシステムをきれいな状態にするために `ubuntu-12.04.3-server-amd64.iso` からインストールし直しました。
その初期設定の話です。

<!--more-->

## インストール準備

### ISO イメージダウンロード

まずインストール用の ISO ファイルをダウンロードしました。
[Get Ubuntu](http://www.ubuntu.com/download)
からだと海外ミラーでダウンロードが遅かったので、
国内のミラーを探してみたところ、
`ftp://ftp.riken.go.jp/Linux/ubuntu-iso/CDs/precise/`
だと torrent ファイルや metalink のファイルしか見えなくて、
`ftp` ではなく `http` の
`http://ftp.riken.go.jp/Linux/ubuntu-iso/CDs/precise/`
からだとダウンロード出来ました。

### 起動用 USB メモリ準備

ダウンロードした ISO を `dd` で USB メモリに書き込んで、そこからインストールしました。
categories につけた Mac を使っている部分はここだけです。

ネットで調べればいろいろ情報がみつかるように
`diskutil list`
でディスクの番号を確認して、
`dd` で読み書きします。
`rdisk` の方が速いらしいので、
実際の書き込みには `rdisk` を使ったのですが、
パーティションを `unmount` しておかないと
`Resource busy` で書き込ませんでした。
eject してしまって `diskutil list` にも
出てこない状態にしてしまうと読み書き出来ないので、
Finder から取り出したり `diskutil unmountDisk` だと
ダメでした。

読み込みの例として MBR の部分をバックアップしてダンプしてみました。

```
% diskutil list
/dev/disk0
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *251.0 GB   disk0
   1:                        EFI                         209.7 MB   disk0s1
   2:          Apple_CoreStorage                         250.1 GB   disk0s2
   3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3
/dev/disk1
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                  Apple_HFS Macintosh HD           *249.8 GB   disk1
/dev/disk2
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *16.1 GB    disk2
   1:             Windows_FAT_32 NO NAME                 16.1 GB    disk2s1
% sudo dd if=/dev/disk2 of=MF-AU316GBS.mbr bs=512 count=1
% hexdump -C MF-AU316GBS.mbr
00000000  33 c0 8e d0 bc 00 7c fb  50 07 50 1f fc be 1b 7c  |3.....|.P.P....||
00000010  bf 1b 06 50 57 b9 e5 01  f3 a4 cb bd be 07 b1 04  |...PW...........|
00000020  38 6e 00 7c 09 75 13 83  c5 10 e2 f4 cd 18 8b f5  |8n.|.u..........|
00000030  83 c6 10 49 74 19 38 2c  74 f6 a0 b5 07 b4 07 8b  |...It.8,t.......|
00000040  f0 ac 3c 00 74 fc bb 07  00 b4 0e cd 10 eb f2 88  |..<.t...........|
00000050  4e 10 e8 46 00 73 2a fe  46 10 80 7e 04 0b 74 0b  |N..F.s*.F..~..t.|
00000060  80 7e 04 0c 74 05 a0 b6  07 75 d2 80 46 02 06 83  |.~..t....u..F...|
00000070  46 08 06 83 56 0a 00 e8  21 00 73 05 a0 b6 07 eb  |F...V...!.s.....|
00000080  bc 81 3e fe 7d 55 aa 74  0b 80 7e 10 00 74 c8 a0  |..>.}U.t..~..t..|
00000090  b7 07 eb a9 8b fc 1e 57  8b f5 cb bf 05 00 8a 56  |.......W.......V|
000000a0  00 b4 08 cd 13 72 23 8a  c1 24 3f 98 8a de 8a fc  |.....r#..$?.....|
000000b0  43 f7 e3 8b d1 86 d6 b1  06 d2 ee 42 f7 e2 39 56  |C..........B..9V|
000000c0  0a 77 23 72 05 39 46 08  73 1c b8 01 02 bb 00 7c  |.w#r.9F.s......||
000000d0  8b 4e 02 8b 56 00 cd 13  73 51 4f 74 4e 32 e4 8a  |.N..V...sQOtN2..|
000000e0  56 00 cd 13 eb e4 8a 56  00 60 bb aa 55 b4 41 cd  |V......V.`..U.A.|
000000f0  13 72 36 81 fb 55 aa 75  30 f6 c1 01 74 2b 61 60  |.r6..U.u0...t+a`|
00000100  6a 00 6a 00 ff 76 0a ff  76 08 6a 00 68 00 7c 6a  |j.j..v..v.j.h.|j|
00000110  01 6a 10 b4 42 8b f4 cd  13 61 61 73 0e 4f 74 0b  |.j..B....aas.Ot.|
00000120  32 e4 8a 56 00 cd 13 eb  d6 61 f9 c3 49 6e 76 61  |2..V.....a..Inva|
00000130  6c 69 64 20 70 61 72 74  69 74 69 6f 6e 20 74 61  |lid partition ta|
00000140  62 6c 65 00 45 72 72 6f  72 20 6c 6f 61 64 69 6e  |ble.Error loadin|
00000150  67 20 6f 70 65 72 61 74  69 6e 67 20 73 79 73 74  |g operating syst|
00000160  65 6d 00 4d 69 73 73 69  6e 67 20 6f 70 65 72 61  |em.Missing opera|
00000170  74 69 6e 67 20 73 79 73  74 65 6d 00 00 00 00 00  |ting system.....|
00000180  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
000001b0  00 00 00 00 00 2c 44 63  18 2e 07 c3 00 00 00 00  |.....,Dc........|
000001c0  31 00 0c fe ff ff 30 00  00 00 50 f6 de 01 00 00  |1.....0...P.....|
000001d0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
000001f0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 55 aa  |..............U.|
00000200
%  sudo dd if=Downloads/ubuntu-12.04.3-server-amd64.iso of=/dev/rdisk2 bs=1m
Password:
dd: /dev/rdisk2: Resource busy
%  diskutil unmount disk2s1
Volume NO NAME on disk2s1 unmounted
%  diskutil list
/dev/disk0
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *251.0 GB   disk0
   1:                        EFI                         209.7 MB   disk0s1
   2:          Apple_CoreStorage                         250.1 GB   disk0s2
   3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3
/dev/disk1
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                  Apple_HFS Macintosh HD           *249.8 GB   disk1
/dev/disk2
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *16.1 GB    disk2
   1:             Windows_FAT_32 NO NAME                 16.1 GB    disk2s1
%  sudo dd if=Downloads/ubuntu-12.04.3-server-amd64.iso of=/dev/rdisk2 bs=1m
665+0 records in
665+0 records out
697303040 bytes transferred in 41.551843 secs (16781519 bytes/sec)
%
```

この USB メモリを自作マシンにさして起動してインストールすることになります。

## インストール

インストール作業は暗号化する以外はあまり特殊なことをせずに最小インストールにしました。

* Del キーで EFI BOIS Menu を開いて、起動メニューを開いて USB メモリからを選んで起動しました。
* タスク (tasksel) は何も選択せずにインストールしました。
* 1TB の HDD (sda) を暗号化 LVM で初期化しました。

## 初期設定

### etckeeper

* `sudo aptitude install git`
* `sudo aptitude install etckeeper`
* `sudoedit /etc/etckeeper/etckeeper.conf` で `VCS="git"` と `GIT_COMMIT_OPTIONS="-v"` を設定
* `sudo etckeeper init`
* `sudo etckeeper commit "Initial commit"`

### ssh

まず `ufw` で接続制限をしました。
多段防御の一つ目としてここではポート番号だけの制限にしています。

* `sudo ufw enable`
* `sudo ufw allow 22/tcp`
* `sudo ufw limit 22/tcp`
  * `サポートされていない IPv6 'limit' ルールを飛ばします` と出るので、
    IPv6 用に allow と limit の両方を実行しています。
* `sudo ufw allow 3843/tcp`
  * 22 以外のポートでも待ち受けるなら、それも許可します。
  * ここでは例として 3843 番ポートにしています。
    (実際に使っているポート番号は違います。)
  * アタックは少なそうなので、`limit` は付けていません。

次に tcp wrapper で接続制限をしました。
ここでは接続元の制限をしています。
接続元として使っていて、逆引きが `.jp` でないところなどは
別途 `sshd: 10.1.2.3` のように許可していきます。

```
$ egrep '^[^#]' /etc/hosts.deny
ALL: ALL
$ egrep '^[^#]' /etc/hosts.allow
sshd: 127.0.0.1 [::1]
sshd: 192.168.0.0/16
sshd: .jp
```

最後に `/etc/sshd/sshd_config` の設定例です。
`ssh_config` とは別物なので注意が必要です。

* `Port 3843` を追加します。
  * 複数ポートで待ち受けるには `Port` 設定の行を複数書けば良いです。
* `PermitRootLogin no` にします。
  `root` ログインが必要なら
  `PermitRootLogin forced-commands-only` か
  `PermitRootLogin without-password` にすればよく、
  公開サーバーで `PermitRootLogin yes` にする必要はないはずです。
* `AllowUsers firstuser` で最初のユーザーのログインを許可します。
* ログインするユーザーの `.ssh/authorized_keys` の設定をしてから
  `PasswordAuthentication no` の設定をします。
  `ChallengeResponseAuthentication no` になっていることを
  確認しておかないと
  `UsePAM yes` との組み合わせで `keyboard-interactive` 認証が
  発生してパスワードで入れてしまうことがあるので注意が必要です。
  昔はこれが原因で公開鍵認証のみに制限出来ていないという話が多かった気がします。

### quota 設定

ユーザー ID ごとのディスク使用量の把握のために、 `quota` を設定します。

`/etc/fstab` の
`errors=remount-ro` を
`errors=remount-ro,noatime,usrquota,grpquota,user_xattr`
に書き換えます。

`quota` パッケージを入れておけば、次回起動時に自動で
`quotacheck` が動いて `repquota` が使えるようになります。
ファイルが多いと `quotacheck` に時間がかかるので、
出来るだけ最小インストールでファイルが少ないうちに
有効にした方が時間の節約になります。

### IP アドレス設定

`bridge-utils` パッケージを入れて、
`sudoedit /etc/network/interfaces` で
`br0` ありの固定 IP アドレスの設定に変更しました。

```
# The primary network interface
auto eth0
iface eth0 inet manual
auto br0
iface br0 inet static
	address 192.168.xx.yy
	netmask 255.255.255.0
	gateway 192.168.253.1
	dns-nameservers 192.168.xx.yy
	dns-search home.example.jp
	bridge_ports eth0
	bridge_stp off
	bridge_fd 0
	bridge_maxwait 0
```

### パスフレーズなし起動設定

リモートから再起動した時などに止まってしまうことを防ぐ設定をします。

これは鍵を暗号化の外に平文で置いておくということで、
セキュリティ的には弱くなってしまうので、
そのトレードオフを許容出来るかどうかを
考えておく必要があります。

```
$ sudo dd if=/dev/random of=/root/sda5_crypt.key bs=512 count=4
dd: warning: partial read (128 bytes); suggest iflag=fullblock
0+4 レコード入力
0+4 レコード出力
447 バイト (447 B) コピーされました、 0.000294833 秒、 1.5 MB/秒
$ sudo chmod 400 /root/sda5_crypt.key
$ sudo cryptsetup luksAddKey /dev/sda5 /root/sda5_crypt.key
パスフレーズを入力:
$ sudo mv /root/sda5_crypt.key /boot/sda5_crypt.key
$ sudoedit /etc/crypttab
$ sudo update-initramfs -u
update-initramfs: Generating /boot/initrd.img-3.8.0-30-generic
```

Ubuntu 12.04 では、
`crypttab` は
`sda5_crypt UUID=(/dev/sda5のUUID) none luks`
を
`sda5_crypt UUID=(/dev/sda5のUUID) /dev/disk/by-uuid/(/bootのUUID):sda5_crypt.key luks,keyscript=/lib/cryptsetup/scripts/passdev`
のように書き換えます。
`/bootのUUID` は `/etc/fstab` を参照するのがわかりやすいと思います。
`by-uuid` だけではなく `by-label` などのパスも使えると思いますが、
一番誤動作が少なそうな `by-uuid` を使っています。


その後、
`update-initramfs`
で
`/boot/initrd.img-*`
に変更を反映させておく必要があります。

### GRUB で止まるのを防ぐ

`/etc/default/grub` に
`GRUB_RECORDFAIL_TIMEOUT=5`
のような設定を追加しておくと
起動時の GRUB で止まることがなくなります。

なぜか前回起動失敗扱いになってしまって、
GRUB で止まってしまう環境があったのですが、
この設定を入れてからは止まらなくなりました。

`/etc/default/grub` の変更は
`update-grub` で
`/boot/grub/grub.cfg` に
反映させる必要があります。

```
$ sudoedit /etc/default/grub
$ sudo update-grub
Generating grub.cfg ...
Found linux image: /boot/vmlinuz-3.8.0-30-generic
Found initrd image: /boot/initrd.img-3.8.0-30-generic
Found linux image: /boot/vmlinuz-3.8.0-29-generic
Found initrd image: /boot/initrd.img-3.8.0-29-generic
Found memtest86+ image: /memtest86+.bin
done
```
