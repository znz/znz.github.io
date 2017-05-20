---
layout: post
title: "OpenSSH の ~/.ssh/config の見直しをした"
date: 2017-05-20 14:30:00 +0900
comments: true
categories: linux ssh
---
最新の man を参考にしながら OpenSSH の `~/.ssh/config` の設定を見直してみたので、使っている設定項目についてまとめてみました。

<!--more-->

## 確認バージョン

全て確認したわけではないですが、古いのや新しいのを確認したいときはこの辺りを使いました。

- Debian GNU/Linux 8.7 (jessie) の OpenSSH_6.7p1 Debian-5+deb8u3, OpenSSL 1.0.1t  3 May 2016
- archlinux の OpenSSH_7.5p1, OpenSSL 1.1.0e  16 Feb 2017

## 参考

https://euske.github.io/openssh-jman/ssh_config.html の日本語訳や `man ssh_config` で英語のマニュアルを確認したりしました。

## 優先順位

1. コマンドラインオプション
2. ユーザごとの設定ファイル `~/.ssh/config`
3. システム全体にわたる (system-wide) 設定ファイル `/etc/ssh/ssh_config`

の順番で最初に見つかった設定が使われます。

設定ファイルは `Host` の行で区切られていて、設定ファイルの中でも前に見つかったものが優先されるので、ホストごとの設定をファイルの先頭の方に、全体的な設定を末尾の方に書くことを想定しているようです。

## Host

`Host` または `Match` は設定ファイルの区切りです。

コマンドラインで指定されたホスト名にマッチするので、`ssh localhost` の時には `Host localhost` の設定が使われて `Host 127.0.0.1` や `Host ::1` の設定は使われません。

## CheckHostIP

同じホスト名なのに IP アドレスが変わる可能性がある場合に `CheckHostIP no` にしておくと `known_hosts` に IP アドレスが記録されないようになります。

DNS を工夫して LAN 内では直接、外からはルーターのポートフォワーディング経由で接続できるようにしている場合など、接続の仕方によって変わる場合や、 GitHub などのようにホスト鍵はそのままで IP アドレスが変わることがあるサービスを使っている時に使うと良いと思います。

## Ciphers

[vagrantなどのローカルへのssh接続のみarcfour256で高速化する](/blog/2014-08-11-openssh-arcfour256.html)ということをしていたこともありましたが、https://srad.jp/comment/2875861 のリンク先の http://www.openssh.com/txt/release-7.1 に

```
Future deprecation notice
=========================

We plan on retiring more legacy cryptography in the next release
including:

 * Refusing all RSA keys smaller than 1024 bits (the current minimum
   is 768 bits)

 * Several ciphers will be disabled by default: blowfish-cbc,
   cast128-cbc, all arcfour variants and the rijndael-cbc aliases
   for AES.

 * MD5-based HMAC algorithms will be disabled by default.

This list reflects our current intentions, but please check the final
release notes for OpenSSH 7.2 when it is released.
```

と書いてあって、 http://www.openssh.com/txt/release-7.2 には

```
Potentially-incompatible changes
================================

This release disables a number of legacy cryptographic algorithms
by default in ssh:

 * Several ciphers blowfish-cbc, cast128-cbc, all arcfour variants
   and the rijndael-cbc aliases for AES.

 * MD5-based and truncated HMAC algorithms.

These algorithms are already disabled by default in sshd.
```


となっていて arcfour 系はデフォルトでは使われないようになったようなので、 arcfour 系はもうあまり使わない方が良さそうです。

OpenSSH_7.5p1 で `ssh -Q cipher` を確認しても残っていますが、サーバー側で無効になっていると結局使えないので、将来的には使えないものと考えて良さそうです。

速度を気にするなら [OpenSSH 6.6p1の各cipherのスループットを計測した](http://blog.uu59.org/2014-04-12-ssh-ciphers.html) を参考にして実際に計測するのが良さそうです。

試した感じだと `chacha20-poly1305@openssh.com` が遅くて `aes*-ctr` 系と `aes*-gcm@openssh.com` 系はそんなに違いがない感じだったので、速度を気にする場合は

    Ciphers aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com

ぐらいでいいのではないかと思いました。

## ControlMaster, ControlPath, ControlPersist

デフォルトの `ControlMaster no` のまま、使いたい Host だけ `ControlMaster auto` を設定する方法と、末尾の `Host *` でデフォルトを `ControlMaster auto` にしてしまって不要な Host や設定していると問題が起きる Host だけ `ControlMaster no` を設定する方法があると思います。

`ControlPath` は XDG Base Directory Specification の `XDG_CACHE_HOME` を参考にして `$HOME/.cache` を使って `ControlPath ~/.cache/ssh,%r,%h,%p,sock` にしています。(区切りはどの OS でも問題が起きにくそうなのとホスト名の中の `.` と区別できるように、ということで `,` にしています。)

`ControlPersist`は OpenSSH 5.6 で、存在を知った当時はまだ対応していない OS の方が多かったので、使っていませんでしたが、せっかくなので `ControlPersist 10` に設定してみました。

## DynamicForward

SOCKS proxy 機能です。

ブラウザーの接続元 IP アドレスを変えてテストしたい時に使うことがあったので、

    Host hoge hoge-proxy
    HostName FQDNかIPアドレス
    # ...その他の設定...

    Host hoge-proxy
    DynamicForward 1088

のように、`ssh hoge-proxy` で有効になるように設定していることもありましたが、滅多に使わないので、コマンドラインで `-D 1080` のように明示的に指定して使うことの方が多くなりました。

## ExitOnForwardFailure

ポートフォワーディングを設定しているのにポートフォワーディングされていないことの方が、ポートフォワーディングを設定している先に多重に接続できなくて困ることより多かったので、 `ExitOnForwardFailure yes` で有効にしています。

## FingerprintHash

サーバー側の OpenSSH が古い時に `ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub` などの結果が md5 固定なので、クライアントの OpenSSH が新しい時に `ssh -o FingerprintHash=md5` で md5 に変更して照合しています。

サーバー側が新しくて、クライアントが古い時に md5 の fingerprint を出すのは `ssh-keygen -l -E md5 -f /etc/ssh/ssh_host_ecdsa_key.pub` などのようです。

おまけの情報として、簡単に fingerprint の照合をするのに、適当なエディターにコピペして検索して全体がハイライトされるのを確認するという方法があります。
ダウンロードしたファイルのハッシュの確認にも同じ方法を使っています。

## ForwardAgent, ForwardX11

セキュリティリスクがあるので、必要な時はコマンドラインで明示的に `-A` や `-X` で使うようにしています。

## HashKnownHosts

不便さの方が大きい (IP アドレスも追加されているかどうか確認しにくいなど) と感じているので、 `HashKnownHosts no` で無効にしています。

## HostKeyAlias

多段 SSH の時などに使っています。

## HostName

    Host hoge hoge.example.com
    HostName hoge.example.com

のようにして、 FQDN の代わりに短いホスト名で接続できるようにしたり、 `HostName IPアドレス` で IP アドレスを直接指定したり、よく使っています。

## IdentitiesOnly, IdentityFile

`IdentitiesOnly yes` と `IdentityFile` をセットで使って特定の鍵を使うように指定できます。
`IdentityFile` のみだと `ssh-agent` に登録されている全ての鍵を試してしまうので、普通は `IdentitiesOnly yes` とセットにして使うと思います。

GitHub や Heroku などのように鍵でユーザーが区別されるサービスで複数ユーザーを使い分ける時には必須だと思います。

## LocalCommand, PermitLocalCommand

あまり使うことはなさそうですが、不用意に出力を伴うコマンドを設定していると、内部的に ssh を使うコマンドが謎の失敗をする原因になることがあります。

## LocalForward

IRC bouncer に接続するのに

    LocalForward 6665 127.0.0.1:6665
    LocalForward 6666 127.0.0.1:6666
    LocalForward 6667 127.0.0.1:6667

という感じで使っていましたが、 TCP over TCP はあまりよくないということで、今は OpenVPN を使うようになったので、使っていません。

## NoHostAuthenticationForLocalhost

仮想環境への接続で問題が起きたことがあるので `NoHostAuthenticationForLocalhost yes` にしています。

## Port

ポート番号を変更している Host の設定によく使います。
明示的に `Port 22` を書いていることもあります。

## ProxyCommand, ProxyJump

gateway を経由して target に接続するのに

    Host target
    HostKeyAlias target.example.com
	Hostname target.example.com
    ProxyCommand ssh gateway.example.com nc -w 330 target.example.com 22

のように使っていました。

このような場合に OpenSSH 7.3 以降だと `ProxyJump` という設定が使えそうです。

今はそういう接続が必要な Host がないので使っていません。

## RequestTTY

[Dokku](http://dokku.viewdocs.io/dokku/ "Dokku") のように常に有効にしていた方が便利な Host に `RequestTTY yes` を指定しています。

たとえば dokku の vagrant 環境用の設定全体は以下のようにしています。

     Host dokku dokku.me
     User dokku
     HostName 10.0.0.2
     Port 22
     UserKnownHostsFile /dev/null
     StrictHostKeyChecking no
     PasswordAuthentication no
     #IdentityFile ~/.vagrant.d/insecure_private_key
     IdentityFile ~/.ssh/id_rsa
     IdentitiesOnly yes
     LogLevel FATAL
     RequestTTY yes

## ServerAliveInterval

接続が切れた時にタイムアウトしてくれる設定です。
昔 Debian には `ProtocolKeepAlives` という設定がありましたが、今は `ServerAliveInterval` がどの OS でも使えます。
`ServerAliveInterval 300` や `ServerAliveInterval 30` を設定しています。

## StrictHostKeyChecking

vagrant 環境など、ローカルの接続でホスト鍵も変わる可能性のある Host で `StrictHostKeyChecking no` にしています。

## Tunnel, TunnelDevice

ssh のプロセスに与える権限が大きくなりすぎてしまうので使いにくいと思って使っていません。
代わりにトンネルには OpenVPN を使っています。

## UpdateHostKeys

OpenSSH 6.8 以降のサーバーとクライアントだと複数のホスト鍵を受け取れるようになるようなので `UpdateHostKeys ask` にしておくと良さそうです。

試してみると、たとえば `ecdsa-sha2-nistp256` のホスト鍵だけ `known_hosts` に登録されているホストに接続する時に

    % ssh localhost
    vagrant@localhost's password:
    Learned new hostkey: RSA SHA256:+sjb9SQChXBlm/pZDyl8ORJQb4wP16eeKDqUvDli5wU
    Learned new hostkey: ED25519 SHA256:rvlZW3lRcN86oPu19ym032zhuzVLz37E88A3VX2fVHE
    Accept updated hostkeys? (yes/no): yes

のように RSA のホスト鍵と ED25519 のホスト鍵も `known_hosts` に登録するかどうかきいてきました。
(試した範囲では DSA の鍵は追加されませんでした。)

サーバー側が古いバージョンでも特に何も悪影響はなさそうなので、クライアントが OpenSSH 6.8 以降ならデフォルトで有効にしても良さそうです。
(クライアントが古いとエラーになるので、バージョンが古いうちから書いておくことはできない。)

`ssh_config` の説明に書いてあるように `ControlPersist` を有効にしていると `UpdateHostKeys` は無効になるようです。

## User

`ssh user@host` で指定する代わりに `~/.ssh/config` で `User` を設定することが多いです。

## UserKnownHostsFile

ローカルの仮想環境は `UserKnownHostsFile /dev/null` で事実上無効にしたり、`UserKnownHostsFile ~/.ssh/known_hosts.d/hoge.known_hosts` のように個別ファイルにして調べる行数を減らして速くしたりしています。

## VerifyHostKeyDNS

[Verifying with DNS](https://devcenter.heroku.com/articles/git-repository-ssh-fingerprints#verifying-with-dns "Verifying with DNS") に書いてあるように `heroku.com` は SSHFP リソースレコードが設定されているようなので、 `VerifyHostKeyDNS yes` にしています。

## まとめ

`ServerAliveInterval` のように以前設定を見直したときにどの環境でも使えるようになっていたものもありましたが、 `ControlPersist` のように以前は使えない環境もあったけど、今はどこでも使える設定があったり、 `UpdateHostKeys` のようにまだ使えない環境もある設定もあったりするので、たまに気が向いたときに設定の見直しをするのはお勧めだと思いました。
