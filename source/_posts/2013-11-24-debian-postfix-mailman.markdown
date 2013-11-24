---
layout: post
title: "debianでpostfixと連携するvirtual hostではないmailmanの設定をした"
date: 2013-11-24 19:50
comments: true
categories: debian postfix mailman
---
Debian で postfix と mailman を組み合わせて使う時に
`lists.example.net` のように ML 専用のサブドメインを使う場合は
`/etc/mailman/postfix-to-mailman.py` のコメントなどを参考にして
設定すれば良いのですが、
ドメインを分けずに他のローカル配送などと同じドメインで
ML を運用する設定の情報があまりなかったので、
どう設定したのかをまとめてみました。

要点としては `alias_maps` に追加するだけでした。

<!--more-->

## 対象バージョン

- Debian GNU/Linux 7.2 (wheezy)
- postfix 2.9.6-2
- mailman 1:2.1.15-1

## mailman のインストール

[nginx + mailman でメーリングリスト: ぽそこし的日乗](http://posokosi.seesaa.net/article/378457038.html)
に書いてあるように、
なぜかインストール時の debconf の質問で設定した内容が反映されないので、
`dpkg-reconfigure mailman` で再設定する必要がありました。

`Default language for Mailman` の設定だけなら
`/etc/mailman/mm_cfg.py` を直接変更でも良いのですが、
`Languages to support` で `ja (Japanese)` にチェックしたときに
生成されるファイルもあるので、
`dpkg-reconfigure mailman` で再設定するのが無難です。

## 文字コードについて

mailman の日本語の文字コードは未だに euc-jp なので、
文字コードを設定できる端末を使うか、
[cocot](https://github.com/vmi/cocot)
で
`cocot -t utf-8 -p euc-jp ssh mlserver`
のように変換をするようにしておかないと文字化けして非常につらいです。

`LANG=C` で mailman 関係のコマンドを実行しても
英語メッセージにはならないので、
注意が必要です。

## postfix の設定

すでに `myhostname` とか `mydestination` に設定しているドメインに同居させるには
`alias_maps` に `hash:/var/lib/mailman/data/aliases` を追加するだけです。

具体的には
`alias_maps = hash:/etc/aliases`
という設定だったのなら、

```
alias_maps = hash:/etc/aliases
 hash:/var/lib/mailman/data/aliases
```

のように追加するだけです。
差分をとりやすいように別の行にするのが好みですが、
同じ行の末尾に追加でもかまいません。

## mm_cfg.py の設定

### MTA

`newlist` で ML を作成する前に
`sudoedit /etc/mailman/mm_cfg.py` で
`MTA='Postfix'` を有効にしておきます。

`newlist` の最後に
`/var/lib/mailman/bin/genaliases`
相当の処理が実行されるようなのですが、
`MTA='Postfix'` の設定をしておくと
`/var/lib/mailman/data/aliases*`
が自動で作成されるようになります。

`MTA` の設定が無いと `aliases` に設定すべき内容を含むメッセージが表示されるだけでした。

### ML のデフォルト値

以下のような感じで ML の設定のデフォルト値を設定しておくと
後から Web で設定変更する手間を省けます。

```
DEFAULT_SUBJECT_PREFIX = "[%(real_name)s:%%d] "
DEFAULT_MSG_FOOTER = ""
DEFAULT_REPLY_GOES_TO_LIST = 1
DEFAULT_MAX_MESSAGE_SIZE = 0
DEFAULT_LIST_ADVERTISED = No
DEFAULT_PRIVATE_ROSTER = 2
```

どんな設定項目があるのかは
`/usr/lib/mailman/Mailman/Defaults.py`
を参考にすれば良さそうです。

`sudo -u list /var/lib/mailman/bin/config_list -o - mailman | iconv -f euc-jp -t utf-8 | pager`
のように既存の ML の設定を (Web で変更しつつ) 参考にするのも良さそうです。

## パーミッション修正

`sudo /var/lib/mailman/bin/check_perms -f`
でパーミッションの修正をします。
`/var/lib/mailman/messages/` の中身のパーミッションの修正があるので、
ほぼ必須だと思います。

最終的には 10 個残りましたが、
シンボリックリンクでリンク先のグループは問題ないようなので、
無視することにしました。

```
$ sudo /var/lib/mailman/bin/check_perms -f
/var/lib/mailman/bin bad group (has: root, expected list) (fixing)
/var/lib/mailman/logs bad group (has: root, expected list) (fixing)
/var/lib/mailman/cron bad group (has: root, expected list) (fixing)
/var/lib/mailman/icons bad group (has: root, expected list) (fixing)
/var/lib/mailman/Mailman bad group (has: root, expected list) (fixing)
/var/lib/mailman/templates bad group (has: root, expected list) (fixing)
/var/lib/mailman/mail bad group (has: root, expected list) (fixing)
/var/lib/mailman/scripts bad group (has: root, expected list) (fixing)
/var/lib/mailman/cgi-bin bad group (has: root, expected list) (fixing)
/var/lib/mailman/locks bad group (has: root, expected list) (fixing)
Problems found: 10
Re-run as list (or root) with -f flag to fix
```

## ML 作成

mailman の依存で自動インストールされる `pwgen` を使って、
`pwgen -sB 20 1`
などでパスワードを生成して、
`sudo -u list $mlname mladmin@example.net $listpass`
のような感じで作成できます。

## その他の設定

後は fml からのメールの移行とか Web 経由での設定をしていきました。
