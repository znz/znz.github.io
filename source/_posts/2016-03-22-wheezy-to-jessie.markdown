---
layout: post
title: "さくらの VPS の Debian wheezy を jessie にあげた"
date: 2016-03-22 12:00:00 +0900
comments: true
categories: debian linux ruby
---
Debian 6 "Squeeze" の [LTS](https://wiki.debian.org/LTS/Using) が終わって Debian 7 "Wheezy" もそろそろ Debian 8 "Jessie" にあげた方が良さそうな気がしてきたので、
[さくらの VPS](http://vps.sakura.ad.jp/) で使っている Debian 環境を Debian 7.9 から Debian 8.3 にあげてみました。

<!--more-->

## 事前準備

[第4章 Debian 7 (wheezy) からのアップグレード](https://www.debian.org/releases/jessie/amd64/release-notes/ch-upgrading.ja.html "第4章 Debian 7 (wheezy) からのアップグレード") などを読んで事前に注意点を確認しておきました。

### 古いパッケージの削除

Squeeze から残っているパッケージを確認するため

    aptitude search '~i!~Odebian'

で現在インストールできないパッケージを調べました。

`pg_upgradecluster` コマンドで移行したのに残したままだった postgresql-8.4 と postgresql-client-8.4 を purge しました。

### scponly

scponly を設定しているユーザーがいたのですが、 wheezy で既にパッケージがなくなっていたことに気づいたので、
「scponly wheezy」で検索して出てきた
http://chxor.chxo.com/post/74647790171/how-to-restrict-linux-users-to-only-sftp-without
を参考にして設定を移行しました。

まず `/home/hoge/` 以下の bin dev etc usr を削除しました。

次に `sftponly` グループを追加して、そのグループに該当ユーザーを追加しました。

    % sudo usermod -a -G sftponly hoge
    usermod: グループ 'sftponly' は存在しません
    zsh: exit 6     sudo usermod -a -G sftponly hoge
    % sudo addgroup sftponly
    グループ `sftponly' (グループ ID 1001) を追加しています...
    完了。
    % sudo usermod -a -G sftponly hoge

`/etc/ssh/sshd_config` の設定を変更しました。
関連するところだけ抜き出すと以下のような変更をしました。

    -Subsystem sftp /usr/lib/openssh/sftp-server
    +#Subsystem sftp /usr/lib/openssh/sftp-server
    +Subsystem sftp internal-sftp

    +# sftponly users, chrooted
    +Match group sftponly
    +ChrootDirectory /home/%u
    +AllowTcpForwarding no
    +X11Forwarding no
    +ForceCommand internal-sftp

`sudo service sshd restart` で設定を反映して、 sftp コマンドで接続して put や rm ができるのを確認しました。

### その他古いパッケージの削除

`sudo aptitude purge '~i!~Odebian'` で古いパッケージを削除しました。

    %  sudo aptitude purge '~i!~Odebian'
    以下のパッケージが削除されます:
      doc-linux-text{p} libbind9-60{p} libboost-iostreams1.42.0{p} libdb4.6{p} libdb4.7{p}
      libdb4.8{p} libdns69{p} libevent-1.4-2{p} libisc62{p} libisccc60{p} libisccfg62{p}
      libkadm5clnt-mit7{p} libkadm5srv-mit7{p} libkdb5-4{p} liblwres60{p} liblzma2{p}
      libssl0.9.8{p} libtokyocabinet8{p}
    更新: 0 個、新規インストール: 0 個、削除: 18 個、保留: 0 個。
    0  バイトのアーカイブを取得する必要があります。展開後に 20.4 M バイトのディスク領域が解放されます。
    先に進みますか? [Y/n/?]

`/etc/init.d` に余計なファイルが残っていると問題がおきるかもしれないので `sudo aptitude purge '~c'` で設定だけ残っているパッケージも purge しておきました。

    %  sudo aptitude purge '~c'
    [sudo] password for adminuser:
    以下のパッケージが削除されます:
      defoma{p} libept1{p} libexiv2-9{p} libgmp3c2{p} libgs8{p} libgsf-1-114{p} libgtk2.0-0{p}
      libgtk2.0-common{p} libmagickcore3{p} libmagickwand3{p} libmysqlclient16{p} libnl1{p}
      libopenipmi0{p} libpango1.0-common{p} libprotobuf6{p} libserf-0-0{p} libxcb-render-util0{p}
      libxcomposite1{p} libxcursor1{p} libxdamage1{p} libxfixes3{p} libxi6{p} libxinerama1{p}
      libxrandr2{p} mysql-server-5.1{p} odbcinst{p} odbcinst1debian2{p} php5-suhosin{p}
      update-inetd{p} x-ttcidfont-conf{p}
    更新: 0 個、新規インストール: 0 個、削除: 30 個、保留: 0 個。
    0  バイトのアーカイブを取得する必要があります。展開後に 0  バイトのディスク領域が新たに消費されます
    。
    先に進みますか? [Y/n/?]

### sudo find /etc -name '*.dpkg-*'

移行時の設定マージ作業が途中で残っていたファイルがあったので、 `sudo find /etc -name '*.dpkg-*'` で探して必要に応じて設定をマージして `*.dpkg-old` や `*.dpkg-dist` は削除しておきました。

### プロセス一覧の確認

pstree などでプロセス一覧をみて、アップグレード時に問題が起きそうなコマンドに目星をつけておきました。
一番の大物は apache 2.2 系から apache 2.4 系だと思いました。
slapd は 2.4.31-2+deb7u1 から 2.4.40+dfsg-1+deb8u2 でバージョン番号の変更も少なく互換性も高そうなので、問題はおきなさそうだと思いました。(実際大丈夫でした。)

## apt-line の変更

コメントアウトされていない部分の wheezy を jessie に置き換えました。

    --- a/apt/sources.list
    +++ b/apt/sources.list
    @@ -4,18 +4,18 @@

     #deb cdrom:[Debian GNU/Linux 6.0.1a _Squeeze_ - Official amd64 NETINST Binary-1 20110320-15:00]/ wheezy main

    -deb http://ftp.jp.debian.org/debian wheezy main non-free contrib
    -deb-src http://ftp.jp.debian.org/debian wheezy main non-free contrib
    +deb http://ftp.jp.debian.org/debian jessie main non-free contrib
    +deb-src http://ftp.jp.debian.org/debian jessie main non-free contrib

    -deb http://security.debian.org/ wheezy/updates main contrib non-free
    -deb-src http://security.debian.org/ wheezy/updates main contrib non-free
    +deb http://security.debian.org/ jessie/updates main contrib non-free
    +deb-src http://security.debian.org/ jessie/updates main contrib non-free

     # wheezy-updates, previously known as 'volatile'
    -deb http://ftp.jp.debian.org/debian wheezy-updates main contrib non-free
    -deb-src http://ftp.jp.debian.org/debian wheezy-updates main contrib non-free
    +deb http://ftp.jp.debian.org/debian jessie-updates main contrib non-free
    +deb-src http://ftp.jp.debian.org/debian jessie-updates main contrib non-free

    -deb http://ftp.jp.debian.org/debian wheezy-backports main contrib non-free
    -deb-src http://ftp.jp.debian.org/debian wheezy-backports main contrib non-free
    +deb http://ftp.jp.debian.org/debian jessie-backports main contrib non-free
    +deb-src http://ftp.jp.debian.org/debian jessie-backports main contrib non-free

     #deb http://ftp.jp.debian.org/debian wheezy-lts main non-free contrib
     #deb-src http://ftp.jp.debian.org/debian wheezy-lts main non-free contrib

## upgrade, dist-upgrade

[4.4. パッケージのアップグレード](https://www.debian.org/releases/jessie/amd64/release-notes/ch-upgrading.ja.html#upgradingpackages "4.4. パッケージのアップグレード") に以前は `aptitude` が推奨されていたこともあったが、今は `apt-get` がおすすめというようなことが書いてあったので、 `apt-get` で upgrade しました。

postgresql はまた /usr/share/doc/postgresql-common/README.Debian.gz をみて `pg_upgradecluster` が必要だと思いました。

`Configuring libc6:amd64` のときに apache2 の restart に失敗しましたが、認証周りなどの設定の書き方が変わった影響だろうと思って、後で直せばいいということで気にせず進みました。

その他は特に問題なく dist-upgrade 自体は終わりました。

## apache2 の設定調整

`/etc/apache2/sites-available/*`, `/etc/apache2/sites-enabled/*` の `.conf` 化などは upgrade 前に作業しておけば停止時間が短くて済ませられたのに、と後から気付きました。

### 認証・認可設定の変更

`/` はアクセス不可で `DocumentRoot` を個別にアクセス許可していたので、

    Order allow,deny
    Allow from all

を

    Require all granted

に変更して回りました。

### mod_python

apache の error.log を見ると

    The mod_python module can not be used on conjunction with mod_wsgi 4.0+. Remove the mod_python module from the Apache configuration.

というエラーが出ていたので、 `libapache2-mod-python` を purge したら apache2 が起動しました。
しかし後で `trac` で使っていたのに気付いたので、入れ直して代わりに `libapache2-mod-wsgi` の方を purge しました。

### .conf 化

設定ファイルに `.conf` が必須化されていたので、 `site-available` のファイルを `hoge.example.com` から `hoge.example.com.conf` のようにに改名して `a2ensite` しなおしました。

`conf.d` も available と enabled に仕組みが変わっていたので、
`/etc/apache2/conf.d/passenger.conf` も `conf-available` に移動して `a2enconf` しました。

### passenger-install-apache2-module

`sudo apache2ctl configtest` で起動しない原因を調べてみると、エラーメッセージを記録し忘れたのですが、`passenger` がリンクエラーで起動しないということになっていたので、 `passenger-install-apache2-module` を実行し直しました。

初回は開発用パッケージが足りないということで出てきたメッセージに従い `sudo apt-get install apache2-threaded-dev` でインストールして再実行して解決しました。 `apache2-dev` パッケージが入りました。

wheezy のときは `apache2-prefork-dev` パッケージを入れていたのですが、自動移行はされなかったようです。

### SSL 関連

`SSLCertificateChainFile` を指定していたところがあったので、 `SSLCertificateFile` に結合した証明書を指定するように移行しました。

### NameVirtualHost

`NameVirtualHost` を指定していたところを削除しました。

## trac

trac のプロジェクトごとのページを開くと

    Error

    TracError: The Trac Environment needs to be upgraded.

    Run "trac-admin /srv/trac/fprog.org/testproject upgrade"

というメッセージが出ていたので、その通りに実行しました。

    % sudo trac-admin /srv/trac/fprog.org/testproject upgrade
    Warning: Detected setuptools version 5.5.1. The environment variable 'PKG_RESOURCES_CACHE_ZIP_MANIFE
    STS' must be set to avoid significant performance degradation.
    アップグレードが終了しました。

    次のコマンドを実行すると Trac のドキュメントをアップグレードできます:

      trac-admin /srv/trac/fprog.org/testproject wiki upgrade

wiki upgrade も促されたので、実行しました。

すると http://www.fprog.org/projects/testproject が

    設定エラー
    None という名前の IRequestFilter インターフェイスの実装を見つけられません。コンポーネントが有効になっているかチェックするか、trac.ini の [trac] request_filters オプションを更新してください。

に変わったので、 `/srv/trac/fprog.org/testproject/conf/trac.ini` の

    request_filters = None

を

    request_filters =

に変更しました。
キーワードが一般的すぎて検索しきれなかったので、バグ報告などはしていません。

http://www.fprog.org/projects の Available Projects を見て、他のプロジェクトも同様に upgrade と `request_filters` の修正をしました。

## nadoka さん

iconv を使っていたところを kconv を使うように[変更](https://github.com/nadoka/nadoka/commit/328e01a5a2ae731ddc09f435dc4089eead3ba4ed)しました。

## w3ml

apache のエラーログに

    AH01215: ./w3ml:8:in `load'
    AH01215: : /home/w3ml/etc/w3ml.conf:10: invalid multibyte char (UTF-8) (SyntaxError)
    AH01215: /home/w3ml/etc/w3ml.conf:10: invalid multibyte char (UTF-8)
    AH01215: \tfrom ./w3ml:8:in `<main>'
    End of script output before headers: w3ml.cgi

と出ていたので、 `/home/w3ml/etc/w3ml.conf` の先頭に

    # -*- coding: euc-jp -*-

を追加しました。
これで表示は問題なく見えるようになりましたが、メールの取り込みなどがちゃんと動くかどうかはまだ様子見です。

## tdiary

ホスティングしている tdiary を確認してみると

    no such file to load -- redcarpet.so (LoadError)
    /usr/lib/ruby/vendor_ruby/redcarpet.rb:1:in `require'

とうエラーが出ていて、 `dpkg -L ruby-redcarpet` を確認してみると `redcarpet.so` は 2.1 用しかなかったので、
wheezy にあげたときに `public_html/diary/index.rb` を `#!/usr/bin/ruby1.8` に変更していたのを
`#!/usr/bin/ruby` に変更しました。

すると次は `500 Internal Server Error` とだけ出るようになったので、 apache のエラーログを確認してみると

    AH01215: /usr/lib/ruby/2.1.0/cgi/util.rb:37:in `gsub': invalid byte sequence in UTF-8 (ArgumentError)
    AH01215: \tfrom /usr/lib/ruby/2.1.0/cgi/util.rb:37:in `escapeHTML'
    AH01215: \tfrom /usr/share/tdiary/index.rb:50:in `rescue in <top (required)>'
    AH01215: \tfrom /usr/share/tdiary/index.rb:16:in `<top (required)>'
    AH01215: \tfrom /usr/lib/ruby/2.1.0/rubygems/core_ext/kernel_require.rb:55:in `require'
    AH01215: \tfrom /usr/lib/ruby/2.1.0/rubygems/core_ext/kernel_require.rb:55:in `require'
    AH01215: \tfrom index.rb:2:in `<main>'

というエラーが出ていました。
そこで `/usr/share/tdiary/index.rb` を `coding: utf-8` から `coding: ascii-8bit` に変更したところ、
`public_html/diary/filter/antirefspam.rb:240: invalid multibyte char (UTF-8)` というエラーが確認できたので、
コメントが euc-jp で書かれていたのが確認できたので `antirefspam.rb` に `coding: euc-jp` を追加しました。
`public_html/diary/filter/default.rb` でも同様のエラーが出たので `coding: ascii-8bit` を追加したところ、
正常に表示できるようになりました。

最初の `500 Internal Server Error` については [tdiary-core に報告](https://github.com/tdiary/tdiary-core/issues/555)したところ、[直った](https://github.com/tdiary/tdiary-core/commit/59557302e2dfd0cfa86b04b5d05e74dfe917900e) ようです。

修正コミットでは update.rb も同様の修正がされていたので、 `/usr/share/tdiary/update.rb` も同様に `coding: ascii-8bit` にしておきました。
