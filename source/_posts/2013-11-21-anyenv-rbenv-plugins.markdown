---
layout: post
title: "anyenvやrbenvのpluginsを更新などをしやすくするプラグインを作った"
date: 2013-11-21 18:25
comments: true
categories: ruby rbenv github anyenv
---
[rbenv](https://github.com/sstephenson/rbenv)
には
[rbenv-update](https://github.com/rkh/rbenv-update)
というプラグインがあって、アップデートだけは簡単にできるのに、
他の `git` の操作をまとめて実行するのが面倒なので、
まとめて実行できる
[rbenv-git](https://github.com/znz/rbenv-git)
というプラグインを作りました。

それから
[anyenv](https://github.com/riywo/anyenv)
の方でもすべての `**env` も含めてアップデートできると便利だと思い、
[anyenv-update](https://github.com/znz/anyenv-update)
と
[anyenv-git](https://github.com/znz/anyenv-git)
を作成しました。

<!--more-->

## インストール

それぞれ `plugins` のディレクトリの中に `git clone` でとってくるだけです。

- `git clone https://github.com/znz/rbenv-git.git $(rbenv root)/plugins/rbenv-git`
- `git clone https://github.com/znz/anyenv-update.git $(anyenv root)/plugins/anyenv-update`
- `git clone https://github.com/znz/anyenv-git.git $(anyenv root)/plugins/anyenv-git`

`README` の方には `mkdir -p $(anyenv root)/plugins` も書いていますが、
git version 1.8.4.3 だと不要で、自動で親ディレクトリも作ってくれるようです。
git のどのバージョンからなのかわからないので、
`README` には `mkdir -p` を残しています。

## 使い方

### rbenv git

`rbenv` と `rbenv` のプラグインの `git` 操作をまとめて実行できます。

`rbenv git pull` で `rbenv update` の代用が出来ます。
`rbenv update` だと `rbenv` 自体を homebrew で入れている時にも
`RBENV_ROOT` で `git` コマンドを実行してしまいますが、
`rbenv git` なら大丈夫です。

`rbenv git gc` で cleanup も出来ます。

`rbenv git remote -v` でどこからとってきたのか確認したり、
`rbenv git status` でローカルで何か変更しているかどうか確認したりも出来ます。

### anyenv update

`anyenv update` で

- `anyenv`
- `anyenv` のプラグイン
- `**env`
- `**env` のプラグイン

がアップデートできます。

git 管理ではないものは skip します。

### anyenv git

`rbenv git` と同様の操作が
`anyenv update` と同様の対象に
まとめて実行できます。

- `anyenv git pull`
- `anyenv git gc`
- `anyenv git remote -v`
- `anyenv git status`

など、
`git` の各種操作ができます。

## 裏話

最初は `rbenv update` のコードを fork しようとしていたのですが、
ライセンスが明記されていなかったので止めて、
他の MIT License と明示されているプラグインを主に参考にして作りました。
他にも `set -eo pipefail` というのは
[dokku](https://github.com/progrium/dokku)
を参考にしました。

ライセンスが不明だと参考にするのにも困るので、
github のレポジトリを作成する時の LICENSE ファイルそのままだけでも良いので、
どういうライセンスにしたいのか明記されているとありがたいと思いました。
