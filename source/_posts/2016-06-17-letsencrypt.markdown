---
layout: post
title: "letsencrypt の Rate Limit について"
date: 2016-06-17 15:44:18 +0900
comments: true
categories: letsencrypt linux
---
[Let’s Encryptをcloudapp.azure.comで使おうとするときのハマりどころ](http://www.clear-code.com/blog/2016/6/16.html "Let’s Encryptをcloudapp.azure.comで使おうとするときのハマりどころ")
という記事で Rate Limit の制限解除方法について誤解があるようなので、解説してみたいと思います。

<!--more-->

## Rate Limit について

`細かな制限はいろいろありますが、どうやら特定ドメインに対しては、週に20個のSSL/TLSサーバー証明書しか発行しないようです。そのため*.cloudapp.azure.comなどのように、皆がこぞってSSL/TLSサーバー証明書を取得しに行くような場合、見事に制限にひっかかってしまいます。`
と書いてありますが、これはその通りだと思います。

`これを回避するには別途独自にドメインをとるなどしなければなりません。`
とありますが、 Dynamic DNS や IaaS などを利用している場合はプロバイダーに対応してもらうという方法があります。
また、その方が Cookie Monster と呼ばれる問題にも対処出来て望ましいと思います。

## 適用例外について

[DDNS Rate Limited](https://github.com/certbot/certbot/issues/1607 "DDNS Rate Limited")
や
[Too many certificates already issued for dynamic dns provider sub domain](https://github.com/certbot/certbot/issues/2186 "Too many certificates already issued for dynamic dns provider sub domain")
のやりとりをみてもらえば詳細はわかるのですが、簡単に言うと

- プロバイダーに [Public Suffix List](https://publicsuffix.org/) に登録してもらう
- [Let's Encrypt のサーバー](https://github.com/letsencrypt/boulder) に反映してもらう

ということになります。

## 「正式版じゃなくてもいいから試してみたい」について

`certbot --help all` で出てくるヘルプにあるように `--test-cert` オプションか `--staging` オプションを使えば `/usr/lib/python2.7/dist-packages/certbot/constants.py` を書き換えるなどという方法をとらなくてもステージング環境の ACME サーバーを使えるようになります。

```
% certbot --help all
(中略)
testing:
  The following flags are meant for testing purposes only! Do NOT change
  them, unless you really know what you're doing!

  --debug               Show tracebacks in case of errors, and allow
                        letsencrypt-auto execution on experimental platforms
                        (default: False)
  --no-verify-ssl       Disable SSL certificate verification. (default: False)
  --tls-sni-01-port TLS_SNI_01_PORT
                        Port number to perform tls-sni-01 challenge. Boulder
                        in testing mode defaults to 5001. (default: 443)
  --http-01-port HTTP01_PORT
                        Port used in the SimpleHttp challenge. (default: 80)
  --break-my-certs      Be willing to replace or renew valid certs with
                        invalid (testing/staging) certs (default: False)
  --test-cert, --staging
                        Use the staging server to obtain test (invalid) certs;
                        equivalent to --server https://acme-
                        staging.api.letsencrypt.org/directory (default: False)
(後略)
```

## まとめ

Dynamic DNS や IaaS などを使っていて Rate Limit に引っかかった時の対処方法について説明しました。
また Cookie Monster と呼ばれる問題の影響についても少し触れました。

また、ステージング環境の ACME サーバーを使うのにソースを書き換えなくてもオプション指定で済むということを説明しました。
