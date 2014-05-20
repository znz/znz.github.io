---
layout: post
title: "ansibleでppaを追加する"
date: 2014-05-20 22:47:22 +0900
comments: true
categories: ansible ubuntu
---
サーバーとしてインストールした Ubuntu だと
`add-apt-repository` コマンドが入っている
`python-software-properties` パッケージが入っていなくて
`ppa` の追加に困ることがありますが、
ansible を使えばサーバー自体に余計なパッケージを入れなくても
`ppa` の apt 設定を追加できました。

<!--more-->

## 動作確認バージョン

- クライアント側 ansible 1.6.1
- サーバー側 Ubuntu 12.04

## ansible コマンド直接

`apt_repository` モジュールの引数の `repo` に `ppa` を指定するだけです。

```console
ansible all -s -K -i ./inventory_hosts -m apt_repository -a "repo='ppa:chris-lea/node.js'"
```

## playbook.yml に書く

`- apt_repository: repo='ppa:chris-lea/node.js'`
のように書くだけです。
普通にインストールすればサーバーでも入っていると思うのですが、
`python-apt` に依存しているので、
`- apt: pkg=python-apt`
も書いておくと良いかもしれません。


[nodejs-ppa.yml](https://github.com/znz/ansible-playbook-passenger/blob/master/provisioning/roles/passenger/tasks/nodejs-ppa.yml)
が実際の使用例です。

```yaml nodejs-ppa.yml
---
- apt: pkg=python-apt
- apt_repository: repo='ppa:chris-lea/node.js'
  when: ansible_distribution_release == "precise"
```
