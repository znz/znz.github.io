---
layout: post
title: "capistrano 3.3.3 が依存している capistrano-stats について"
date: 2014-12-01 21:33:21 +0900
comments: true
categories: ruby rails capistrano
---
capistrano が 3.3.3 にあがって
[capistrano-stats](https://github.com/capistrano/stats)
という gem に依存するようになりました。
これは `metrics.capistranorb.com:1200` にバージョン情報などを送信して
capistrano のサポートの改善に役立てようとするもののようです。

<!--more-->

## 対象バージョン

- capistrano 3.3.3
- capistrano-stats 1.0.3

## Capfile locked

capistrano-stats とは関係ないですが、
`Capfile locked at 3.2.1, but 3.3.3 is loaded`
と出るときは `config/deploy.rb` の最初の方にある
`lock '3.2.1'`
という行を
`lock '3.3.3'`
に書き換えれば OK です。

## capistrano-stats 1.0.3 の挙動

初回実行時に
`Do you want to enable statistics? (y/N): `
ときかれて、
そのまま Enter か `n` を入力すると ruby や capistrano のバージョンは送信されなくなるのですが、
(少なくとも私の) 期待に反して
`metrics.capistranorb.com:1200`
への送信自体は発生するようになっています。

## 何が問題か

全く送信してほしくない場合にも通信が発生したり、
firewall で塞がれている場合にエラーになったり
という問題があります。

## 一時的に無効にするには?

`CAPISTRANO_METRICS=localhost:1200 bundle exec cap default`
のように環境変数 `CAPISTRANO_METRICS` で送っても無視されるところに送信すれば良さそうです。

## 完全に無効にするには?

pull request 1 の
[Honor the users privacy decision and do not send UDP packets if he does not want to send metrics](https://github.com/capistrano/stats/pull/1 "Honor the users privacy decision and do not send UDP packets if he does not want to send metrics")
にあるように

```ruby
Rake::Task['metrics:collect'].clear_actions
```

を `config/deploy.rb` に追加すると送信されなくなりました。

## 今後

[Allow users to opt-out of sending the UDP ping and improve error handling when the sendto syscall fails](https://github.com/capistrano/stats/pull/2 "Allow users to opt-out of sending the UDP ping and improve error handling when the sendto syscall fails")
という pull request が続きで出ているので、
将来のバージョンでは全く送信しない選択肢が用意されるかもしれません。
