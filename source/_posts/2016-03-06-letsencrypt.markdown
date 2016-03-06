---
layout: post
title: "letsencrypt をメールサーバーにも導入して自動化するまで"
date: 2016-03-06 23:00:00 +0900
comments: true
categories: letsencrypt linux ubuntu
---
Ubuntu 12.04.5 LTS で letsencrypt 0.4.2 を使って apache2 と postfix と dovecot の証明書の自動更新を設定してみました。

<!--more-->

## 対象環境

- Ubuntu 12.04.5 LTS (amd64)
- letsencrypt 0.4.2
- apache2-mpm-prefork 2.2.22-1ubuntu1.10
- postfix 2.9.6-1~12.04.3
- dovecot-imapd 1:2.0.19-0ubuntu2.2
- python 2.7.3-0ubuntu2.2
- git 1:1.7.9.5-1ubuntu0.2

## 事前知識

https://letsencrypt.jp/ や [Qiita の記事](http://qiita.com/tags/letsencrypt) などを見て事前に予備知識を得た上で、
[Getting Started](https://letsencrypt.org/getting-started/ "Getting Started") などで最新情報を確認しました。

[Let's Encrypt 証明書の自動発行とELB自動登録を行ったログ](http://qiita.com/hidekuro/items/482520f220a305dc147b "Let's Encrypt 証明書の自動発行とELB自動登録を行ったログ")に書いてある

- letsencrypt-auto は virtualenv 上で動き、実行ユーザーの ~/.local に Python 環境を作ります。
- Let's Encrypt は、 5 renew / 7 days の制限があります。

の 2 点は事前に知っておくと良いと思います。

## 初期設定 (インストール)

まず
[Getting Started](https://letsencrypt.org/getting-started/ "Getting Started")
に従って初期設定をしました。

- 一般ユーザーのホームディレクトリで `git clone https://github.com/letsencrypt/letsencrypt` で最新版をとってきます。
- `cd letsencrypt` で clone したディレクトリに入ります。
- `letsencrypt-auto` スクリプトの中をある程度確認しておきます。
  - ここで precise (Ubuntu 12.04 LTS) や wheezy (Debian 7) だと backports の apt-line が設定されていない場合は自動で追加されるというのを知りました。 (apache プラグインが libaugeas0 の 1.0 以上に依存しているため)
- `./letsencrypt-auto --help` で依存関係の自動インストールが走ります。
  - sudo のパスワードがきかれるので入力します。
  - apt で libaugeas0 などが勝手に入ります。
  - その後、`Installing Python packages...` でしばらく時間がかかります。 (ここで `~/.local/share/letsencrypt` 以下にファイルがインストールされるので、何か不具合があった時には `~/.local` 以下を消して試すと良いらしいです。)

この時点では `/etc` 以下に変化はありませんでした。 (backports の apt-line は既に追加済みだったため)

## ヘルプ表示

`./letsencrypt-auto --help` や `./letsencrypt-auto --help all` でヘルプが確認できます。
ヘルプの表示だけでも `sudo` を経由するようです。

## テスト実行

1 週間に 5 回までの制限にひっかからないように、最初は staging サーバーで試そうと思ったため、 `--test-cert` を付けて
`./letsencrypt-auto certonly --test-cert --webroot -w /srv/www/hoge.n-z.jp/htdocs -d hoge.n-z.jp`
のように実行しました。

初回実行のため、メールアドレスと ToS の Agree が必要でした。

     ┌──────────────────────────────────────────────────────────────────────┐
     │ Enter email address (used for urgent notices and lost key recovery)  │
     │ ┌──────────────────────────────────────────────────────────────────┐ │
     │ │                                                                  │ │
     │ └──────────────────────────────────────────────────────────────────┘ │
     ├──────────────────────────────────────────────────────────────────────┤
     │                     < 了解 >           < 取消 >                      │
     └──────────────────────────────────────────────────────────────────────┘

でメールアドレスを入力しました。

     ┌──────────────────────────────────────────────────────────────────────┐
     │ Please read the Terms of Service at                                  │
     │ https://letsencrypt.org/documents/LE-SA-v1.0.1-July-27-2015.pdf. You │
     │ must agree in order to register with the ACME server at              │
     │ https://acme-v01.api.letsencrypt.org/directory                       │
     │                                                                      │
     │                                                                      │
     │                                                                      │
     │                                                                      │
     │                                                                      │
     │                                                                      │
     │                                                                      │
     │                                                                      │
     │                                                                      │
     │                                                                      │
     │                                                                      │
     │                                                                      │
     ├──────────────────────────────────────────────────────────────────────┤
     │                     <Agree >           <Cancel>                      │
     └──────────────────────────────────────────────────────────────────────┘

で表示された URL の PDF の内容を確認して Agree しました。

成功すると最後に以下のような注意事項が表示されました。
メールアドレスや ToS への同意は `/etc/letsencrypt/accounts` 以下にアカウントの秘密鍵などと一緒に保存されるようです。

    IMPORTANT NOTES:
     - If you lose your account credentials, you can recover through
       e-mails sent to mail@example.jp.
     - Congratulations! Your certificate and chain have been saved at
       /etc/letsencrypt/live/hoge.n-z.jp/fullchain.pem. Your cert will
       expire on 2016-06-04. To obtain a new version of the certificate in
       the future, simply run Let's Encrypt again.
     - Your account credentials have been saved in your Let's Encrypt
       configuration directory at /etc/letsencrypt. You should make a
       secure backup of this folder now. This configuration directory will
       also contain certificates and private keys obtained by Let's
       Encrypt so making regular backups of this folder is ideal.
     - If you like Let's Encrypt, please consider supporting our work by:
    
       Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
       Donating to EFF:                    https://eff.org/donate-le

## Apache 2.2 への設定反映

Apache 2.4.8 未満のため、 `fullchain.pem` ではなく `chain.pem` と `cert.pem` で設定します。

    SSLCertificateKeyFile /etc/letsencrypt/live/hoge.n-z.jp/privkey.pem
    SSLCertificateChainFile /etc/letsencrypt/live/hoge.n-z.jp/chain.pem
    SSLCertificateFile /etc/letsencrypt/live/hoge.n-z.jp/cert.pem

`sudo apache2ctl graceful` で反映し、 `https://hoge.n-z.jp` を開いて信頼していないルート証明書の問題による証明書エラーが出るのを確認しました。

## 本番実行

`--test-cert` を外して、
`./letsencrypt-auto certonly --webroot -w /srv/www/hoge.n-z.jp/htdocs -d hoge.n-z.jp`
のように実行しました。

staging サーバー (`acme-staging.api.letsencrypt.org`) ではなく、本番サーバー (`acme-v01.api.letsencrypt.org`) への接続になるため、メールアドレスの入力と ToS への同意が再度必要になりました。

再作成扱いのためか、こんな確認ダイアログが出たので、 2 を選択して OK を押しました。

    ┌──────────────────────────────────────────────────────────────────────┐
    │ You have an existing certificate that contains exactly the same      │
    │ domains you requested and isn't close to expiry.                     │
    │ (ref: /etc/letsencrypt/renewal/hoge.n-z.jp.conf)                     │
    │                                                                      │
    │ What would you like to do?                                           │
    │ ┌──────────────────────────────────────────────────────────────────┐ │
    │ │        1  Keep the existing certificate for now                  │ │
    │ │        2  Renew & replace the cert (limit ~5 per 7 days)         │ │
    │ │                                                                  │ │
    │ │                                                                  │ │
    │ │                                                                  │ │
    │ │                                                                  │ │
    │ │                                                                  │ │
    │ │                                                                  │ │
    │ │                                                                  │ │
    │ └──────────────────────────────────────────────────────────────────┘ │
    ├──────────────────────────────────────────────────────────────────────┤
    │                     <  OK  >           <Cancel>                      │
    └──────────────────────────────────────────────────────────────────────┘

`sudo apache2ctl graceful` で反映し、 `https://hoge.n-z.jp` を開いて証明書エラーが出ないのを確認しました。

## renew

letsencrypt 0.4 で `letsencrypt renew` というサブコマンドが増えていて、有効期限が 30 日未満の証明書の更新が簡単にできるようになっています。

これも最初は `--dry-run` を付けて staging サーバーで試してみました。
cron で実行する時のことを考えて、最初から root 権限付きになるように `sudo` 付きで実行してみました。
すると以下のようになって `/etc/letsencrypt/csr` と `/etc/letsencrypt/keys` 以下にはファイルが作成されてしまいました。
`/etc` を汚したくない場合は `--dry-run` はあまり実行しない方が良さそうだと思いました。

    % sudo /home/hoge/letsencrypt-auto renew --dry-run
    Checking for new version...
    Requesting root privileges to run letsencrypt...
       /home/hoge/.local/share/letsencrypt/bin/letsencrypt renew --dry-run
    Processing /etc/letsencrypt/renewal/hoge.n-z.jp.conf
    ** DRY RUN: simulating 'letsencrypt renew' close to cert expiry
    **          (The test certificates below have not been saved.)
    
    Congratulations, all renewals succeeded. The following certs have been renewed:
      /etc/letsencrypt/live/hoge.n-z.jp/fullchain.pem (success)
    ** DRY RUN: simulating 'letsencrypt renew' close to cert expiry
    **          (The test certificates above have not been saved.)

普通に実行したら skipped になって何も変化はありませんでした。

    % sudo /home/hoge/letsencrypt/letsencrypt-auto renew
    Checking for new version...
    Requesting root privileges to run letsencrypt...
       /home/hoge/.local/share/letsencrypt/bin/letsencrypt renew
    Processing /etc/letsencrypt/renewal/hoge.n-z.jp.conf
    
    The following certs are not due for renewal yet:
      /etc/letsencrypt/live/hoge.n-z.jp/fullchain.pem (skipped)
    No renewals were attempted.

## メールサーバー用証明書作成

まず、ドメイン認証で必要になるため、 `mx6.n-z.jp` にも apache2 の VirtualHost 設定を追加しました。
`https` は必要ないため、 `http` だけの設定をしました。

今回は 2 度目なので、いきなり `--test-cert` なしで実行しました。

    % /home/hoge/letsencrypt/letsencrypt-auto certonly --webroot -w /srv/www/mx6.n-z.jp/htdocs -d mx6.n-z.jp
    Checking for new version...
    Requesting root privileges to run letsencrypt...
       sudo /home/hoge/.local/share/letsencrypt/bin/letsencrypt certonly --webroot -w /srv/www/mx6.n-z.jp/htdocs -d mx6.n-z.jp
    
    IMPORTANT NOTES:
     - Congratulations! Your certificate and chain have been saved at
       /etc/letsencrypt/live/mx6.n-z.jp/fullchain.pem. Your cert will
       expire on 2016-06-04. To obtain a new version of the certificate in
       the future, simply run Let's Encrypt again.
     - If you like Let's Encrypt, please consider supporting our work by:
    
       Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
       Donating to EFF:                    https://eff.org/donate-le

## postfix の設定変更

`/etc/postfix/main.cf` で以下の設定をしました。

    smtpd_tls_cert_file = /etc/letsencrypt/live/mx6.n-z.jp/fullchain.pem
    smtpd_tls_key_file = /etc/letsencrypt/live/mx6.n-z.jp/privkey.pem
    smtpd_tls_mandatory_protocols = !SSLv2,!SSLv3

そして `sudo service postfix reload` で反映しました。

postfix の reload は `apache2ctl graceful` などと違ってメッセージが出てくるのが後の自動化の時に邪魔だったので標準出力に捨てるようにしました (後述)。

    % sudo service postfix reload
     * Reloading Postfix configuration...
       ...done.

## dovecot の設定変更

`/etc/dovecot/local.conf` で以下の設定をしました。

    ssl_cert = </etc/letsencrypt/live/mx6.n-z.jp/fullchain.pem
    ssl_key = </etc/letsencrypt/live/mx6.n-z.jp/privkey.pem

`sudo service dovecot reload` で反映しました。

## 自動更新

[Writing your own renewal script](https://letsencrypt.org/getting-started/#writing-your-own-renewal-script "Writing your own renewal script") を参考にして、以下のように `/etc/cron.daily/letsencrypt` を作成して毎日 `letsencrypt-auto renew` が実行されるようにしました。
postfix と dovecot の reload を追加しています。
前述したように、 postfix の reload は標準出力を捨てて、余計なメールが飛ばないようにしています。

    % sudoedit /etc/cron.daily/letsencrypt
    % cat /etc/cron.daily/letsencrypt
    #!/bin/sh
    if ! /home/hoge/letsencrypt/letsencrypt-auto renew > /var/log/letsencrypt/renew.log 2>&1 ; then
        echo Automated renewal failed:
        cat /var/log/letsencrypt/renew.log
        exit 1
    fi
    apachectl graceful
    service postfix reload >/dev/null
    service dovecot reload
    % sudo chmod +x /etc/cron.daily/letsencrypt

作成したファイルの動作確認をしておきます。

    % sudo /etc/cron.daily/letsencrypt
    % sudo cat /var/log/letsencrypt/renew.log
    Checking for new version...
    Requesting root privileges to run letsencrypt...
       /home/hoge/.local/share/letsencrypt/bin/letsencrypt renew
    Processing /etc/letsencrypt/renewal/mx6.n-z.jp.conf
    Processing /etc/letsencrypt/renewal/hoge.n-z.jp.conf
    
    The following certs are not due for renewal yet:
      /etc/letsencrypt/live/mx6.n-z.jp/fullchain.pem (skipped)
      /etc/letsencrypt/live/hoge.n-z.jp/fullchain.pem (skipped)
    No renewals were attempted.
    %

## まとめ

letsencrypt の導入から証明書の自動更新の設定までしました。

簡単に導入できて自動化もでき、しかも無料なので、まだベータ版であるという点が問題にならないところでは、積極的に使っていけば良さそうだと思いました。
