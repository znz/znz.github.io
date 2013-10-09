---
layout: post
title: "octopress をアップデートした (.ruby-version が conflict した) 話"
date: 2013-10-09 21:21
comments: true
categories: octopress
---
octopress で使う ruby を 1.9.3-p448 から 2.0.0-p247 にあげようと思って、
そのついでに octopress 自体も master の最新のものに更新したので、
その手順のメモです。

その時に `.ruby-version` が conflict したので、
その話がメインです。

<!--more-->

## `.ruby-version` とは?

rbenv が `.rbenv-version` から `.ruby-version` に変わった時には
知らなかったのですが、
[A Common .ruby-version File For Ruby Projects](https://gist.github.com/fnichol/1912050)
というものがあって、
rbenv や rvm のような切り替えツール共通で
`.ruby-version`
というファイルを使おうということのようです。

## octopress の .ruby-version

その発想は良いのですが、
rbenv + ruby-build だと 1.9.3-p448 や 2.0.0-p247 のように
パッチレベルまで `.rbenv-version` に入るのが普通なのに
rvm が作る `.ruby-version` だと `1.9.3` のように
パッチレベルが入らないようで、
octopress の `.ruby-version` で問題が起きました。


元々使い始めた時の octopress の master には
`.ruby-version`
ファイルが入っていて、
`1.9.3`
という内容でした。

手元での回避策としては、
`ln -snf 1.9.3-p448 ~/.rbenv/versions/1.9.3`
のようにシンボリックリンクで回避するとか、
`configure` で `--prefix=$HOME/.rbenv/versions/1.9.3` で
別途インストールするとかも考えたのですが、
他に影響が出るのが嫌だったので、
結局
`.ruby-version`
を書き換えて使うことにしました。

rbenv を使っていて、
リリースされているバージョンの ruby のインストールには
ruby-build を使っていて
`1.9.3-p448`
にする必要があったので、
`rbenv local 1.9.3-p448`
で置き換えました。

## バージョンアップ手順

まず remote を確認します。

```
 git remote -v
octopress       git://github.com/imathis/octopress.git (fetch)
octopress       git://github.com/imathis/octopress.git (push)
origin  git@github.com:znz/znz.github.io (fetch)
origin  git@github.com:znz/znz.github.io (push)
```

octopress という名前にしていることが確認出来たので、
fetch します。

```
% git fetch octopress
remote: Counting objects: 85, done.
remote: Compressing objects: 100% (34/34), done.
remote: Total 46 (delta 30), reused 26 (delta 10)
Unpacking objects: 100% (46/46), done.
From git://github.com/imathis/octopress
   9699df7..5a6b8e4  2.5        -> octopress/2.5
   f75547f..ff71657  master     -> octopress/master
   8d7ef9a..98bd6c9  site-2.1   -> octopress/site-2.1
```

merge する前に
`git diff source...octopress/master`
で変更点を確認してから merge します。

```
% git diff source...octopress/master
% git merge octopress/master
Auto-merging Rakefile
CONFLICT (modify/delete): .ruby-version deleted in octopress/master and modified in HEAD. Version HEAD of .ruby-version left in tree.
Automatic merge failed; fix conflicts and then commit the result.
zsh: exit 1     git merge octopress/master
```

conflict しました。
`.ruby-version`
は消してしまって
`rbenv global 2.0.0-p247`
で設定している `2.0.0-p247` を使う予定なので、
消してしまいます。
その変更を commit すると merge 完了でした。

```
% git rm .ruby-version
.ruby-version: needs merge
rm '.ruby-version'
% git commit -av
[source 136b37f] Merge remote-tracking branch 'octopress/master' into source
```

うまくいったので、バックアップをかねて github に push しておきます。

```
% git push
Counting objects: 55, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (39/39), done.
Writing objects: 100% (41/41), 4.90 KiB | 0 bytes/s, done.
Total 41 (delta 28), reused 0 (delta 0)
To git@github.com:znz/znz.github.io
   5dabdd2..136b37f  source -> source
```

rake のバージョンの関係で
`rake preview`
などではなく
`bundle exec rake preview`
のように実行することが必須になってしまいました。

```
% rake preview
rake aborted!
You have already activated rake 10.1.0, but your Gemfile requires rake 0.9.2.2. Using bundle exec may solve this.
```

## まとめ

事前に予測できた範囲の conflict しかなくて、
簡単に解決できました。

`.ruby-version`
は
`.gitignore`
には入っていないので、
余計な conflict を避けるために
`.gitignore`
を変更せずに使いたいのなら、
親ディレクトリに
`.ruby-version`
を用意してバージョンを指定するのが良いと思います。
`.git`
などと同じように
`.ruby-version`
もディレクトリを上がっていって最初に見つかったものが使われるので、
そういう運用が可能です。
