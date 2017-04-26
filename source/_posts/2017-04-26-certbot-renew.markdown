---
layout: post
title: "cronでcertbot renewの--force-renewalを使用してはいけない"
date: 2017-04-26 19:30:00 +0900
comments: true
categories: letsencrypt
---
タイトルで言いたいことはすべてですが、 `cron`で定期実行する `certbot renew` で `--force-renewal` オプションは使わない方が良いという話です。

<!--more-->

## `--force-renewal` とは?

`certbot` の `renew` サブコマンドは、標準で期限切れが近い (30 日未満) の証明書だけを自動で更新してくれる便利なサブコマンドです。
期限切れが近い証明書がなければ letsencrypt のサーバーへのアクセスがなく、余計な負荷をかけないので、1日2回実行が推奨されています。

`--force-renewal` をつけると有効期限に関係なく更新が実行されます。

## 悪い設定例

例えば [スタートアップスクリプト「Mastodon」の更新のお知らせ・旧スクリプトを使用して作成されたインスタンス向けの作業のお願い | さくらのクラウドニュース](http://cloud-news.sakura.ad.jp/2017/04/24/mastodon-startupscript-update/ "スタートアップスクリプト「Mastodon」の更新のお知らせ・旧スクリプトを使用して作成されたインスタンス向けの作業のお願い | さくらのクラウドニュース") に書いてある

    echo "0 5 1 * * root /usr/local/certbot/certbot-auto renew --webroot --webroot-path /home/mastodon/live/public --force-renew && /bin/systemctl reload nginx postfix" > /etc/cron.d/certbot-auto

という設定だと毎月 1 日の 5:00 に更新するという設定になっています。

## letsencrypt へのサーバーの負荷の問題

### 回数の問題

letsencrypt の証明書の有効期限は 90 日なので、 `--force-renewal` がなければ 2 ヶ月ごとに letsencrypt のサーバーへのアクセスが発生するのに対して、上記の例だと毎月アクセスが発生して、 letsencrypt のサーバーへの負荷は 2 倍になります。

### 固定時刻の問題

また 5:00 固定なので、同じ設定をしているサーバーが増えれば同時にアクセスが発生するのもよくない設定です。

## 更新失敗時の問題

マシンが起動していなくて `cron` が動かなかったとか、何らかの理由で証明書の更新に失敗した場合、毎月 1 回しか実行しない場合は 2,3 回しか更新のタイミングがないので、すべて失敗して証明書の期限切れになる可能性が高いです。

それに対して毎日 2 回実行している場合、失敗しても少なくとも 29 日の間再実行され続けるので、少なくとも 58 回はチャンスがあります。
(更新タイミング次第で 59 回になりそうです。計算が間違ってなければ。)

## Debian パッケージの certbot の例

Debian の certbot パッケージでは

    0 */12 * * * root test -x /usr/bin/certbot -a \! -d /run/systemd/system && perl -e 'sleep int(rand(3600))' && certbot -q renew

という設定になっていて、 (systemd 環境でなければ) perl によるランダムスリープを入れて時間がばらけるようになっています。

systemd 環境では以下のように `ExecStartPre` でランダムスリープするようになっています。

```
% cat /lib/systemd/system/certbot.timer
[Unit]
Description=Run certbot twice daily

[Timer]
OnCalendar=*-*-* 00,12:00:00
Persistent=true

[Install]
WantedBy=timers.target
%  cat /lib/systemd/system/certbot.service
[Unit]
Description=Certbot
Documentation=file:///usr/share/doc/python-certbot-doc/html/index.html
Documentation=https://letsencrypt.readthedocs.io/en/latest/
[Service]
Type=oneshot
ExecStartPre=/usr/bin/perl -e 'sleep int(rand(3600))'
ExecStart=/usr/bin/certbot -q renew
PrivateTmp=true
```

## まとめ

`--force-renewal` 付きで少ない回数の `certbot renew` を実行するのは欠点しかありません。

公式に推奨されているように `certbot renew` を `--force-renewal` なしで複数回実行する方が良いでしょう。
