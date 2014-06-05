---
layout: post
title: "ansible-galaxy を使ってみた"
date: 2014-06-02 22:00:00 +0900
comments: true
categories: ansible git github
---
ansible の複数の playbook で role を共有するのに
[Ansible Galaxy](https://galaxy.ansible.com/)
はどうだろうと思って登録してみました。

<!--more-->

## 動作確認バージョン

- ansible 1.6.2

## アカウント登録

後で github から role を import するということもあって、
github 連携でアカウントを登録しました。

アカウントは4文字以上必須ということで `znzj` にしました。
([docker index](https://index.docker.io/) とか
[Vagrant Cloud](https://vagrantcloud.com/) とか
最近は4文字以上必須が多いのでしょうか?)

## role を github に公開

`ansible-galaxy init ansible-role-ja_jp` でひな形を作成して、
`provisioning/roles/ja_jp` というディレクトリの中身だったものを移行して
https://github.com/znz/ansible-role-ja_jp として公開しました。

apt のミラーサイトとして日本のミラーを使ったり、日本語 locale にしたり、タイムゾーンを JST にしたりする role です。

## Ansbile Galaxy に role を追加

ログインした状態だと右上の方に出てくる `Add a Role` というリンクから登録します。

- Github Username/Account : znz
- Github Repository : ansible-role-ja_jp
- Alternate Name : ja_jp

という内容で登録したところ、
https://galaxy.ansible.com/list#/roles/967
で見えるようになりました。

URL をみるとわかるように JavaScript オフだと個別の role は見えません。
ちょっと試した感じだと代用の URL も見つけられませんでした。

## 動作確認

`ansible-galaxy install znzj.ja_jp` だと `/usr/local/etc/ansible/roles/znzj.ja_jp` にインストールされてしまった (Homebrew でインストールした ansible を使っている場合) ので、
`ansible-galaxy install --roles-path ./roles znzj.ja_jp`
でカレントディレクトリの中にインストールし直しました。

## 感想

これだけだと git submodule に比べて利点が見えてこないのですが、
依存関係やバージョン指定をちゃんと使えれば便利そうに思いました。

自分が登録した role にバージョンを指定する方法がわからず、
今のところバージョンなしのままなので、どうにかしたいです。

## 参考サイト

- [AnsibleWorks Galaxyにroleを登録してみる - Qiita](http://qiita.com/kiri/items/f329a35543022d6e45d0)
  - スクリーンショットをみると AnsibleWorks Galaxy になっているところが今は Ansible Galaxy なので、いつの間にか名前が変わっているようです。
