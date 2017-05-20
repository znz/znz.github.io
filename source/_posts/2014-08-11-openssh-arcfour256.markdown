---
layout: post
title: "vagrantなどのローカルへのssh接続のみarcfour256で高速化する"
date: 2014-08-11 21:53:54 +0900
comments: true
categories: linux ssh vagrant
---
[大容量ファイルのSCP転送を高速にする方法 - 元RX-7乗りの適当な日々](http://d.hatena.ne.jp/rx7/20101025/p1 "大容量ファイルのSCP転送を高速にする方法 - 元RX-7乗りの適当な日々")
をみて、安全な可能性が高い経路だけ `arcfour256` のような高速な `Ciphers` を使いたいと思って、そうなるように設定しました。

[VirtualBoxでdokkuを試した]({{ root_url }}/blog/2013-11-15-dokku.html "VirtualBoxでdokkuを試した")
での設定は意図通りには動いていませんでした。

<!--more-->

## 2017-05-20 追記

[OpenSSH の ~/.ssh/config の見直しをした](/blog/2017-05-20-openssh-config.html)話で書いたように、今後は arcfour 系は使えなくなるようなので、デフォルトのままか `chacha20-poly1305@openssh.com` を外して `aes*-ctr` 系を使う方が良さそうです。

## 確認バージョン

- Mac OS X 10.9.4
- OpenSSH_6.2p2, OSSLShim 0.9.8r 8 Dec 2011

## OpenSSH の `~/.ssh/config` の設定例

最初に例を出しておきます。
詳しいことは説明は後でしますが、
`Host *` の設定は例として書いているだけで、
書かないことを推奨します。

```text ~/.ssh/config
HashKnownHosts no
Host 127.0.0.1
	Ciphers arcfour256,arcfour128
Host *
	Ciphers aes256-ctr,aes192-ctr,aes128-ctr
```

### 設定の固まり (セクション)

`~/.ssh/config` の設定は `Host ` で始まる行ごとの固まり (セクション) に分かれていて、
上の例の場合は `HashKnownHosts` が全体の設定、
次が `127.0.0.1` のみの設定、
最後が `*` つまりワイルドカードですべてのホストに対する設定になります。

### 設定の優先順位

最初に見つかった設定が使われます。
これが以前は勘違いしていた点で、
ホストごとの設定を優先したいのなら、
`Host` の上のファイルの冒頭には書かずに、
ファイルの最後に `Host *` で設定する必要があります。

さらに詳しい優先順位は `man ssh_config` で参照できますが、

1. コマンドラインオプション
2. ユーザー設定 (`~/.ssh/config`)
3. システムの設定 (`/etc/ssh/ssh_config`)

という順番で、その中で最初に見つかったものを使うようになっています。

つまり、ホストごとの設定はファイルの先頭に近い方に、
一般的な設定は最後に書く必要があります。

## Ciphers の一般設定

デフォルトは `man` で確認すると
`aes128-ctr,aes192-ctr,aes256-ctr,arcfour256,arcfour128,aes128-gcm@openssh.com,aes256-gcm@openssh.com,aes128-cbc,3des-cbc,blowfish-cbc,cast128-cbc,aes192-cbc,aes256-cbc,arcfour`
になっていて、
AES の CTR のビット数が多いものを優先するために上記の設定にしています。

設定するなら OS などの更新ごとに毎回ちゃんと `man` でデフォルトを確認すべきです。
よくわからないのなら、
`Host *` での設定はしない方が良いでしょう。

## 127.0.0.1 向けの Ciphers 設定

参考にしたサイトのコメントに「
[ただ、arcfourには別の問題が有るので、使わない方がいいです。 arcfour128/256はその問題の対処版なので、これらのみを使うようにした方がいいでしょう。](http://d.hatena.ne.jp/rx7/20101025/p1#c1291741909 "ただ、arcfourには別の問題が有るので、使わない方がいいです。 arcfour128/256はその問題の対処版なので、これらのみを使うようにした方がいいでしょう。")
」とあり、速度もほとんど変わらないので、
「arcfour128/256」だけ使う設定にしています。

## vagrant 用設定

`vagrant ssh --help` で追加のオプションが渡せるとわかったので、
`vagrant ssh -- -v` で接続時の状況を調べました。

コマンドラインオプションが優先されるということで、
`vagrant ssh -- -v -o Ciphers=arcfour256`
のように接続すると `arcfour256` になることが確認できました。

以前は `Host vagrant` で設定して `vagrant ssh` の代わりに `ssh vagrant` を使っていたのですが、
複数 `vagrant up` したときに最初に起動した VM のポート (2222) にしか接続できないという問題がありました。

そこで、他の VM のときにも使える設定を考えたところ、
`Host 127.0.0.1` でうまくいくことがわかりました。

## まとめ

元々のデフォルトは時代に合わせて最適なものに更新されているので、
不用意に固定してしまうと
[OpenSSLの脆弱性(CVE-2014-3511)でTLSプロトコルの基礎を学ぶ - ぼちぼち日記](http://d.hatena.ne.jp/jovi0608/20140808/1407483168 "OpenSSLの脆弱性(CVE-2014-3511)でTLSプロトコルの基礎を学ぶ - ぼちぼち日記")
のように強いものに固定していたつもりが弱いものに固定されてしまうことになる可能性があるので注意が必要です。

`127.0.0.1` 以外にも LAN 内のホストなど、経路の信頼性が比較的高くて高速に転送したい場合は `Host` 設定で `Ciphers arcfour256` を追加すると良いのではないでしょうか。

## 参考 URL

- [大容量ファイルのSCP転送を高速にする方法 - 元RX-7乗りの適当な日々](http://d.hatena.ne.jp/rx7/20101025/p1 "大容量ファイルのSCP転送を高速にする方法 - 元RX-7乗りの適当な日々")
- [GitHub で clone するときは SSH じゃなく HTTP を使ったほうが高速 - てっく煮ブログ](http://tech.nitoyon.com/ja/blog/2013/01/11/github-clone-http/ "GitHub で clone するときは SSH じゃなく HTTP を使ったほうが高速 - てっく煮ブログ")
- [githubのssh接続が速くなるらしい | Logicky Blog](http://endoyuta.com/2014/03/12/github%E3%81%AEssh%E6%8E%A5%E7%B6%9A%E3%81%8C%E9%80%9F%E3%81%8F%E3%81%AA%E3%82%8B%E3%82%89%E3%81%97%E3%81%84/ "githubのssh接続が速くなるらしい | Logicky Blog")
- [GitHub で SSH 接続できなくなった。SSH をつかった場合に高速化する設定が原因だった。 - そんなこと覚えてない](http://blog.eiel.info/blog/2013/11/09/no-mathcing-cipher-found-on-github/ "GitHub で SSH 接続できなくなった。SSH をつかった場合に高速化する設定が原因だった。 - そんなこと覚えてない")
- [OpenSSLの脆弱性(CVE-2014-3511)でTLSプロトコルの基礎を学ぶ - ぼちぼち日記](http://d.hatena.ne.jp/jovi0608/20140808/1407483168 "OpenSSLの脆弱性(CVE-2014-3511)でTLSプロトコルの基礎を学ぶ - ぼちぼち日記")
