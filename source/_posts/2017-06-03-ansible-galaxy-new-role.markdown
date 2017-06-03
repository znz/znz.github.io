---
layout: post
title: "ansible で role を新規作成して ansible galaxy で公開して更新した"
date: 2017-06-03 23:30:00 +0900
comments: true
categories: ansible debian ubuntu
---
ansible で role を新規作成して ansible galaxy で公開して更新するときにやっているいつもの手順を紹介します。

<!--more-->

## 対象バージョン

- ansible 2.3.0.0

## ansible-galaxy init

`ansible-galaxy init` でファイルを作成します。

    % ansible-galaxy init ansible-role-unattended-upgrades
    - ansible-role-unattended-upgrades was created successfully
	% cd ansible-role-unattended-upgrades
    % find . -type f | sort
    ./README.md
    ./defaults/main.yml
    ./handlers/main.yml
    ./meta/main.yml
    ./tasks/main.yml
    ./tests/inventory
    ./tests/test.yml
    ./vars/main.yml

`tests` は使い方がよくわからないのですが、そのままにしています。

## LICENSE 作成

すでに作成済みの role から The MIT License (MIT) のファイルをコピーしてきました。
新規の場合は github に push した後にブラウザーから作成するのが簡単だと思います。

## README 更新

ここで自分の他の role を参考に書き換えました。

## vars 削除

上書きしやすいように、 `defaults/main.yml` しか使っていないので `vars/main.yml` は削除しました。

    % rm -r vars

## 中身作成

`tasks`, `defaults`, `handlers`, `files`, `templates` などのディレクトリを使って作成します。

## meta/main.yml 更新

`meta/main.yml` を更新します。
`galaxy_tags` は https://galaxy.ansible.com/ の browse roles を参考にして適当に選んでいます。

## examples

テスト用に vagrant で serverspec を動かせるように `examples` を入れています。
どうするのが良いのかわかっていないのですが、とりあえず全くテストしないよりはましなので、こういう方法をとっています。

`debian/wheezy64` の box は synced folder が rsync なのでシンボリックリンクのループでエラーになってしまうので、 `vagrant up wheezy64`, `vagrant provision wheezy64`, `rake spec:wheezy64` のように provision を別途実行しないといけないのが不便なのですが、 https://atlas.hashicorp.com/debian/ には VirtualBox Guest Addition が入った box が `debian/contrib-jessie64` しかないので、今のところ wheezy と stretch ではどうしようもなさそうです。

テストに使った VM はディスクの無駄なので、こまめに vagrant destroy しています。

## git push

`basename $(pwd) | pbcopy` した名前で github に repository を作成します。
タグもうっておきます。

    git remote add origin git@github.com:znz/ansible-role-unattended-upgrades.git
    git push -u origin master
    git tag v1.0.0
    git push --tags

## ansible galaxy に反映

github 連携でログインして、 my roles を開きます。

Search Roles の入力欄の右にあるボタンをクリックして refresh して github の新しい repository を表示させます。
(たぶん `meta/main.yml` をチェックして一覧に出すかどうか決めているのだと思います。)

追加した role を有効にします。
Role Settings を開くとわかるのですが、なぜか Role Name は自動的に `ansible-role-` がとれて `unattended-upgrades` になっています。

## role の更新

role を更新したら [セマンティック バージョニング](http://semver.org/lang/ja/) にそってバージョン番号をあげて、タグをうって push しておきます。

my roles のページで該当する role の行の一番右にある Import Role をクリックすると新しいタグが反映されます。
(role 個別ページには該当する操作はなさそうです。)

## まとめ

ansible で新規 role を作成して、 github と ansible galaxy で公開して更新しているときにやっている手順を紹介しました。

昔は ansible-galaxy で role を取ってくるのに ansible galaxy への登録が必須だったので登録していたのですが、最近は他の人が role を作るときに参考になるかもと思って登録しています。
あまり汎用性のない自分用の role の場合は github だけに登録して YAML ファイルで

```
- src: https://github.com/znz/ansible-role-nadoka
  version: master
  name: znz.nadoka
```

のように指定して使っていたり (この場合でも meta/main.yml は必要)、 playbook 用の repository の role 以下にそのまま入れていたり (この場合は meta/main.yml は不要) します。

複数 role を登録していて、こういうフローで作業をしている人もいるということで、参考になれば幸いです。
