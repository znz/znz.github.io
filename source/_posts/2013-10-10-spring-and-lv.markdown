---
layout: post
title: "spring rails console で PAGER=lv だと /dev/tty: Device not configured"
date: 2013-10-10 21:28
comments: true
categories: ruby rails tty
---
`spring rails console`
で
`pry`
を使っていて
pager
を起動するような出力をしたときに
`/dev/tty: Device not configured`
になることがあったので原因を調べてみました。
結論としてはタイトルに書いてありますが、
`spring`
と
`lv`
の組み合わせが原因でした。

<!--more-->

## 確認バージョン

関係する gem のバージョンは以下の通りです。

* pry 0.9.12.2
* pry-rails 0.3.2
* rails 3.2.14, 4.0.0
* spring 0.0.10, 0.0.11

関係するプログラムのバージョンは以下の通りです。

* ruby 2.0.0-p247
* lv 4.51

環境は Mac OS X 10.7.5 です。

## 原因の切り分け

まず
`rails console`
だと問題は発生しないので、
`spring`
が原因のひとつなのは確実になって、
当面の回避策として、
`spring rails console`
の代わりに
`rails console`
を使っていました。

次にふと他の pager を使ってみるとどうだろうと思って、
`spring rails console`
の中で
`ENV['PAGER']="less"`
として `lv` から `less` に変えて試してみたところ、
問題なく使えました。
これがきっかけで `lv` も原因のひとつだということに気づきました。

## 原因調査

ここまでわかれば原因は調べやすいので、
まずエラーメッセージを出しているところを探しました。
これはすぐに見つかって、
`lv` の `src/stream.c` の `perror( "/dev/tty" )` でした。

```c lv451/src/stream.c
  close( 0 );
  if( IsAtty( 1 ) && 0 != open( "/dev/tty", O_RDONLY ) )
    perror( "/dev/tty" ), exit( -1 );
```

STDIN を開き直している処理があって、
ここで `"/dev/tty"` を開けないのが原因でした。

`spring rails console`
の中で直接
`open("/dev/tty","r")`
を試しても同様に
`Errno::ENXIO: Device not configured - /dev/tty`
になってしまうので、
`lv` の方を変えない限りどうしようもなさそうです。

というわけで、
これはもう `lv` の処理が `spring` と相性が悪いということで、
`spring` の中では `lv` を避けるしかなさそうです。

## 結論

`~/.spring.rb`
で
`ENV["PAGER"]`
を書き換えることにしました。

```ruby ~/.spring.rb
ENV["PAGER"] = "less" if ENV["PAGER"] == "lv"
```

## 余談

原因を調べているときに
`spring`
の標準入出力の処理周りをみていたのですが、
`UNIXSocket#send_io`
と
`UNIXSocket#recv_io`
で受け渡していました。

`recv_io` で mode を指定していないので、
`spring rails console` では
`STDIN.write` が使えたり `STDOUT.gets` が使えたりしてしまうようです。

これは `recv_io(IO, "r")` などに直せば良さそうに見えますが、
特に実害もなさそうなので、このままでもあまり問題はなさそうです。
