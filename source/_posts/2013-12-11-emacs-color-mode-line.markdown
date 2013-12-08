---
layout: post
title: "emacsのmode-lineのminor-modeなどに色をつける"
date: 2013-12-11 00:00
comments: true
categories: emacs
---
emacs の mode-line の minor mode の表示が長いので短くしているのですが、
単純に短くするとわかりにくいので、色もつけています。

ところが、変数で設定できるようになっているものだと単純に
`propertize` して `setq` するだけでは色がつかなかったので、
その対処も含めてまとめてみました。

この投稿は
[.emacs Advent Calendar 2013](http://qiita.com/advent-calendar/2013/dot-emacs)
の11日目の記事です。

<!--more-->

## 単純に短くする

単純に短くするには `minor-mode-alist` の文字列を直接変更してしまうだけです。

```common-lisp
(when (consp (assq 'abbrev-mode minor-mode-alist))
  (setcar (cdr (assq 'abbrev-mode minor-mode-alist)) " ab"))
(when (consp (assq 'auto-fill-function minor-mode-alist))
  (setcar (cdr (assq 'auto-fill-function minor-mode-alist)) " Fil"))
```

## 色も付ける

`propertize` で `face` を付けた文字列を設定すると色が付きます。

`face` プロパティの設定としては、
最低限 `:foreground` と `:background` を知っておけば良いと思います。
指定できる色は `list-colors-display` で一覧できます。

```common-lisp
(when (consp (assq 'abbrev-mode minor-mode-alist))
  (setcar (cdr (assq 'abbrev-mode minor-mode-alist))
	  (propertize "省" 'face '(:foreground "green"))))
(when (consp (assq 'auto-fill-function minor-mode-alist))
  (setcar (cdr (assq 'auto-fill-function minor-mode-alist))
	  (propertize "詰" 'face '(:foreground "yellow"))))
```

## 設定対象の探し方

ここでは例として `abbrev-mode` と `auto-fill-mode` をあげていますが、
`auto-fill-mode` が `auto-fill-function` になっているなど、
単純に決まっているものだけではなさそうなので、
他の設定をどうするのかは、
`describe-variable` で `minor-mode-alist` をみて探すのが良さそうでした。

## 設定用関数

毎回長々と書くのは面倒なので、以下の関数を定義して使っています。

```common-lisp
(defun my-shorten-minor-mode-name (mode-sym short-name &optional face)
  "minor-modeの名前を短くする。"
  (let ((cell (assq mode-sym minor-mode-alist)))
    (when (consp cell)
      (if face
          (setq short-name (propertize short-name 'face face))
        (setq short-name (concat " " short-name)))
      (setcar (cdr cell) short-name))
    ))
```

## 起動時に読み込まれていない minor-mode の設定

`view-mode` のように起動時に読み込まれていない `minor-mode` は
`.emacs` のタイミングで書き換えようとしても
`minor-mode-alist` に登録されていないのでうまくいきません。
こういう場合は `eval-after-load` で設定しています。

```common-lisp
(eval-after-load "view"
  '(my-shorten-minor-mode-name
    'view-mode "見" '(:foreground "white" :background "DeepPink1")))
```

## 変数で設定できる場合

`eldoc-minor-mode` のように変数で設定できるようになっているものがあります。
こういう場合は単純に `propertize` した文字列を `setq` しても
プロパティが無視されて色がつきません。

理由は `mode-line-format` のドキュメントの
[23.4.6 Properties in the Mode Line](http://www.gnu.org/software/emacs/manual/html_node/elisp/Properties-in-Mode.html#Properties-in-Mode)
に書いてあって、
`risky-local-variable` が non-nil じゃないとテキストプロパティが
無視されるということでした。

そこで `risky-local-variable` に `t` を設定すれば色がつくようになりました。

```common-lisp
(put 'eldoc-minor-mode-string 'risky-local-variable t)
(setq eldoc-minor-mode-string
      (propertize "d" 'face '(:foreground "purple" :background "yellow")))
```

## anzu.el での例

他の例も挙げておくと
[anzu.el](http://qiita.com/syohex/items/56cf3b7f7d9943f7a7ba)
なら以下のように設定しています。

```common-lisp
  (when (require 'anzu nil t)
    (put 'anzu-mode-lighter 'risky-local-variable t)
    (setq anzu-mode-lighter (propertize "杏" 'face 'anzu-mode-line))
    (global-anzu-mode +1)
    )
```

## まとめ

ここでは `minor-mode` を中心に `mode-line` を短くして、
色をつける方法に付いて紹介しました。
`major-mode` も含めて実際に使っている設定は
[50mode-line.el](https://github.com/znz/dot-emacs/blob/8434c73ba833791eedc1411360e10441e52b370e/init.el.d/50mode-line.el)
に公開しているので、参考にしてください。
