---
layout: post
title: "Vagrant CloudからWheezyを入れてみた"
date: 2014-07-18 21:30:25 +0900
comments: true
categories: vagrant vagrantcloud debian
---
Debian 7.6 がリリースされたので、新しい box がないか探してみたところ、
[A list of base boxes for Vagrant - Vagrantbox.es](http://www.vagrantbox.es/ "A list of base boxes for Vagrant - Vagrantbox.es")
から探すのではなく
[Vagrant Cloud](https://vagrantcloud.com/)
を使えば良いということがわかりました。

<!--more-->

## 動作確認バージョン

- VirtualBox 4.3.12
- Vagrant 1.6.3
- [Debian Wheezy 7.6.0 x86_64](https://vagrantcloud.com/ffuenf/debian-7.6.0-amd64 "Debian Wheezy 7.6.0 x86_64") 0.0.27

## 書き換え

古い box は

```
  config.vm.box = ENV["VM_BOX"] || "opscode_debian-7.4_chef-provisionerless"
  config.vm.box_url = ENV["VM_BOX_URL"] || "http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_debian-7.4_chef-provisionerless.box"
```

と指定していたのを
[ffuenf/vagrant-boxes](https://github.com/ffuenf/vagrant-boxes "ffuenf/vagrant-boxes")
からリンクされている
[ffuenf/debian-7.6.0-amd64](https://vagrantcloud.com/ffuenf/debian-7.6.0-amd64 "ffuenf/debian-7.6.0-amd64")
の説明通り `vagrant init ffuenf/debian-7.6.0-amd64` で作成された
`Vagrantfile` を参考にして、

```
  config.vm.box = "ffuenf/debian-7.6.0-amd64"
```

に書き換えました。

以前の box を使っている 古い VM を `vagrant destroy` ですべて破棄した後、
`vagrant box remove opscode_debian-7.4_chef-provisionerless`
で box も削除しました。

## 余談

`vagrant box outdated` や `vagrant box update` の使い方がわからなかったのですが、
`box_url` でダウンロードしてきた box で使うものではなく Vagrant Cloud から
ダウンロードしてきた box で使うものだったようです。
