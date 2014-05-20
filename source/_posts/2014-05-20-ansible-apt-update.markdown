---
layout: post
title: "ansible で sources.list を変更したときに update する"
date: 2014-05-20 22:58:09 +0900
comments: true
categories: ansible ubuntu debian
---
ansible で sources.list を変更したときには
`apt-get update` して欲しいのですが、
毎回 update するのは無駄なので、
`cache_valid_time` も使いたいと思ったので対処しました。

<!--more-->

## 動作確認バージョン

- クライアント側 ansible 1.6.1
- サーバー側 Ubuntu 12.04

## やり方

`register` で `template` モジュールの実行結果を受けて、
`changed` の場合は必ず update して、
`skipped` の場合は `cache_valid_time` で情報が古い場合だけ update するようにしました。

{% raw %}
```yaml tasks/apt.yaml
---
- template: src=sources.list.{{ ansible_distribution }}.j2 dest=/etc/apt/sources.list owner=root group=root mode=0644
  register: apt_sources_list
- apt: update_cache=yes
  when: apt_sources_list|changed
- apt: update_cache=yes cache_valid_time=3600
  when: apt_sources_list|skipped
```
{% endraw %}

## テンプレートの内容

gathering facts で設定された `ansible_distribution_release` と
`vars` で別途設定した `apt_ubuntu_uri` と `apt_ubuntu_components` を
組み合わせて apt-line を作るようにしました。

Ubuntu をインストールした直後の sources.list では
components が別の行になっているものもあったのですが、
1 行にまとめました。

{% raw %}
```text templates/sources.list.Ubuntu.j2
deb {{ apt_ubuntu_uri }} {{ ansible_distribution_release }} {{ apt_ubuntu_components | join(" ") }}
deb-src {{ apt_ubuntu_uri }} {{ ansible_distribution_release }} {{ apt_ubuntu_components | join(" ") }}
deb {{ apt_ubuntu_uri }} {{ ansible_distribution_release }}-updates {{ apt_ubuntu_components | join(" ") }}
deb-src {{ apt_ubuntu_uri }} {{ ansible_distribution_release }}-updates {{ apt_ubuntu_components | join(" ") }}
#deb {{ apt_ubuntu_uri }} {{ ansible_distribution_release }}-backports {{ apt_ubuntu_components | join(" ") }}
#deb-src {{ apt_ubuntu_uri }} {{ ansible_distribution_release }}-backports {{ apt_ubuntu_components | join(" ") }}
deb http://security.ubuntu.com/ubuntu {{ ansible_distribution_release }}-security {{ apt_ubuntu_components | join(" ") }}
deb-src http://security.ubuntu.com/ubuntu {{ ansible_distribution_release }}-security {{ apt_ubuntu_components | join(" ") }}
```
{% endraw %}

`backports` は使っていなかったのでコメントアウトしていますが、
`deb-src` も使わないのならコメントアウトしておいても良さそうです。

## `vars` の内容

例として jaist ミラーを使うようにしました。

```yaml vars/main.yml
---
apt_ubuntu_uri: "http://ftp.jaist.ac.jp/pub/Linux/ubuntu/"
apt_ubuntu_components:
- main
- restricted
- universe
- multiverse
```
