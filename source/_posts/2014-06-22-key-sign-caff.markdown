---
layout: post
title: "caffでキーサインした"
date: 2014-06-22 19:07:19 +0900
comments: true
categories: event debian gpg
---
[第 85 回 関西 Debian 勉強会](https://wiki.debian.org/KansaiDebianMeeting/20140622 "第 85 回 関西 Debian 勉強会")
で
[坂本さん](https://launchpad.net/~mocchi)とキーサインをしたので、そのメモです。

<!--more-->

## 対象バージョン

- Ubuntu 12.04.4 LTS
- gnupg 1.4.11-3ubuntu2.5
- signing-party 1.1.4-1

## 事前準備

事前にキーサインをするとわかっていれば `gpg-key2ps` コマンドで fingerprint の紙を用意しておくと良いと思います。
今回は少人数だったので、
fingerprint は画面上で見せて確認してもらいました。

## 本人確認

対面で運転免許証などの写真付きの身分証明書で名前を確認して、
それと署名対象の鍵の uid に入っている名前が一致するのを確認しておきます。
また、後で署名するために fingerprint の情報も入手しておきます。

## caff の設定

基本的には
[Why GPG Key sign? 東京エリア Debian 勉強会 in OSC 2009 Tokyo/Fall](http://tokyodebian.alioth.debian.org/pdf/debianmeetingresume200910-presentation.pdf)
の PDF の内容のままです。

### .caffrc

自分の鍵 ID を `gpg --list-secret-keys` で確認すると、
`4096R/B4222F7A` とわかるので、
`gpg --fingerprint B4222F7A`
で fingerprint 全体を確認しておきます。
(fingerprint の末尾が鍵 ID です。)

spam よけのために email のところはちょっと改変していますが、
`~/.caffrc` は以下のように設定しています。
`keyid` は fingerprint の末尾のうち、
設定例と同じ長さだけ普通の鍵 ID よりちょっと長めに取り出して設定しています。
`owner` と `email` はメール送信の時に使われます。

```perl ~/.caffrc
    # .caffrc -- vim:ft=perl:
    # This file is in perl(1) format - see caff(1) for details.
    
    $CONFIG{'owner'} = 'Kazuhiro NISHIYAMA';
    $CONFIG{'email'} = 'zn mbf.nifty.com';
    #$CONFIG{'reply-to'} = 'foo@bla.org';
    
    # You can get your long keyid from
    #   gpg --with-colons --list-key <yourkeyid|name|emailaddress..>
    #
    # If you have a v4 key, it will simply be the last 16 digits of
    # your fingerprint.
    #
    # Example:
    #   $CONFIG{'keyid'} = [ qw{FEDCBA9876543210} ];
    #  or, if you have more than one key:
    #   $CONFIG{'keyid'} = [ qw{0123456789ABCDEF 89ABCDEF76543210} ];
    $CONFIG{'keyid'} = [ qw{262ED8DBB4222F7A} ];
    
    # Select this/these keys to sign with
    #$CONFIG{'local-user'} = [ qw{EE739B28C657086C 9B585538ED7E1B73 262ED8DBB4222F7A C9429DABCB28285B} ];
    
    # Additionally encrypt messages for these keyids
    #$CONFIG{'also-encrypt-to'} = [ qw{EE739B28C657086C 9B585538ED7E1B73 262ED8DBB4222F7A C9429DABCB28285B} ];
    
    # Mail template to use for the encrypted part
    #$CONFIG{'mail-template'} = << 'EOM';
    #Hi,
    #
    #please find attached the user id{(scalar @uids >= 2 ? 's' : '')}
    #{foreach $uid (@uids) {
    #    $OUT .= "\t".$uid."\n";
    #};}of your key {$key} signed by me.
    #
    #If you have multiple user ids, I sent the signature for each user id
    #separately to that user id's associated email address. You can import
    #the signatures by running each through `gpg --import`.
    #
    #Note that I did not upload your key to any keyservers. If you want this
    #new signature to be available to others, please upload it yourself.
    #With GnuPG this can be done using
    #       gpg --keyserver pool.sks-keyservers.net --send-key {$key}
    #
    #If you have any questions, don't hesitate to ask.
    #
    #Regards,
    #{$owner}
    #EOM
```

### ~/.caff/gnupghome/gpg.conf の設定

以前参考にした設定のまま

```text ~/.caff/gnupghome/gpg.conf
keyserver pgp.mit.edu
cert-digest-algo SHA256
personal-digest-preferences SHA256
```

となっていました。
PDF では SHA512 になっていたので、
SHA256 から SHA512 に変更しました。
今日の caff での署名した時点では SHA256 のままだったので、
次回から変わる予定です。

```text ~/.caff/gnupghome/gpg.conf
keyserver pgp.mit.edu
cert-digest-algo SHA512
personal-digest-preferences SHA512
```

## caff -u で署名

spam よけのためメールアドレスの所は改変した状態のログは以下の通りです。
「本当に署名しますか? (y/N)」のところで身分証明書と一緒に確認した fingerprint と合っているか確認します。

最後にメールを送信して終了です。
相手の鍵で暗号化されたメールが localhost の SMTP サーバー送信されます。

```console
    % caff -u B4222F7A D66FD341
    [INFO] Importing key 262ED8DBB4222F7A from your normal GnuPGHome.
    [INFO] fetching keys, this will take a while...
    [INFO] Sign the following keys according to your policy, then exit gpg with 'save' after signing each key
    gpg --local-user B4222F7A --homedir=/home/kazu/.caff/gnupghome --secret-keyring /home/kazu/.gnupg/secring.gpg --no-auto-check-trustdb --trust-model=always --edit 25DA5B9699F132DB74BD2270B5A586C7D66FD341 sign
    gpg (GnuPG) 1.4.11; Copyright (C) 2010 Free Software Foundation, Inc.
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.
    
    
    pub  4096R/D66FD341  作成: 2014-06-22  満了: 無期限       利用法: SC
    sub  4096R/5D3BA622  作成: 2014-06-22  満了: 無期限       利用法: E
    [ unknown] (1). Takashi Sakamoto <o-takashi sakamocchi.jp>
    
    
    pub  4096R/D66FD341  作成: 2014-06-22  満了: 無期限       利用法: SC
     主鍵の指紋: 25DA 5B96 99F1 32DB 74BD  2270 B5A5 86C7 D66F D341
    
         Takashi Sakamoto <o-takashi sakamocchi.jp>
    
    本当にこの鍵にあなたの鍵“Kazuhiro NISHIYAMA <zn mbf.nifty.com>”で署名してよいですか
    (B4222F7A)
    
    本当に署名しますか? (y/N) y
    
    次のユーザーの秘密鍵のロックを解除するには
    パスフレーズがいります:“Kazuhiro NISHIYAMA <zn mbf.nifty.com>”
    4096ビットRSA鍵, ID B4222F7A作成日付は2010-06-27
    
    
    gpg> save
    [INFO] B5A586C7D66FD341 1 Takashi Sakamoto <o-takashi sakamocchi.jp> done.
    [INFO] key 25DA5B9699F132DB74BD2270B5A586C7D66FD341 done.
    Mail signature for Takashi Sakamoto <o-takashi sakamocchi.jp> to 'o-takashi sakamocchi.jp'? [Y/n]
    %
```

## caff からのメールを受け取った相手のすべきこと

暗号化されたメールが届くので、
対応する秘密鍵を使って復号してメールを確認します。
さらにその中にある署名を自分の鍵束にインポートしてキーサーバーに送信します。

caff のやり方はここでメールアドレスの到達性もチェックしているようなので、
署名した側はキーサーバーに送信する必要はなさそうです。
むしろそういうことをしないようにするために
`~/.caff/gnupghome` に独自の鍵束を用意しているように思いました。
