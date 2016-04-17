---
layout: post
title: "jessie で dovecot-imapd と postfix の設定をして Thunderbird 用の autoconfig ファイルを用意した"
date: 2016-04-17 21:14:05 +0900
comments: true
categories: debian linux
---
[Ubuntu Weekly Recipe 第387回　UbuntuでSSLを利用したサービスを構築する](http://gihyo.jp/admin/serial/01/ubuntu-recipe/0387?page=4) を参考にして、 Debian jessie で dovecot-imapd で IMAP サーバーの設定をしました。

<!--more-->

## 対象バージョン

- Debian GNU/Linux 8.4 (jessie)
- postfix 2.11.3-1
- dovecot-imapd 1:2.2.13-12~deb8u1

## インストール

`sudo aptitude install dovecot-imapd` でインストールしました。

## 認証設定

クライアントによっては plain 認証は使えず login 認証が必要なので `sudoedit /etc/dovecot/conf.d/10-auth.conf` で `auth_mechanisms = plain login` に設定を変更しました。

## ssl 設定

`sudoedit /etc/dovecot/conf.d/10-ssl.conf` で以下の設定を変更しました。

- `ssl = no` を `ssl = required` に
- `ssl_cert`, `ssl_key` を設定
- `#ssl_protocols = !SSLv2` を `ssl_protocols = !SSLv2 !SSLv3` に

http://wiki.dovecot.org/SSL/DovecotConfiguration によると `ssl` の設定については `ssl=no`, `ssl=yes and disable_plaintext_auth=yes`, `ssl=yes and disable_plaintext_auth=yes`, `ssl=required` という設定の組み合わせがあるようです。

## Maildir 設定

postfix の設定で `Maildir` への配送を使っているので、 `sudoedit /etc/dovecot/conf.d/10-mail.conf` で `mail_location = mbox:~/mail:INBOX=/var/mail/%u` を `mail_location = maildir:~/Maildir` に設定を変更しました。

## postfix との認証連携

まず `sudo ls -al /var/spool/postfix/private` で `auth` がないのを確認しました。

`sudoedit /etc/dovecot/conf.d/10-master.conf` で

```plain /etc/dovecot/conf.d/10-master.conf
  #unix_listener /var/spool/postfix/private/auth {
  #  mode = 0666
  #}
```

を

```plain /etc/dovecot/conf.d/10-master.conf
  unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
  }
```

に変更しました。

設定反映後に `sudo ls -al /var/spool/postfix/private` で `auth` が owner も group も postfix でパーミッションが `srw-rw----` になっているのを確認します。

## 設定反映

`sudo service dovecot restart` で設定を反映しました。

`journalctl -u dovecot.service` や `sudo ss -lntp | grep dovecot` で問題なく起動していることを確認しました。

## ufw でポート開放

`ufw allow 993/tcp` と `ufw limit 993/tcp` でポートを開放して連続接続数制限をしました。
## postfix 設定

### SSL 設定

`smtpd_tls_cert_file` と `smtpd_tls_key_file` を設定します。
`smtpd_tls_cert_file` は中間証明書も結合したファイルを指定します。

その他の設定は以下のように設定しました。

```plain /etc/postfix/main.cf
smtpd_tls_mandatory_protocols = !SSLv2, !SSLv3
smtpd_tls_protocols = !SSLv2, !SSLv3
smtp_tls_mandatory_protocols = !SSLv2, !SSLv3
smtp_tls_protocols = !SSLv2, !SSLv3
```

`smtpd_tls_security_level = may` は設定すると外部からのメール送信で問題が起きたことがあったので、メールの送信テストをしつつ、様子を見ながら設定するかどうか決めます。

### SASL 設定

[Ubuntu Weekly Recipe](http://gihyo.jp/admin/serial/01/ubuntu-recipe/0387?page=4) では `smtpd_sasl_auth_enable` は `/etc/postfix/main.cf` でも `yes` にしていますが、 `/etc/postfix/master.cf` で `smtps` と `submission` で個別に `yes` に設定されているので、 `/etc/postfix/main.cf` では `no` のままにしておきます。

そうしておかないと 25 番ポートでも SMTP-Auth が有効になって、しかも `smtpd_tls_security_level = may` だとパスワードが平文で流れても良いということになるので、危険なことが起きる可能性があります。
最近は OBP25B という対策が普及しているので、一般のクライアントが間違って平文で送信してしまう可能性は低いと思いますが、余計な危険は避けておくのが良いと思います。

CRAM-MD5 が有効なら `smtpd_sasl_security_options = noanonymous,noplaintext` にして `master.cf` で `noanonymous` だけにしても良いかと思ったのですが、 plain 認証と login 認証しかない状態で `noplaintext` もつけてしまうと 25 番ポートで listen しているのに接続しても最初の `220 mail.example.org ESMTP Postfix (Debian/GNU)` が出てこなくてログに `fatal: no SASL authentication mechanisms` と記録されるという状態になってしまいました。
`/etc/passwd` (`/etc/shadow`) による認証だとパスワードをハッシュ化された状態でしか保存していないため、 CRAM-MD5 は使えないので、これ以上の検証はできませんでした。

以上を踏まえて以下のように設定しました。

```plain /etc/postfix/main.cf
smtpd_sasl_auth_enable = no
smtpd_sasl_local_domain = $myhostname
smtpd_sasl_security_options = noanonymous
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
broken_sasl_auth_clients = yes
```

`smtpd_sasl_local_domain` や `broken_sasl_auth_clients` は不要かもしれません。

## Thunderbird 用自動設定ファイル設置

[Thunderbird のアカウント情報自動設定機能](https://developer.mozilla.org/ja/docs/Mozilla/Thunderbird/Autoconfiguration)のために `mail/config-v1.1.xml` を設置します。

`@example.com` のメールアドレスに対して `http://autoconfig.example.com/mail/config-v1.1.xml` か `http://example.com/.well-known/autoconfig/mail/config-v1.1.xml` を用意します。

内容は以下のような感じになります。
`mozilla.org` の例との主な違いは `authentication` を `password-encrypted` ではなく `password-cleartext` にしているところと、 `username` を `%EMAILADDRESS%` ではなく `%EMAILLOCALPART%` にしているところです。

```xml mail/config-v1.1.xml
<?xml version="1.0" encoding="utf-8"?>
<clientConfig version="1.1">
  <emailProvider id="example.com">
    <domain>example.com</domain>
    <displayName>Example.com Mail</displayName>
    <displayShortName>example.com</displayShortName>
    <incomingServer type="imap">
      <hostname>mail.example.com</hostname>
      <port>993</port>
      <socketType>SSL</socketType>
      <authentication>password-cleartext</authentication>
      <username>%EMAILLOCALPART%</username>
    </incomingServer>
    <outgoingServer type="smtp">
      <hostname>mail.example.com</hostname>
      <port>587</port>
      <socketType>STARTTLS</socketType>
      <authentication>password-cleartext</authentication>
      <username>%EMAILLOCALPART%</username>
    </outgoingServer>
  </emailProvider>
</clientConfig>
```

## まとめ

[Ubuntu Weekly Recipe](http://gihyo.jp/admin/serial/01/ubuntu-recipe/0387?page=4)
