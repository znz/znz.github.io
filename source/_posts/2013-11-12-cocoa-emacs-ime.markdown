---
layout: post
title: "HomebrewのEmacsにIMEインラインパッチが取り込まれたので設定した"
date: 2013-11-12 18:30
comments: true
categories: emacs mac ime
---
[h12o さんのインラインパッチの pull request が取り込まれた](http://qiita.com/h12o@github/items/07f773909da5ebdb7b7c)
(コメント参照)
ということで、
`brew uninstall emacs` で一度アンインストールしてから
`brew install --cocoa --srgb --with-gnutls --japanese emacs -v`
でインストールして、
IME 関連の設定をしてみました。

<!--more-->

## IME 関連設定全体

以下のように設定してみました。

以降は設定内容を個別に解説していきます。

```scheme
(global-set-key (kbd "M-¥") (lambda () (interactive) (insert "\\")))

(static-when (eq window-system 'ns)
  (static-when (assoc "MacOSX" input-method-alist)
    ;; ime inline patch
    (setq default-input-method "MacOSX")
    (static-when t
      (mac-translate-from-yen-to-backslash)
      (global-set-key (kbd "M-¥") (lambda () (interactive) (insert "¥")))
      )

    (mapc
     (lambda (param)
       (let ((name (car param)))
         (cond
          ((string-match "Japanese\\(\\.base\\)?\\'" name) ;; ひらがなの日本語入力
           (mac-set-input-method-parameter name 'cursor-color "blue"))
          ((string-match "Japanese" name) ;; カナなどの日本語入力
           (mac-set-input-method-parameter name 'cursor-color "red"))
          ((string-match "Roman" name) ;; 英字
           (mac-set-input-method-parameter name 'cursor-color "black"))
          (t ;; その他
           (mac-set-input-method-parameter name 'cursor-color "yellow"))
          )
         ))
     mac-input-method-parameters)
    (mac-set-input-method-parameter "com.apple.keylayout.US" 'cursor-color "black")
    )
```

## 環境のチェック

`(eq window-system 'ns)`
で Cocoa Emacs かどうかチェックしています。

## パッチのチェック

`(boundp 'mac-input-method-parameters)`
でチェックしている例が多かったのですが、
`(assoc "MacOSX" input-method-alist)`
で真面目に `default-input-method` に設定できる値かどうか調べています。

`(boundp 'mac-input-method-parameters)`
の方が一般的なようなので、
今後使えなくなる可能性も低そうなので、
`alist` をたどる必要があって遅そうな `assoc` のチェックは
あまりおすすめできないです。

## Option+backslash

普段はデフォルトのままの「ことえり」を使っていて、
Linux での OADG109A と同じように `¥` キーはちゃんと yen sign の入力になるようにしています。

そして、
`Option+¥`
で backslash が入力できます。
外側の
`(global-set-key (kbd "M-¥") (lambda () (interactive) (insert "\\")))`
で Emacs でも同じ挙動になるようにしています。
インラインパッチがない環境だとこの挙動になります。
とりあえず入力できれば良いと思って作った設定なので、
prefix argument などには対応していません。
(`C-u M-¥` でも4文字ではなく1文字しか入らない)

ところが、これだと
`C-¥`
で
`toggle-input-method`
がきかなかったので、不本意ですが
`(mac-translate-from-yen-to-backslash)`
を使いました。
そして、逆に
`Option+¥`
で yen sign が入力できるようにするために
`(global-set-key (kbd "M-¥") (lambda () (interactive) (insert "¥")))`
と設定しています。

## カーソルの色の設定

IME 対応で一番期待していた部分がこれでした。

ddskk で入力状態によってカーソルの色を変えられるのが便利だったので、
TeraTerm Pro のように IME の状態によってカーソルの表示を変更できるものは
積極的に利用していたこともあって、ちょっと頑張って設定してみました。

`mac-input-method-parameters` の初期値の一部を抜粋すると以下のように設定されていて、
ほとんどの IME は「ことえり」と同じように
`.Japanese`
で終わっているものが一般的な日本語入力状態に見えたので、
青色にすることにしました。
「Google 日本語入力」はよくわかりませんが、
たぶん `.Japanese.base` で終わっているものが、
そうだろうと思って設定しました。

そしてカタカナなどの他の入力状態の時は赤色にしました。

それから、
`.Roman`
で終わっているものが、英字入力状態のようだったので、
黒色に戻すようにしました。

ちゃんと戻す設定も入れないと変わりっぱなしになってしまうので、
戻す設定も重要です。

最後にその他の日本語以外の IME のときは黄色にしました。

ネットで見つかる設定例では
`"com.apple.keylayout.US"`
にも
`mac-set-input-method-parameter`
をしているのですが、
`mac-input-method-parameters`
にはなかったので、
後で設定しています。
(先に設定すると、その他の黄色で上書きしてしまう。)

```scheme ns-win.el.gz
(defvar mac-input-method-parameters
  '(
    ("com.apple.inputmethod.Kotoeri.Roman"
     (title . "A")
     (cursor-color)
     (cursor-type))
    ("com.apple.inputmethod.Kotoeri.Japanese"
     (title . "あ")
     (cursor-color)
     (cursor-type))
    ("com.apple.inputmethod.Kotoeri.Japanese.Katakana"
     (title . "ア")
     (cursor-color)
     (cursor-type))
    ("com.apple.inputmethod.Kotoeri.Japanese.FullWidthRoman"
     (title . "Ａ")
     (cursor-color)
     (cursor-type))
    ("com.apple.inputmethod.Kotoeri.Japanese.HalfWidthKana"
     (title . "ｱ")
     (cursor-color)
     (cursor-type))
    ("com.apple.inputmethod.kotoeri.Ainu"
     (title . "アイヌ")
     (cursor-color)
     (cursor-type))
	...
    ("com.google.inputmethod.Japanese.Roman"
     (title . "G")
     (cursor-color)
     (cursor-type))
    ("com.google.inputmethod.Japanese.base"
     (title . "ぐ")
     (cursor-color)
     (cursor-type))
    ("com.google.inputmethod.Japanese.Katakana"
     (title . "グ")
     (cursor-color)
     (cursor-type))
    ("com.google.inputmethod.Japanese.FullWidthRoman"
     (title . "Ｇ")
     (cursor-color)
     (cursor-type))
    ("com.google.inputmethod.Japanese.HalfWidthKana"
     (title . "ｸﾞ")
     (cursor-color)
     (cursor-type))
    ("jp.monokakido.inputmethod.Kawasemi.Roman"
     (title . "K")
     (cursor-color)
     (cursor-type))
    ("jp.monokakido.inputmethod.Kawasemi.Japanese"
     (title . "か")
     (cursor-color)
     (cursor-type))
    ("jp.monokakido.inputmethod.Kawasemi.Japanese.Katakana"
     (title . "カ")
     (cursor-color)
     (cursor-type))
    ("jp.monokakido.inputmethod.Kawasemi.Japanese.FullWidthRoman"
     (title . "Ｋ")
     (cursor-color)
     (cursor-type))
    ("jp.monokakido.inputmethod.Kawasemi.Japanese.HalfWidthKana"
     (title . "ｶ")
     (cursor-color)
     (cursor-type))
    ("jp.monokakido.inputmethod.Kawasemi.Japanese.HalfWidthRoman"
     (title . "_K")
     (cursor-color)
     (cursor-type))
    ("jp.monokakido.inputmethod.Kawasemi.Japanese.Code"
     (title . "C")
     (cursor-color)
     (cursor-type))
    ("com.justsystems.inputmethod.atok21.Roman"
     (title . "A")
     (cursor-color)
     (cursor-type))
    ("com.justsystems.inputmethod.atok21.Japanese"
     (title . "あ")
     (cursor-color)
     (cursor-type))
    ("com.justsystems.inputmethod.atok21.Japanese.Katakana"
     (title . "ア")
     (cursor-color)
     (cursor-type))
    ("com.justsystems.inputmethod.atok21.Japanese.FullWidthRoman"
     (title . "英")
     (cursor-color)
     (cursor-type))
    ("com.justsystems.inputmethod.atok21.Japanese.HalfWidthEiji"
     (title . "半英")
     (cursor-color)
     (cursor-type))
    ("com.justsystems.inputmethod.atok22.Roman"
     (title . "A")
     (cursor-color)
     (cursor-type))
    ("com.justsystems.inputmethod.atok22.Japanese"
     (title . "あ")
     (cursor-color)
     (cursor-type))
    ("com.justsystems.inputmethod.atok22.Japanese.Katakana"
     (title . "ア")
     (cursor-color)
     (cursor-type))
    ...
```
