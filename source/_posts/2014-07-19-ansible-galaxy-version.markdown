---
layout: post
title: "ansible-galaxy用のroleにversionを付ける"
date: 2014-07-19 23:30:00 +0900
comments: true
categories: ansible github
---
[Ansible Galaxy](https://galaxy.ansible.com/ "Ansible Galaxy")
に登録されている role を使う時にバージョンを指定する方法は書いてあるのに、
自分で登録した role にバージョンを付ける方法がわからなかったので、
既にバージョンが付いているものを参考にして調べてみました。

<!--more-->

## 結論

[Ansible Galaxy の intro](https://galaxy.ansible.com/intro)
の最後の方に
「If you have applied any tags in your repo, Ansible Galaxy will automatically make a “version” object for each tag. This means users will be able to choose which version (tag) to download.」
と書いてあるように github で tag (release) を作るだけでした。

## バージョンの命名規則

同じページのバージョン指定の例が

    user1.role1,v1.0.0
    user2.role2,v0.5
    ...

となっているので、頭に `v` を付けて `v0.1.2` のようなバージョンの付け方にするのが良さそうです。

