---
layout: post
title: "letsencryptの証明書の更新に失敗していた(IPv6が原因だった)"
date: 2017-06-14 19:36:08 +0900
comments: true
categories: linux letsencrypt
---
Let's Encrypt の証明書の自動更新が失敗しているサーバーがあって、原因を調べたら AAAA レコードに設定している IPv6 アドレスが間違っていたのが原因でした。

<!--more-->

## 環境

- Debian GNU/Linux 8.8 (jessie)
- certbot 0.10.2-1~bpo8+1
- さくらインターネットの VPS で IPv6 を使用 (過去に tun6rd を使っていた)

## 現象

2016-03-29 に現在のサーバーに移動した時に A レコードを書き換えただけではなく、追加で tun6rd の頃の IPv6 アドレスを AAAA レコードに設定してしまいました。
別の IPv6 アドレスを設定しているサーバーからの接続に時間がかかるという現象が発生していたものの、原因がわからず、ずっとそのままの状態でした。

StartCom の証明書が事実上使えなくなってしまったので、 2016-12-04 に Let's Encrypt の証明書に変更しました。
初回の証明書の発行のときには問題なく発行できていました。
2017-02-03,2017-04-04 の自動更新も問題なく動いていました。

6月の自動更新で突然失敗するようになり、数日様子を見ていましたが、失敗し続けていたので、詳しく調査することにしました。

## 調査

色々悩んだ結果、 `/var/log/letsencrypt/letsencrypt.log` を眺めていたところ `addressUsed` に IPv6 のアドレスが出ているのに気づいて、もしかして、と思ってさらに調べることにしました。

    "validationRecord": [
      {
        "url": "http://XXX.example.org/.well-known/acme-challenge/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
        "hostname": "XXX.example.org",
        "port": "80",
        "addressesResolved": [
          "XX.XXX.XXX.XX",
          "2001:e41:XXXX:XXXX::1"
        ],
        "addressUsed": "2001:e41:XXXX:XXXX::1",
        "addressesTried": []
      }
    ]

該当のサーバーから `ping6 www.kame.net` などは問題なく通り、該当サーバーへの `ping6` も問題なく通ることなどを確認していたところ、IPv6 アドレスが違うことに気づきました。

## 修正

AAAA レコードを `2403:3a00:XXX:XXXX:XX:XXX:XXX:XX` に修正して、急いでいるわけでもないので certbot の自動実行を待ってみたところ、ちゃんと更新されました。

## 関連情報

[DVSNI challenge エラーの対処法](https://letsencrypt.jp/usage/dvsni-challenge-error.html)に `urn:acme:error:connection` の原因の例として A レコードのことは書いてあったのに AAAA レコードのことが書かれていなくて、可能性に気づくのが遅れたので、 AAAA レコードのことも書いてあると良いのではないかと思いました。

## まとめ

Let's Encrypt のサーバーの実装が変わったのか、IPv6 アドレスから IPv4 へのフォールバックをしなくなっていて、IPv6 アドレスの間違いに気づくことができ、接続が遅かった現象も解決しました。

主に IPv4 を使っているとなかなか気づかないので、 AAAA レコードを設定するときは、ちゃんと確認しておかないと、後でわかりにくいトラブルの原因になると実感しました。
