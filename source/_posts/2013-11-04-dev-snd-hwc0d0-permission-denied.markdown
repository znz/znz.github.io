---
layout: post
title: "/dev/snd/hwC0D0でPermission deniedになる問題を調べた"
date: 2013-11-04 14:22
comments: true
categories: linux debian ubuntu
---
`/dev/snd/hwC0D0` を `O_RDWR` で `open(2)` するところで
`Permission denied` になるという話
(
[ツイート](https://twitter.com/takaswie/status/397014733494026240)、
[Ubuntu日本語フォーラム](https://forums.ubuntulinux.jp/viewtopic.php?pid=100488#p100488)
)
が気になったので、調べてみました。

<!--more-->

## 結論

最終的にどうすれば良いか知りたい人向けの情報としては、
`setcap cap_sys_rawio=ep filename`
でケーパビリティ (capability) を設定する、ということになります。

以下は、その結論にたどり着くまでに調べたことのメモです。

## パーミッションと ACL

[第 77 回 関西 Debian 勉強会に参加した](http://blog.n-z.jp/blog/2013-10-27-kansai-debian-meeting.html)
で書いたように、
audio グループに属しているか、コンソールから直接ログインしていれば
パーミッションの問題はないはずです。

## 余談: デバイスの sticky bit

`crw-rw---T` になっていて、
もしかして末尾の sticky bit が影響しているのかと思って調べてみたところ、
[Re: Sticky bit on device files?](http://lists.debian.org/debian-user/2012/02/msg01273.html)
によると udev の管理用のフラグとして使われているようでした。
今回の件とは関係なさそうだったので、これ以上深追いはしていません。

## AppArmor

`/etc/apparmor.d/abstractions/audio` に
`/dev/snd/*      rw,` とあるので、
念のため
`sudo service apparmor stop`
で `AppArmor` を止めて試してみましたが、
変化が無かったので、
`sudo service apparmor start`
で戻しました。

## カーネルのソースコード探索

`strace` などで確認しても、
ユーザーランドでは `EACCES` が返ってくるとしかわからないので、
こうなったらカーネルのソースコードから `EACCES` を返しているところを
探すしかないということで、
`apt-get source linux-image-$(uname -r)`
でソースコードをダウンロードして探してみました。

`grep -r EACCES sound` で探してみると
`sound/pci/hda/hda_hwdep.c` で以下のように
`CAP_SYS_RAWIO` をみていることがわかりました。

```c sound/pci/hda/hda_hwdep.c
 static int hda_hwdep_open(struct snd_hwdep *hw, struct file *file)
 {
 #ifndef CONFIG_SND_DEBUG_VERBOSE
 	if (!capable(CAP_SYS_RAWIO))
 		return -EACCES;
 #endif
 	return 0;
 }
```

## Linux のケーパビリティ (capability)

[Man page of CAPABILITIES](http://linuxjm.sourceforge.jp/html/LDP_man-pages/man7/capabilities.7.html)
が関係しているということがわかったところで、
設定方法も調べてみると、
`setcap` で設定できるとわかったので、
以下のような簡単なテストプログラムを用意して、
`sudo setcap cap_sys_rawio=ep ./a.out`
でケーパビリティを設定すると
`open(2)`
に成功するのを確認できました。
`cap_sys_rawio=ep` は危険そうなので、
テストプログラムとはいえ、
任意のパスを受け取れるようにするのは止めた方が良さそうに思いました。

```c open-hwC0D0.c
 #include <sys/types.h>
 #include <sys/stat.h>
 #include <fcntl.h>
 #include <stdio.h>

 int main() {
 	open("/dev/snd/hwC0D0", O_RDWR);
	perror("open");
	return 0;
 }
```
