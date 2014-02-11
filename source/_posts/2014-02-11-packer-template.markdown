---
layout: post
title: "packerのtemplateを1から作ってみた"
date: 2014-02-11 16:37:00 +0900
comments: true
categories: packer
---
packer.io のドキュメントの例の最初が amazon になっていて、
気軽に試しにくいと思ったので、
VirtualBox で最小限の設定を順番に試してみました。

<!--more-->

## 動作確認バージョン

- homebrew で入れた Packer v0.5.1
- VirtualBox 4.3.6

## 空ファイルから builders の type 設定まで

まず最低限必要な設定を `packer validate` で調べていきます。
最初は短いのでパイプを使ってコマンドラインのみで試し始めました。

```console
% packer -v
Packer v0.5.1
% echo | packer validate -
Failed to parse template: Error in line 2, char -1: unexpected end of JSON input

% echo '{}' | packer validate -
Failed to parse template: 1 error(s) occurred:

* No builders are defined in the template.
% echo '{"builders":[{}]}' | packer validate -
Failed to parse template: 2 error(s) occurred:

* builder 1: missing 'type'
* No builders are defined in the template.
% echo '{"builders":[{"type":"virtualbox-iso"}]}' | packer validate -
Template validation failed. Errors are shown below.

Errors validating build 'virtualbox-iso'. 4 error(s) occurred:

* An ssh_username must be specified.
* Due to large file sizes, an iso_checksum is required
* The iso_checksum_type must be specified.
* One of iso_url or iso_urls must be specified.
```

## virtualbox-iso の最低限の設定

長くなってきたので、ここからファイルを使うことにしました。

```console
% cat template.json
{
  "builders": [{
    "type": "virtualbox-iso",
    "ssh_username": "packer",
    "iso_checksum": "db79d463072da26b91c14e08b5a77a77bec9476ad1e5b0d2241228e4d59233f12c74477e77d427e407e1f45da4d2414c76367554352f57480fc7c00dde164028",
    "iso_checksum_type": "sha512",
    "iso_urls": [
      "http://ftp.jaist.ac.jp/debian-cd/7.4.0/amd64/iso-cd/debian-7.4.0-amd64-netinst.iso",
      "http://cdimage.debian.org/debian-cd/7.4.0/amd64/iso-cd/debian-7.4.0-amd64-netinst.iso"
    ]
  }]
}
% packer validate template.json
Template validation succeeded, but there were some warnings.
These are ONLY WARNINGS, and Packer will attempt to build the
template despite them, but they should be paid attention to.

Warnings for build 'virtualbox-iso':

* A shutdown_command was not specified. Without a shutdown command, Packer
will forcibly halt the virtual machine, which may result in data loss.
```

## boot_command 設定

preseed.cfg と組み合わせるための boot_command の最低限の設定は以下のようになりました。
オプションの部分は追加削除しやすいように行ごとに `<wait>` を入れてソートしています。

```json
    "boot_command": [
      "<esc><wait>",
      "install<wait>",
      " auto=true<wait>",
      " netcfg/get_domain=example.jp<wait>",
      " netcfg/get_hostname={{ .Name }}<wait>",
      " url=http://{{ .HTTPIP }}:{{ .HTTPPort }}/preseed.cfg<wait>",
      "<enter><wait>"
    ],
```

### boot: プロンプト

Ubuntu の server 用インストーラーだと `<esc><esc><enter><wait>` になっているようですが、
Debian の netinst のインストーラーの場合は `<esc><wait>` だけで `boot:` プロンプトになりました。
余計な `<esc>` などがあるとメニューに戻ってしまうようなので、
うまくいかないときは実際に試して確認するのが良さそうです。

### install

`Install` のところで Tab を押してコマンドラインを確認すると
`/install.amd/vmlinuz vga=788 initrd=/install.amd/initrd.gz -- quiet`
となっていたので、展開して以下の設定でも良いようです。
`--` より後ろの部分はインストール後のカーネルの引数にも使われるので、
`quiet` を削りたい場合は展開する必要がありそうです。

```json
    "boot_command": [
      "<esc><wait>",
      "/install.amd/vmlinuz<wait>",
      " auto=true<wait>",
      " netcfg/get_domain=example.jp<wait>",
      " netcfg/get_hostname={{ .Name }}<wait>",
      " url=http://{{ .HTTPIP }}:{{ .HTTPPort }}/preseed.cfg<wait>",
      " vga=788<wait>",
      " initrd=/install.amd/initrd.gz<wait>",
      " -- quiet<wait>",
      "<enter><wait>"
    ],
```

### 自動モード

言語設定などを preseed.cfg に書くための
[自動モード](https://www.debian.org/releases/stable/amd64/apbs02.html.ja#preseed-auto)
は
[The parameter of boot option “auto” had been changed for auto install of wheezy](http://d.palmtb.net/2013/08/22/the_parameter_of_boot_option__auto__had_been_changed_for_auto_install_of_wheezy.html)
に書いてあるように wheezy だと `auto` ではなく `auto=true` にする必要がありました。

[preseed で利用できるエイリアス](https://www.debian.org/releases/stable/amd64/apbs02.html.ja#preseed-aliases)
は preseed.cfg の中では使えないようです。
(試した範囲では `d-i keymap select jp` ではなく `d-i keyboard-configuration/xkb-keymap select jp` にする必要がありました。)

### preseed.cfg

`"http_directory": "http"` で指定したディレクトリに `preseed.cfg` を置いて
`url=http://{{ .HTTPIP }}:{{ .HTTPPort }}/preseed.cfg`
で参照しています。

`http_directory` の指定を忘れると `{{ .HTTPPort }}` が 0 になっているなど、
ちゃんと見ていればなんか変だと気付けるようです。

### ホスト名、ドメイン名

`{{ .Name }}` を使えるように boot_command で設定しています。
固定なら preseed.cfg の中でも良いと思います。

## preseed.cfg

https://www.debian.org/releases/wheezy/example-preseed.txt などを参考にして、
preseed.cfg を以下のように設定して途中で止まることなくインストールできました。

example-preseed.txt にはなかった設定の
`d-i grub-installer/only_debian boolean true`
を追加すると grub を MBR にインストールするかどうかというところも
止まらずに進みました。

```text http/preseed.cfg
#### Contents of the preconfiguration file (for wheezy)
### Localization
d-i debian-installer/locale string ja_JP
d-i localechooser/supported-locales multiselect en_US.UTF-8, ja_JP.EUC-JP
#d-i keymap select jp
d-i keyboard-configuration/xkb-keymap select jp

### Network configuration
d-i netcfg/choose_interface select auto
d-i netcfg/wireless_wep string

### Mirror settings
d-i mirror/protocol string http
d-i mirror/country string manual
d-i mirror/http/hostname string ftp.jp.debian.org
d-i mirror/http/directory string /debian
d-i mirror/http/proxy string
d-i mirror/suite string wheezy

### Account setup
d-i passwd/root-login boolean false
d-i passwd/make-user boolean true
d-i passwd/user-fullname string Vagrant User
d-i passwd/username string vagrant
d-i passwd/user-password password vagrant
d-i passwd/user-password-again password vagrant

### Clock and time zone setup
d-i clock-setup/utc boolean true
d-i time/zone string Japan
d-i clock-setup/ntp boolean true
d-i clock-setup/ntp-server string ntp.mfeed.ad.jp

### Partitioning
d-i partman-auto/method string lvm
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-md/device_remove_md boolean true
d-i partman-lvm/confirm boolean true
d-i partman-lvm/confirm_nooverwrite boolean true
d-i partman-auto/choose_recipe select atomic
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true
# Install the GRUB boot loader to the master boot record?
d-i grub-installer/only_debian boolean true

### Package selection
tasksel tasksel/first multiselect minimal
d-i pkgsel/include string openssh-server build-essential
d-i pkgsel/upgrade select none
popularity-contest popularity-contest/participate boolean false

### Finishing up the installation
d-i finish-install/reboot_in_progress note
```

## template.json

最終的に上の `preseed.cfg` と組み合わせる最低限の template.json は以下のようになりました。

```json template.json
{
  "builders": [{
    "type": "virtualbox-iso",
    "boot_command": [
      "<esc><wait>",
      "install<wait>",
      " auto=true<wait>",
      " netcfg/get_domain=example.jp<wait>",
      " netcfg/get_hostname={{ .Name }}<wait>",
      " url=http://{{ .HTTPIP }}:{{ .HTTPPort }}/preseed.cfg<wait>",
      "<enter><wait>"
    ],
    "guest_os_type": "Debian_64",
    "http_directory": "http",
    "ssh_username": "vagrant",
    "ssh_password": "vagrant",
    "shutdown_command": "echo vagrant|sudo -S /sbin/shutdown -hP now",
    "iso_checksum": "db79d463072da26b91c14e08b5a77a77bec9476ad1e5b0d2241228e4d59233f12c74477e77d427e407e1f45da4d2414c76367554352f57480fc7c00dde164028",
    "iso_checksum_type": "sha512",
    "iso_urls": [
      "http://ftp.jaist.ac.jp/debian-cd/7.4.0/amd64/iso-cd/debian-7.4.0-amd64-netinst.iso",
      "http://cdimage.debian.org/debian-cd/7.4.0/amd64/iso-cd/debian-7.4.0-amd64-netinst.iso"
    ]
  }]
}
```

## まとめ

`template.json` と `http/preseed.cfg` を作成して `packer build template.json` で `output-virtualbox-iso/packer-virtualbox-iso-disk1.vmdk` と `output-virtualbox-iso/packer-virtualbox-iso.ovf` が 8 分ぐらいで作成できました。
(iso は packer_cache/ にダウンロード済みの状態)

この ovf ファイルは VirtualBox にインポートして使えます。

最低限の設定しかしていないので、他の template.json を参考にして provisioners の設定を足していくなどのカスタマイズをしていけば良さそうだと思いました。

```console
% packer validate template.json
Template validated successfully.
% packer build template.json
virtualbox-iso output will be in this color.

==> virtualbox-iso: Downloading or copying Guest additions checksums
    virtualbox-iso: Downloading or copying: http://download.virtualbox.org/virtualbox/4.3.6/SHA256SUMS
==> virtualbox-iso: Downloading or copying Guest additions
    virtualbox-iso: Downloading or copying: http://download.virtualbox.org/virtualbox/4.3.6/VBoxGuestAdditions_4.3.6.iso
==> virtualbox-iso: Downloading or copying ISO
    virtualbox-iso: Downloading or copying: http://ftp.jaist.ac.jp/debian-cd/7.4.0/amd64/iso-cd/debian-7.4.0-amd64-netinst.iso
==> virtualbox-iso: Starting HTTP server on port 8081
==> virtualbox-iso: Creating virtual machine...
==> virtualbox-iso: Creating hard drive...
==> virtualbox-iso: Creating forwarded port mapping for SSH (host port 3213)
==> virtualbox-iso: Starting the virtual machine...
==> virtualbox-iso: Waiting 10s for boot...
==> virtualbox-iso: Typing the boot command...
==> virtualbox-iso: Waiting for SSH to become available...
==> virtualbox-iso: Connected to SSH!
==> virtualbox-iso: Uploading VirtualBox version info (4.3.6)
==> virtualbox-iso: Uploading VirtualBox guest additions ISO...
==> virtualbox-iso: Gracefully halting virtual machine...
    virtualbox-iso:
    virtualbox-iso: We trust you have received the usual lecture from the local System
    virtualbox-iso: Administrator. It usually boils down to these three things:
    virtualbox-iso:
    virtualbox-iso: #1) Respect the privacy of others.
    virtualbox-iso: #2) Think before you type.
    virtualbox-iso: #3) With great power comes great responsibility.
    virtualbox-iso:
    virtualbox-iso: [sudo] password for vagrant:
    virtualbox-iso: The system is going down for system halt NOW!
==> virtualbox-iso: Preparing to export machine...
    virtualbox-iso: Deleting forwarded port mapping for SSH (host port 3213)
==> virtualbox-iso: Exporting virtual machine...
==> virtualbox-iso: Unregistering and deleting virtual machine...
Build 'virtualbox-iso' finished.

==> Builds finished. The artifacts of successful builds are:
--> virtualbox-iso: VM files in directory: output-virtualbox-iso
```
