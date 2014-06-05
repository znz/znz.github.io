---
layout: post
title: "Vagrant の Multi VM 間で IPsec を試した"
date: 2014-06-04 22:00:00 +0900
comments: true
categories: vagrant virtualbox linux ubuntu ipsec
---
正常に IPsec の暗号化通信ができているときの racoon のログなどを確認したかったので、
Vagrant と ansible で
IPsec で通信できる Multi VM 環境を作ってみました。

playbook は
https://github.com/znz/ansible-playbook-ipsec-demo
で公開しています。

<!--more-->

## 動作確認バージョン

- ホスト OS : Mac OS X 10.9.3
- VirtualBox 4.3.12
- Vagrant 1.6.3
- ansible 1.6.2
- ゲスト OS : Ubuntu 14.04 &times; 2

Ubuntu は https://cloud-images.ubuntu.com/vagrant/trusty/current/ のイメージを使ったので、
daily build の状態によっては動作が変わっているかもしれません。

## ホストオンリーネットワーク

使った後は

- vboxnet1 (192.168.50.1/24)
- vboxnet2 (192.168.11.1/24)
- vboxnet3 (192.168.12.1/24)

が勝手に増えているので、不要なら設定で消しておくと良いと思います。

ネットワークとしては以下のように 192.168.50.0/24 の部分で IPsec 接続をして、
192.168.11.11 と 192.168.12.12 をつなぐ、という感じにしています。
デフォルトの eth0 は外部への接続用としてそのままにしています。

```
192.168.11.11 (vm1:eth2)
       |
192.168.50.11 (vm1:eth1)
       |
192.168.50.12 (vm2:eth1)
       |
192.168.12.12 (vm2:eth2)
```

## 準備

Usage に書いてあるように準備をしておきます。
先日作った `ja_jp` role は git submodule にしているので、

```
% git submodule init
% git submodule update
```

で取得する必要があります。

## 試し方

`vagrant up`
すると
`/etc/ipsec-tools.conf`
と
`/etc/racoon/racoon.conf`
に設定が入っている状態になっているので、
始点アドレスを指定して
`ipsec-tools.conf`
に設定した経路を通るようにパケットを送ると
IPsec VPN がつながります。

`ping` コマンドのように直接始点アドレスを指定するオプションがない場合は
`ping -I eth2 192.168.12.12` のように network interface で指定すれば
良いようです。

## 動作確認

以下のコマンドの出力が接続前後で変わることが確認できます。

- `racoonctl -l show-sa isakmp`
- `racoonctl -l show-sa ipsec`
- `setkey -D`

racoon のログが `/var/log/syslog` に大量に出ているのを確認できます。
(`racoon.conf` で `log debug` にしているため)

## tshark でパケットの確認

`tshark` パッケージをインストールした後、
`sudo dpkg-reconfigure wireshark-common`
で一般ユーザーでも実行を許可するようにして、
実行を許可するユーザーを `wireshark` に追加します。

グループの追加を反映するためにログインし直すと、
`tshark -i eth1 -V 'port 500'`
などでパケットの確認ができるようになります。

暗号化されていて詳細はわかりませんが、
UDP の 500 番ポートで Informational のパケットが流れていることが確認できました。

GUI の wireshark の方では
http://ask.wireshark.org/questions/12019/how-can-i-decrypt-ikev1-andor-esp-packets
の方法で暗号化の解除もできるようです。

syslog の方で racoon のログをみると、
DPD のパケットらしいということが確認できました。
syslog の方で `#012` となっている部分がありますが、
これは rsyslog で改行が変換されたたもので空白として無視すれば良くて、
例えば

```
Jun  4 17:54:43 vm1 racoon: DEBUG: new cookie:#012831d33d46e20f8e6
Jun  4 17:54:43 vm1 racoon: DEBUG: final encryption key computed:
Jun  4 17:54:43 vm1 racoon: DEBUG: #012932d361f fc62ddc2 6164d513 d40d211f b7364166 232cf490
```

というログなら cookie は `831d33d46e20f8e6` になります。

鍵は `932d361ffc62ddc26164d513d40d211fb7364166232cf490` になると思ったのですが、
これを
Edit -> Preferences -> Protocols -> ISAKMP -> IKEv1 Decryption Table:
に設定すれば良いはずなのですが、試したところ decrypt されなかったので、
あっているのかどうかはわかりませんでした。
cookie の方は他のログなどで確認できたので、あっているはずです。
