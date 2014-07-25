---
layout: post
title: "zshの機能のみで既に存在しないディレクトリをcdrのリストから削除する"
date: 2014-07-25 21:54:57 +0900
comments: true
categories: zsh cdr
---
[既に存在しないディレクトリを cdrのリストから削除する - Life is very short](http://d.hatena.ne.jp/syohex/20140425/1398394421 "既に存在しないディレクトリを cdrのリストから削除する - Life is very short")
をみて perl を使っていて、
ファイル名の変更にも対応していなくてポータブルではないと思ったので、
zsh の機能のみで実装してみました。

<!--more-->

## 動作確認バージョン

- zsh 5.0.2

## 実装

cdr の実装の中の
`chpwd_recent_add`
`chpwd_recent_dirs`
`chpwd_recent_filehandler`
の中を良くみてみると
引数なしで `chpwd_recent_filehandler` を呼び出すと
`$reply` に配列でディレクトリ一覧を返してくれて、
引数を渡すとファイルに保存してくれるとわかりました。

そこで、その間で `(N)` を使って存在しないディレクトリを除外すれば良いということで
以下の実装になりました。
`emulate -L zsh` などは参考にした部分にあったので、そのまま使っています。

```sh
my-compact-chpwd-recent-dirs () {
    emulate -L zsh
    setopt extendedglob
    local -aU reply
    integer history_size
    autoload -Uz chpwd_recent_filehandler
    chpwd_recent_filehandler
    history_size=$#reply
    reply=(${^reply}(N))
    (( $history_size == $#reply )) || chpwd_recent_filehandler $reply
}
```

使い方としては

- 必要に応じて手で `my-compact-chpwd-recent-dirs` を呼び出す
- `.zshrc` から起動時に `my-compact-chpwd-recent-dirs` を実行
- `add-zsh-hook chpwd my-compact-chpwd-recent-dirs` で毎回実行

などが考えられます。

[pecoとcdrなどを組み合わせてみた](http://blog.n-z.jp/blog/2014-07-17-peco-cdr.html "pecoとcdrなどを組み合わせてみた")ときに、
`peco` に渡す前のところで `(N-/)` でフィルタリングしていたので、
個人的には必要に応じて手で呼び出す使い方にしようと思っています。
