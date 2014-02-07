---
layout: post
title: "pre-commit hookでmasterへのcommitを禁止した"
date: 2014-02-07 16:57:54 +0900
comments: true
categories: git
---
git flow を使っていて、
`git flow release finish`
後に master ブランチのままなのに気付かずに
develop ブランチのつもりで git commit してしまうということが起きるので、
pre-commit hook で禁止するようにしてみました。

<!--more-->

## 参考

twitter で
https://github.com/bleis-tift/Git-Hooks/blob/rewrite/hooks/pre-commit
というのを教えてもらって、
Git.pm に依存しているこれをそのまま使うのは面倒そうだったので、
シェルスクリプトで実装してみました。

## pre-commit の引数

さきほどの pre-commit だと `my $gitdir = shift;` としていて、
pre-commit の第一引数で git-dir が渡されるのかと思ったのですが、
実際に試してみるとそんなことはなくて、
`.git/hooks/pre-commit.sample`
をみても `--git-dir` は付けていなかったので、
付けずに git コマンドを呼び出すことにしました。

## is_master_branch

`is_master_branch` の実装を GitHub の検索を使って探してみると
`git --git-dir="$dir" symbolic-ref HEAD 2>/dev/null`
の結果を
`s"refs/heads/""`
で処理して
`'master'`
と比較しているだけだったので、

```sh is_master_branch
if test "`git symbolic-ref HEAD | sed -e 's:^refs/heads/::'`" = master; then
	echo "cannot commit on master branch."
	exit 1
fi
```

とすることにしました。

pre-commit.sample を pre-commit にコピーして、
`exec 1>&2`
の下に追加しました。

## pre-commit 全体

元の pre-commit.sample からコピーした部分も含めて、
pre-commit hook 全体は以下のようになりました。

```sh .git/hooks/pre-commit
#!/bin/sh
#
# An example hook script to verify what is about to be committed.
# Called by "git commit" with no arguments.  The hook should
# exit with non-zero status after issuing an appropriate message if
# it wants to stop the commit.
#
# To enable this hook, rename this file to "pre-commit".

if git rev-parse --verify HEAD >/dev/null 2>&1
then
	against=HEAD
else
	# Initial commit: diff against an empty tree object
	against=4b825dc642cb6eb9a060e54bf8d69288fbee4904
fi

# If you want to allow non-ascii filenames set this variable to true.
allownonascii=$(git config hooks.allownonascii)

# Redirect output to stderr.
exec 1>&2

if test "`git symbolic-ref HEAD | sed -e 's:^refs/heads/::'`" = master; then
	echo "cannot commit on master branch."
	exit 1
fi

# Cross platform projects tend to avoid non-ascii filenames; prevent
# them from being added to the repository. We exploit the fact that the
# printable range starts at the space character and ends with tilde.
if [ "$allownonascii" != "true" ] &&
	# Note that the use of brackets around a tr range is ok here, (it's
	# even required, for portability to Solaris 10's /usr/bin/tr), since
	# the square bracket bytes happen to fall in the designated range.
	test $(git diff --cached --name-only --diff-filter=A -z $against |
	  LC_ALL=C tr -d '[ -~]\0' | wc -c) != 0
then
	echo "Error: Attempt to add a non-ascii file name."
	echo
	echo "This can cause problems if you want to work"
	echo "with people on other platforms."
	echo
	echo "To be portable it is advisable to rename the file ..."
	echo
	echo "If you know what you are doing you can disable this"
	echo "check using:"
	echo
	echo "  git config hooks.allownonascii true"
	echo
	exit 1
fi

# If there are whitespace errors, print the offending file names and fail.
exec git diff-index --check --cached $against --
```
