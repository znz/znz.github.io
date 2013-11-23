---
layout: post
title: "postfixとmilter-managerの設定をした"
date: 2013-11-23 21:25
comments: true
categories: debian linux postfix milter-manager
---
Debian 7.2 (amd64) のサーバーにメールサーバーの設定をしたところ、
[milter-manager](http://milter-manager.sourceforge.net/)
関連と gmail への IPv6 経由でのメール送信関連でちょっとひっかかりましたが、
すぐに解決できました。

<!--more-->

## milter-manager の話

まず `milter-manager` は
[Debianへインストール - milter manager](http://milter-manager.sourceforge.net/reference/ja/install-to-debian.html)
の手順通りに設定してみたのですが、確認のところで、

```
 $ sudo -u postfix milter-test-server -s unix:/var/spool/postfix/milter-manager/milter-manager.sock
 status: temporary-failure
 elapsed-time: 0.007781 seconds
```

のように `temporary-failure` になりました。
そこで依存しているデーモンを調べてみたところ、
`clamd` が起動していなかったので、

    sudo service clamav-daemon start

で起動した後、

    sudo service clamav-milter restart

で clamav-milter も再起動したら

    status: accept

になりました。

後で気付いたので、最初のインストール時のメッセージを
ちゃんと確認できていないのですが、
`/etc/init.d/clamav-daemon` の `start`
の処理に cvd ファイルなどの存在をチェックして起動を止める処理があるので、
`freshclam` の処理を待ってから起動しないとダメだったようです。

## gmail と IPv6 の話

[gmail の逆引き制限](http://ya.maya.st/d/201308c.html#s20130822_1)
の人と同じように IPv6 で逆引き必須にするのは否定的なのですが、
それはおいといて、現実問題として送信できないのは困るので、
`smtp_address_preference = ipv4` の設定を追加しました。

その設定をする前にテストメールを送ったら、
以下のようなエラーメールが返ってきて、
エラーメッセージの中の URL が
[以前ひっかかったとき](http://www.postfix-jp.info/ML/arc-2.5/msg00237.html)
とは変わっているということに気付きました。

```
<自分のアドレス@gmail.com> (expanded from <自分@あるサーバー>): host
    gmail-smtp-in.l.google.com[2607:f8b0:4002:c01::1a] said: 550-5.7.1
    [2401:xxxx:xxx:xxxx:xxx:xxx:xxx:xxx      16] Our system has detected
    550-5.7.1 that this message does not meet IPv6 sending guidelines regarding
    PTR 550-5.7.1 records and authentication. Please review 550-5.7.1
    https://support.google.com/mail/?p=ipv6_authentication_error for more 550
    5.7.1 information. e2si14078426yhm.125 - gsmtp (in reply to end of DATA
    command)
```

## 余談

以前はメールサーバーの spam 対策の設定は秘伝のたれ状態で
だんだんメンテナンスが難しくなってしまっていましたが、
spam 対策は milter-manager だったり、
firewall は [ufw](packages.debian.org/ufw) だったり、
いろいろと共通で使えるものが増えてきて楽になってきたように思います。
