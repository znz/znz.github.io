---
layout: post
title: "LC_COLLATEの問題でuniqで丸数字が同一視されてしまう"
date: 2013-10-31 23:50
comments: true
categories: linux debian ubuntu
---
`uniq -c` で重複がないのを確認しようとしたら、
丸数字のところだけ違う行が同一視されてしまって、
2以上になることがあって困ったので、
原因を調べてみました。

<!--more-->


## 現象

以下のように丸数字などが同一視されています。

```
$ cat n.txt
①
②
$ uniq n.txt
①
$ sort n.txt
①
②
$ tac n.txt | sort
②
①
$
```

```
$ cat n.txt
①
②
$ LANG=ja_JP.utf8 uniq -c n.txt
      2 ①
$ LANG=C uniq -c n.txt
      1 ①
      1 ②
$ uniq --version
uniq (GNU coreutils) 8.20
Copyright (C) 2012 Free Software Foundation, Inc.
ライセンス GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

作者 Richard M. Stallman および David MacKenzie。
$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 13.04
Release:        13.04
Codename:       raring
$
```

## 調査

`apt-get source locales`
でソースをとってきて調べてみると
`locales/ja_JP` の `LC_COLLATE` から `END LC_COLLATE`
の間に書いていないコードポイントは同一視されているように見えました。

これは glibc の問題だと思って、既に報告されているかどうか調べてみると
[Bug 13063 – 'sort -u' will erase some Chinese characters](https://sourceware.org/bugzilla/show_bug.cgi?id=13063)
に同じような話がありました。

## ML で聞いてみた

[linux-users: 108951](http://listserv.linux.or.jp/pipermail/linux-users/2013-October/000527.html)
で質問してみたところ、
[linux-users: 108952](http://listserv.linux.or.jp/pipermail/linux-users/2013-October/000528.html)
で返信があり、
`LC_COLLATE`
をちゃんと定義して解決するか、
単純にバイト順でソートしたいだけなら
`LC_COLLATE=C` で良いという話でした。

## 結論

結局今回の目的はソートではなく重複検査だったので、
`LC_COLLATE=C` で解決ということになりました。

誰か興味のある人は真面目に
`LC_COLLATE`
の定義に挑戦してみると良いのではないでしょうか。

それとは別に定義されていない文字を同一視せずにバイト順でも何でもいいので、
適当に別の文字として扱ってくれるようになればいいのに、
とは思いました。
誰かそういう方向での対応も挑戦してみると
他の言語も含めて幸せになれるのではないかと思いました。
