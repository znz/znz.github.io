---
layout: post
title: "第 129 回関西 Debian 勉強会 に参加しました"
date: 2017-11-26 21:00:00 +0900
comments: true
categories: debian event
---
[第 129 回関西 Debian 勉強会](https://wiki.debian.org/KansaiDebianMeeting/20171126) に参加しました。
一般ユーザー権限で LXC を使ってみるという内容でした。

<!--more-->

## 会場

いつもの福島区民センターでした。

## 事前課題

     lxc libvirt0 libpam-cgroup libpan-cgroup libpam-cgfs bridge-utils

とあったうち libpan-cgroup というのは間違いだったようです。

https://wiki.debian.org/LXC 参照。

- [Eucalyptus (software)](https://en.wikipedia.org/wiki/Eucalyptus_%28software%29) は開発が止まっている?
- [Hyper-Vとの関係](https://ja.wikipedia.org/wiki/Xen_%28%E4%BB%AE%E6%83%B3%E5%8C%96%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2%29#Hyper-V.E3.81.A8.E3.81.AE.E9.96.A2.E4.BF.82) によると Microsoft と XenSource は共同でやっているかも。

## 仮想化について

[カーネル/VM勉強会@関西 其の参 - カーネル／VM探検隊](http://www.kernelvm.org/ima-made-no-matome/kaneru-vm-mian-qiang-hui-guan-xi-qino-can) から「BHyVeってなんや」を参考にしながら概要を解説

## Debian Stretch で LXC を使う

- vagrant で [bento/debian-9.2](https://app.vagrantup.com/bento/boxes/debian-9.2) の box を使って試しました
- lxc-net を有効にするために `/etc/default/lxc` で `USE_LXC_BRIDGE="true"` に変更 (Debian Wiki は記述が古い (testing の時のパッケージが変更途中の内容?) のか `/etc/default/lxc-net` と書いてあるがそんなファイルはなかった)
- lxc-net の変更を反映するために再起動した (`sudo systemctl start lxc-net` とかでも反映できるかもしれないが未確認)
- `lxc-checkconfig` でチェック (今の安定版は全部緑の enabled になるはず (昔はカーネルが対応していなくてダメなものがあったはず) )
- `sudo sh -c 'echo "kernel.unprivileged_userns_clone=1" > /etc/sysctl.d/80-lxc-userns.conf'`
- `sudo sysctl --system`
- `kernel.unprivileged_userns_clone` の設定は Debian 固有のパッチの設定らしい? (1の方がバニラカーネルのデフォルト動作っぽい?)

- `sudo usermod --add-subuids 1258512-1324047 $USER` と `sudo usermod --add-subgids 1258512-1324047 $USER` はしなくても `/etc/subuid` と `/etc/subgid` に入っていた (`usermod` の引数は端の値の指定で `/etc/sub[ug]id` ファイルに書かれているのは開始 id と個数で別の意味なので注意)

```
vagrant@debian-9:~$ cat /etc/subuid
vagrant:100000:65536
vagrant@debian-9:~$ cat /etc/subgid
vagrant:100000:65536
```

- `echo "$USER veth lxcbr0 10"| sudo tee -i /etc/lxc/lxc-usernet` で一般ユーザー権限で作成できるブリッジの数を制限するらしい (`lxcbr0` の部分はブリッジ名依存)
- `mkdir -p .config/lxc`
- `.config/lxc/default.conf` を作成
- `id_map` の部分は subuid と subgid と同じ値にする必要あり
- `lxcbr0` の部分も `ip` コマンドなどで確認して合わせる必要あり

```
vagrant@debian-9:~$ cat .config/lxc/default.conf
lxc.include = /etc/lxc/default.conf
# Subuids and subgids mapping
lxc.id_map = u 0 100000 65536
lxc.id_map = g 0 100000 65536
# "Secure" mounting
lxc.mount.auto = proc:mixed sys:ro cgroup:mixed

# Network configuration
lxc.network.type = veth
lxc.network.link = lxcbr0
lxc.network.flags = up
#lxc.network.hwaddr = 00:16:3e:xx:xx:xx
```

- hwaddr は [MACアドレス](https://ja.wikipedia.org/wiki/MAC%E3%82%A2%E3%83%89%E3%83%AC%E3%82%B9) 参照
- コメントアウトしてみると自動設定になった

## lxc-create

特権だと `/var/lib/lxc` を使われるが、一般ユーザー権限だと普通は書き込めないのでディレクトリ指定をする必要あり (絶対パスじゃないとダメらしい)

```
vagrant@debian-9:~$ lxc-create -n stretch -t download -P ~/work/lxc
Setting up the GPG keyring
Downloading the image index

---
DIST	RELEASE	ARCH	VARIANT	BUILD
---
(略)
debian	stretch	amd64	default	20171124_22:42
(略)
---
Distribution: debian
Release: stretch
Architecture: amd64

Downloading the image index
Downloading the rootfs
Downloading the metadata
The image cache is now ready
Unpacking the rootfs

---
You just created a Debian container (release=stretch, arch=amd64, variant=default)

To enable sshd, run: apt-get install openssh-server

For security reason, container images ship without user accounts
and without a root password.

Use lxc-attach or chroot directly into the rootfs to set a root password
or create user accounts.
vagrant@debian-9:~$
```

## 起動

```
vagrant@debian-9:~$ lxc-ls --fancy -P ~/work/lxc
NAME    STATE   AUTOSTART GROUPS IPV4 IPV6
stretch STOPPED 0         -      -    -
vagrant@debian-9:~$ lxc-start -d -n stretch -P ~/work/lxc
vagrant@debian-9:~$ lxc-ls --fancy -P ~/work/lxc
NAME    STATE   AUTOSTART GROUPS IPV4 IPV6
stretch RUNNING 0         -      -    -
vagrant@debian-9:~$ lxc-ls --fancy -P ~/work/lxc
NAME    STATE   AUTOSTART GROUPS IPV4       IPV6
stretch RUNNING 0         -      10.0.3.146 -
```

## 接続して動作確認

```
vagrant@debian-9:~$ lxc-attach -n stretch
You lack access to /home/vagrant/.local/share/lxc
vagrant@debian-9:~$ lxc-attach -n stretch -P ~/work/lxc
root@stretch:/# apt update
...
1 package can be upgraded. Run 'apt list --upgradable' to see it.
root@stretch:/# ls -al /var/lib/apt/lists/
total 65872
drwxr-xr-x 3 root root     4096 Nov 26 06:32 .
drwxr-xr-x 5 root root     4096 Nov 24 22:47 ..
-rw-r--r-- 1 root root 38923281 Oct  7 09:04 deb.debian.org_debian_dists_stretch_main_binary-amd64_Packages
-rw-r--r-- 1 root root 26443489 Oct  7 09:04 deb.debian.org_debian_dists_stretch_main_i18n_Translation-en
-rw-r--r-- 1 root root   117945 Oct  7 09:46 deb.debian.org_debian_dists_stretch_Release
-rw-r--r-- 1 root root     2479 Oct  7 09:52 deb.debian.org_debian_dists_stretch_Release.gpg
-rw-r----- 1 root root        0 Nov 26 06:32 lock
drwx------ 2 _apt root     4096 Nov 26 06:32 partial
-rw-r--r-- 1 root root    62959 Nov 25 10:01 security.debian.org_dists_stretch_updates_InRelease
-rw-r--r-- 1 root root  1257072 Nov 21 22:08 security.debian.org_dists_stretch_updates_main_binary-amd64_Packages
-rw-r--r-- 1 root root   624275 Nov 21 22:08 security.debian.org_dists_stretch_updates_main_i18n_Translation-en
root@stretch:/# exit
vagrant@debian-9:~$ ls -al ~/work/lxc/stretch/rootfs/var/lib/apt/lists/
total 65872
drwxr-xr-x 3 100000 100000     4096 Nov 26 06:32 .
drwxr-xr-x 5 100000 100000     4096 Nov 24 22:47 ..
-rw-r--r-- 1 100000 100000 38923281 Oct  7 09:04 deb.debian.org_debian_dists_stretch_main_binary-amd64_Packages
-rw-r--r-- 1 100000 100000 26443489 Oct  7 09:04 deb.debian.org_debian_dists_stretch_main_i18n_Translation-en
-rw-r--r-- 1 100000 100000   117945 Oct  7 09:46 deb.debian.org_debian_dists_stretch_Release
-rw-r--r-- 1 100000 100000     2479 Oct  7 09:52 deb.debian.org_debian_dists_stretch_Release.gpg
-rw-r----- 1 100000 100000        0 Nov 26 06:32 lock
drwx------ 2 100104 100000     4096 Nov 26 06:32 partial
-rw-r--r-- 1 100000 100000    62959 Nov 25 10:01 security.debian.org_dists_stretch_updates_InRelease
-rw-r--r-- 1 100000 100000  1257072 Nov 21 22:08 security.debian.org_dists_stretch_updates_main_binary-amd64_Packages
-rw-r--r-- 1 100000 100000   624275 Nov 21 22:08 security.debian.org_dists_stretch_updates_main_i18n_Translation-en
```

## NAT

https://wiki.debian.org/LXC/SimpleBridge の

    up iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE

は

    down iptables -t nat -D POSTROUTING -o wlan0 -j MASQUERADE

もないと up down を繰り返すと増えそう。

lxc-net で試した環境は自動で NAT 設定が入っていた。

```
vagrant@debian-9:~$ sudo iptables -nL -t nat
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  10.0.3.0/24         !10.0.3.0/24
```

## 停止

資料にはなかったけど、個人的に停止と削除も試しました。

```
vagrant@debian-9:~$ lxc-stop -n stretch -P ~/work/lxc
vagrant@debian-9:~$ lxc-ls --fancy -P ~/work/lxc
NAME    STATE   AUTOSTART GROUPS IPV4 IPV6
stretch STOPPED 0         -      -    -
```

## 削除

```
vagrant@debian-9:~$ lxc-destroy -n stretch -P ~/work/lxc
Destroyed container stretch
vagrant@debian-9:~$ lxc-ls --fancy -P ~/work/lxc
vagrant@debian-9:~$ ls work/lxc/
lxc-monitord.log
```

## 休憩中の話

- https://www.ubuntu.com/server/maas

## ネットワーク図

- ネットワーク図を書いて議論
- https://twitter.com/YukiharuYABUKI/status/934688472845058054

## 次回

- 2017/12/24(日)

## まとめ

発表者の佐々木さんが病欠で、時間に余裕があったので、資料や Wiki の記述や不足点などをツッコミを入れたりするような感じになっていました。
その後は、ネットワーク図を書いて色々と議論をして、少し早めに終わりました。
