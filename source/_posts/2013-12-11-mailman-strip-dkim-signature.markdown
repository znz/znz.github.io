---
layout: post
title: "mailmanでDKIM-Signatureヘッダを削除する"
date: 2013-12-11 12:30
comments: true
categories: mailman debian ubuntu
---
fml から移行した mailman で Subject の書き換える設定をしていると、
元の `DKIM-Signature` が残っていると受け取った側で
メールを改ざんしているのと区別がつかないので、
`DKIM` の検証に失敗してしまいます。

そこで mailman で `DKIM-Signature` を削除するように設定しました。

<!--more-->

## mailman の設定

`REMOVE_DKIM_HEADERS` という設定があったので、
`/etc/mailman/mm_cfg.py` の末尾に

```
REMOVE_DKIM_HEADERS = Yes
```

という設定を追加して、
`sudo service mailman restart`
しました。

`mm_cfg.py` での設定なので、
ML 個別の設定ではなく
mailman 全体の設定になるようです。

## 参考

[Bug #557493 “Mailman must not strip DKIM-Signature headers” : Bugs : GNU Mailman](https://bugs.launchpad.net/mailman/+bug/557493)
という話があって、設定が追加されたようです。

`mm_cfg.py` で設定できる項目の一覧は
`/usr/lib/mailman/Mailman/Defaults.py`
にあります。
このファイルを良く見ていれば、もっと早くこの設定項目に気付けたのかもしれません。
