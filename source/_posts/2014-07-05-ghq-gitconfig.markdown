---
layout: post
title: "GOPATHとghqの設定を変更した話とgitconfigのコマンドでの設定の話"
date: 2014-07-05 09:55:07 +0900
comments: true
categories: go ghq git shell zsh
---
[ghqを使ったローカルリポジトリの統一的・効率的な管理について - delirious thoughts](http://blog.kentarok.org/entry/2014/06/03/135300 "ghqを使ったローカルリポジトリの統一的・効率的な管理について - delirious thoughts")
を参考にして、
go と ghq で同じディレクトリを使っていたのですが、
go で自動で追加ダウンロードされたものと
自分でダウンロードしたものが混ざるとわかりにくいと思ったので、
分けることにしました。

<!--more-->

## 対象バージョン

- go version go1.3 darwin/amd64
- git verseion 2.0.0
- Mercurial Distributed SCM (version 3.0.1)
- ghq version HEAD (2014-06-27 が最新コミットの状態)

## go などのインストール

[Homebrew](http://brew.sh/) を使って
`brew install go --cross-compile-common`
でインストールしました。

git と mercurial もインストールしておきました。

## シェルの設定 (GOPATH など)

bash と zsh の共通の設定として
[50gopath.sh](https://github.com/znz/dot-shell/blob/bb84c5aefc83eab5ce1841508abd726ee9db6577/profile.d/50gopath.sh "50gopath.sh")
で以下のように設定しています。

統合している時は go とか git とかの意味をかねて g だけにしていましたが、
今は go 専用にしています。

```sh 50gopath.sh
    if [ -z "${GOPATH:-}" ]; then
        export GOPATH=$HOME/g
        PATH=$PATH:$GOPATH/bin
    fi
```

その後に zsh 専用の追加設定で
[50gopath.zsh](https://github.com/znz/dot-shell/blob/bb84c5aefc83eab5ce1841508abd726ee9db6577/profile.d/50gopath.zsh "50gopath.zsh")
で以下のように設定しています。

GOPATH が PATH のように `:` 区切りで複数になる可能性を考慮してしまったのですが、
[The GOPATH environment variable](http://golang.org/doc/code.html#GOPATH "The GOPATH environment variable")
をみるとそういうことはなさそうなので、
`${^${(s/:/)GOPATH}}` は単純に `${GOPATH}` でも良さそうです。

そして fpath に `_ghq` のパスを追加して `ghq` の引数を補完できるようにしています。

```sh 50gopath.zsh
    path=( $path ${^${(s/:/)GOPATH}}/bin(N) )
    fpath=( $fpath ${^${(s/:/)GOPATH}}/src/*/*/ghq/zsh(N) )
```

## ghq のインストール

`go get -v github.com/motemen/ghq`
でインストールしました。

### `https://` の要不要

`ghq get` では `github.com` から書く時は `https://` が必須なのですが、
こちらは `https://` は不要というか、
むしろ付けるとエラーになりました。

## .gitconfig の設定

[git-config.sh](https://github.com/znz/dot-shell/blob/bb84c5aefc83eab5ce1841508abd726ee9db6577/git-config.sh "git-config.sh")
で `user.name` や `user.email` 以外の共通のものはまとめて設定しています。

### コマンドでの設定の追加削除

詳細はドキュメントをみてもらうとして、まとめておくと以下のようになります。

- 単独の設定 : `git config --global section.key value`
- 複数設定 : `git config --global section.key value` の後で `git config --global --add section.key value`
- 複数設定削除 : `git config --global --unset-all section.key` (ただしセクションが空になっても残る)
- セクションごと削除 : `git config --global --remove-section セクション`

### ghq の設定

`ghq.root` を `--unset-all` だけすると git-config.sh を実行する度に
ghq セクション、つまり `[ghq]` の行が増えていってしまったので、
`--remove-section` を使いました。
他に `ghq` セクションに設定を入れていたら、
それも消えてしまうので注意してください。

```sh
    # ghq section
    git config --global --remove-section "ghq" || :
    GHQ_ROOT="ghq.root"
    #git config --global --unset-all "$GHQ_ROOT" || :
    git config --global "$GHQ_ROOT" "$HOME/s"
    git config --global --add "$GHQ_ROOT" "$HOME/g/src"
```

これで `ghq get` は `$HOME/s` に入って
`ghq look` では両方見えるようになりました。

### github の URL 統一の話

github の URL は https に統一して、
gitconfig の設定で
push は ssh プロトコル、
pull などは git プロトコル (昔は Git Read-Only と書いてあった URL)
を使うようにしています。

```sh
    # github upload
    GITHUB_URL_PREFIX="url.git@github.com:"
    git config --global --remove-section "$GITHUB_URL_PREFIX" || :
    git config --global "$GITHUB_URL_PREFIX".pushInsteadOf "git://github.com/"
    git config --global --add "$GITHUB_URL_PREFIX".pushInsteadOf "https://github.com/"
    # gist upload
    git config --global "url.git@gist.github.com:".pushInsteadOf "https://gist.github.com/$(git config github.user)/"
    # github download
    git config --global url."git://github.com/".insteadOf "https://github.com/"
```

詳細は
[githubでhttpsのURLを指定してもgitプロトコルやssh経由にする方法](http://blog.n-z.jp/blog/2013-11-28-git-insteadof.html "githubでhttpsのURLを指定してもgitプロトコルやssh経由にする方法")
を参照してください。
