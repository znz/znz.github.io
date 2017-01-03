---
layout: post
title: "certbot の renew hook について"
date: 2016-11-20 14:52:31 +0900
comments: true
categories: letsencrypt
---
certbot で設定の再読み込みには post-hook よりも renew-hook を使った方が良さそうでした。

<!--more-->

## 対象バージョン

- Debian GNU/Linux 8.6 (jessie)
- certbot 0.9.3-1~bpo8+1

## hook の指定方法について

`certbot renew --pre-hook "service nginx stop" --post-hook "service nginx start"` のように `certbot` コマンドの引数で直接指定するか、 `/etc/letsencrypt/cli.ini` または `$XDG_CONFIG_HOME/letsencrypt/cli.ini` (`$XDG_CONFIG_HOME` が設定されていなければ `~/.config/letsencrypt/cli.ini`) に `renew-hook = service nginx reload` のように `--` を省いたオプション名で ini ファイルに指定する方法があるようです。(`ini` ファイルは `--config cli.ini` または `-c cli.ini` のようにコマンドラインで指定も可能)

## pre-hook, post-hook について

`pre-hook` と `post-hook` は standalone プラグインを使っている時に、 `certbot` の Web サーバーが 80 番ポートを使えるようにするために、通常の Web サーバーを止める用途に適しているようです。

そのため、`--dry-run` の時でも呼ばれるようです。

使用例:
`certbot renew --pre-hook "service nginx stop" --post-hook "service nginx start"`

## renew-hook について

`renew-hook` は証明書の更新が成功するごとに呼ばれるので、証明書の再読み込みに適しているようです。

環境変数 `RENEWED_LINEAGE` に `/etc/letsencrypt/live/www.example.com` のような証明書の場所へのパスが、
環境変数 `RENEWED_DOMAINS` に `www.example.com example.com` のようにスペース区切りの更新されたドメインのリストが渡ってくるようです。

証明書の更新ごとに呼ばれるようなので、たまたま同じタイミングで `www.example.com` と `other.example.net` の更新が起こったとして、
証明書の作成の時に `-d` を同時に指定して同じ証明書の SAN (Subject Alternative Name) に入っているなら、
`RENEWED_DOMAINS` に並んでいて、
別々に証明書を作成していれば `renew-hook` が別々に呼ばれました。

(2017-01-03 追記: `post-hook` は 1 回しか呼ばれませんでした。詳細は[certbot の renew hook について (その2)](/blog/2017-01-03-certbot-renew-hook.html)を参照)

## renew-hook のすすめ

以上の違いから、
`webroot` プラグインを使っている時の証明書の自動再読み込みには
`post-hook` ではなく `renew-hook` を使うのがおすすめです。

Web 上で `post-hook` を使っている例の方が多いのは、
`renew-hook` の方が後から実装されたからではないかと思います。
