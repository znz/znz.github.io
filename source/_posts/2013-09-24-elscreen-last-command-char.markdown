---
layout: post
title: "elscreen で Symbol's value as variable is void: last-command-char"
date: 2013-09-24 13:50
comments: true
categories: emacs elscreen
---
elscreen で
`Symbol's value as variable is void: last-command-char`
というエラーになったので、
[elscreen: From Emacs 24.3: Symbol's value as variable is void: last-command-char](http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=705436)
のパッチを参考にして解決しました。

<!--more-->

`Symbol's value as variable is void: last-command-char`
で検索すると
[Emacs 24.3 は last-input-char, last-command-char が削除されています](http://oku.edu.mie-u.ac.jp/~okumura/texwiki/?YaTeX)
という話が一番目に見つかって、
`last-command-char elscreen`
で検索すると
[elscreen: From Emacs 24.3: Symbol's value as variable is void: last-command-char](http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=705436)
が一番目に見つかって解決方法がわかりました。

パッチを見てみると、
`xemacs`
の時は
`(event-to-character last-command-event)`
で、それ以外は
`last-command-event`
となっていて、
手元では GNU Emacs だけ対応出来れば良いので、
[defadvice で last-command-event を last-command-char に設定](https://github.com/znz/dot-emacs/commit/12599e870f9b1a5b4411ef875c3cb647ef3096dd)
することにしました。

```scheme
  (defadvice elscreen-jump (around elscreen-last-command-char-event activate)
    (let ((last-command-char last-command-event))
      ad-do-it))
```
