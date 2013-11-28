---
layout: post
title: "githubでhttpsのURLを指定してもgitプロトコルやssh経由にする方法"
date: 2013-11-28 22:22
comments: true
categories: git github
---
要約すると `url.<base>.inteadOf` や `url.<base>.pushInsteadOf`
を使えば良いという話です。
github 以外でも使えます。

<!--more-->

## 設定例

自分では今のところ、以下の設定にしています。

最初の
`[url "git@github.com:"]`
のセクションは URL の指定として
`https:` や `git:` を使っていても
`git push` のときには `ssh` 経由にする、
という意味になります。

次の
`[url "git://github.com/"]`
は、その他の `git fetch` や `git pull` の時は
`https:` の代わりに `git:` を使うという意味になります。

```ini ~/.gitconfig
[url "git@github.com:"]
	pushInsteadOf = git://github.com/
	pushInsteadOf = https://github.com/
[url "git://github.com/"]
	insteadOf = https://github.com/
```

## pushInsteadOf なしの場合

`pushInsteadOf` を指定せずに `insteadOf` だけの場合は
`git push` の時も `insteadOf` の設定が使われます。

例えば設定例とは逆に、
proxy を通さないといけないとかの理由で
すべて https に統一したいのなら
以下のような設定になると思います。

```ini ~/.gitconfig
[url "https://github.com/"]
	insteadOf = git@github.com:
	insteadOf = git://github.com/
```

## コマンドでの設定方法

`git config` コマンド経由で `pushInsteadOf` を設定するなら、
以下のように指定します。

`~/.gitconfig` に `github.token` などの秘密にすべき情報が入っていて
共有しにくい時にはコマンドで設定できた方が便利です。

```bash
GITHUB_URL_PREFIX="url.git@github.com:"
git config --global --remove-section "$GITHUB_URL_PREFIX" || :
git config --global       "$GITHUB_URL_PREFIX".pushInsteadOf "git://github.com/"
git config --global --add "$GITHUB_URL_PREFIX".pushInsteadOf "https://github.com/"
```

## 設定確認方法

実際にどういう URL が使われるのかは
`git remote -v`
で確認できます。

`.git/config` の `remote` の `url` と見比べると
ちゃんと設定が反映されていることがわかると思います。

```console
$ git remote -v
origin	git://github.com/znz/rbenv-plug.git (fetch)
origin	git@github.com:znz/rbenv-plug.git (push)
$ cat .git/config
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
	ignorecase = true
	precomposeunicode = false
[remote "origin"]
	url = https://github.com/znz/rbenv-plug.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
```

## URL の選ばれ方

github の URL で説明していますが、
git の機能なので他の URL でも当然使えます。

URL は最初の部分だけ見て、最長一致するものが使われるようです。
git 1.8.4.3 の `git config --help` に
`When more than one insteadOf strings match a given URL, the longest match is used.`
と書いてあります。

## まとめ

`~/.gitconfig` での記述例の話はよくあるのですが、
コマンドでの設定方法とか、
設定の確認方法を見かけたことがなかったので、
まとめてみました。

そして github 以外でも使えるという話も付け加えておきました。
