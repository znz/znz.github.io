---
layout: post
title: "Vagrant で Ubuntu 14.04 を試す"
date: 2014-04-18 13:00:00 +0900
comments: true
categories: ubuntu vagrant
---
Vagrant で Ubuntu 14.04 LTS (trusty) を試しました。

<!--more-->

## Vagrantfile 作成

まず `Vagrantfile` を作成します。
box ファイルの URL は
https://cloud-images.ubuntu.com/vagrant/trusty/
から探します。
current にするといつのイメージかわかりにくいので、
日付の URL と box 名を指定しています。

https://cloud-images.ubuntu.com/vagrant/
には
`Ubuntu Server 14.04 LTS (Trusty Tahr) daily builds`
と書いてあるのに 20140417 と 20140418 の前が 20140221 と 20140222 になっていて
daily ではなさそうに見えるのですが、
リリース前は更新を止めていただけなのかもしれません。

```console
% mkdir trusty
% cd trusty
% vagrant init
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
% vi Vagrantfile
(config.vm.box と config.vm.box_url を設定)
% egrep '^ *[^ #]' Vagrantfile
VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "trusty-amd64-20140418"
  config.vm.box_url = "https://cloud-images.ubuntu.com/vagrant/trusty/20140418/trusty-server-cloudimg-amd64-vagrant-disk1.box"
end
```

この段階ではファイルとディレクトリを削除すれば元に戻せます。

## box の追加と起動

初回の `vagrant up` は box のダウンロードとインストールもするので
時間がかかります。

```console
% vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Box 'trusty-amd64-20140418' could not be found. Attempting to find and install...
    default: Box Provider: virtualbox
    default: Box Version: >= 0
==> default: Adding box 'trusty-amd64-20140418' (v0) for provider: virtualbox
    default: Downloading: https://cloud-images.ubuntu.com/vagrant/trusty/20140418/trusty-server-cloudimg-amd64-vagrant-disk1.box
==> default: Box download is resuming from prior download progress
==> default: Successfully added box 'trusty-amd64-20140418' (v0) for 'virtualbox'!
==> default: Importing base box 'trusty-amd64-20140418'...
==> default: Matching MAC address for NAT networking...
==> default: Setting the name of the VM: trusty_default_1397790655665_45006
==> default: Clearing any previously set forwarded ports...
==> default: Fixed port collision for 22 => 2222. Now on port 2200.
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 => 2200 (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2200
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: Warning: Connection timeout. Retrying...
    default: Warning: Remote connection disconnect. Retrying...
==> default: Machine booted and ready!
GuestAdditions 4.3.10 running --- OK.
==> default: Checking for guest additions in VM...
==> default: Mounting shared folders...
    default: /vagrant => /path/to/trusty
```

この段階では vagrant box が追加されて VM が作成されているので、
元に戻すには `vagrant destroy` で VM を削除して、
`vagrant box remove box名` で box を削除します。

普通は `vagrant destroy` で VM を破棄するだけで
box は使い回すと思います。

box を先に削除してしまうと VM の削除でエラーになるようなので、
その場合は GUI からエラーになっているディスクイメージを無視するようにするなど
がんばって削除します。

