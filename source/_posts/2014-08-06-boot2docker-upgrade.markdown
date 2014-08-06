---
layout: post
title: "boot2docker のバージョンアップ"
date: 2014-08-06 23:10:26 +0900
comments: true
categories: docker boot2docker
---
boot2docker の ISO の更新は専用コマンドで簡単にできるとわかったので、
わざと古いバージョンに戻したりして動作確認してみました。

<!--more-->

## 対象バージョン

- Mac OS X 10.9.4
- VirtualBox 4.3.14
- docker 1.1.2
- boot2docker 1.1.2

## 更新準備

`boot2docker delete` して消してから ISO を更新して
`boot2docker init` で作り直すという説明もありますが、
再起動しても残る部分は ISO とは別の仮想ディスクの `/dev/sda1` に保存されていて、
そのまま使い回せることがほとんどなので、
作り直さなくても更新できます。

大きくバージョンをあげるとか、クリーンな環境でやり直したいと言うときは消して作り直せば良いと思います。

参考: [boot2dockerのデータ永続化まとめ - Qiita](http://qiita.com/tukiyo3/items/07f1eb77b5ffd9031e30 "boot2dockerのデータ永続化まとめ - Qiita")

## 更新方法

### VM を停止して更新

`boot2docker upgrade` で VM が起動していれば停止して更新して起動し直してくれます。

### ISO だけ更新

`boot2docker download` で ISO だけ無条件にダウンロードしてくれます。

### boot2docker help

以上の説明は `boot2docker help` にちゃんと書いてある通りです。

```
% boot2docker help
Usage: boot2docker [<options>] <command> [<args>]

boot2docker management utility.

Commands:
    init                    Create a new boot2docker VM.
    up|start|boot           Start VM from any states.
    ssh [ssh-command]       Login to VM via SSH.
    save|suspend            Suspend VM and save state to disk.
    down|stop|halt          Gracefully shutdown the VM.
    restart                 Gracefully reboot the VM.
    poweroff                Forcefully power off the VM (might corrupt disk image).
    reset                   Forcefully power cycle the VM (might corrupt disk image).
    delete|destroy          Delete boot2docker VM and its disk image.
    config|cfg              Show selected profile file settings.
    info                    Display detailed information of VM.
    ip                      Display the IP address of the VM's Host-only network.
    status                  Display current state of VM.
    download                Download boot2docker ISO image.
    upgrade                 Upgrade the boot2docker ISO image (if vm is running it will be stopped and started).
    version                 Display version information.

Options:
      --basevmdk="": Path to VMDK to use as base for persistent partition
      --dhcp=true: enable VirtualBox host-only network DHCP.
      --dhcpip=192.168.59.99: VirtualBox host-only network DHCP server address.
  -s, --disksize=20000: boot2docker disk image size (in MB).
      --dockerport=2375: host Docker port (forward to port 2375 in VM).
      --hostip=192.168.59.3: VirtualBox host-only network IP address.
      --iso="/Users/knishiyama/.boot2docker/boot2docker.iso": path to boot2docker ISO image.
      --lowerip=192.168.59.103: VirtualBox host-only network DHCP lower bound.
  -m, --memory=2048: virtual machine memory size (in MB).
      --netmask=ffffff00: VirtualBox host-only network mask.
      --serial=false: try serial console to get IP address (experimental)
      --serialfile="": path to the serial socket/pipe.
      --ssh="ssh": path to SSH client utility.
      --ssh-keygen="ssh-keygen": path to ssh-keygen utility.
      --sshkey="/Users/knishiyama/.ssh/id_boot2docker": path to SSH key to use.
      --sshport=2022: host SSH port (forward to port 22 in VM).
      --upperip=192.168.59.254: VirtualBox host-only network DHCP upper bound.
      --vbm="VBoxManage": path to VirtualBox management utility.
  -v, --verbose=false: display verbose command invocations.
      --vm="boot2docker-vm": virtual machine name.
```
