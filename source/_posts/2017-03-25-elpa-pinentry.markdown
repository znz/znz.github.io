---
layout: post
title: "elpaのpinentry.elを試してみた"
date: 2017-03-25 12:57:57 +0900
comments: true
categories: emacs gpg
---
Emacs の easypg と gnupg 2 で `Opening input file: Decryption failed,` になることの対処として、
[ELPA の pinentry.el](https://elpa.gnu.org/packages/pinentry.html) を試してみました。

<!--more-->

## 動作確認環境

デフォルトの gpg コマンドが gnupg 2 になっている Debian 系の環境で安定版がリリースされているものということで、
yakkety を使いました。

- Ubuntu 16.10 (yakkety)
- gnupg 2.1.15-1ubuntu6
- pinentry-curses 0.9.7-5 (をリビルドしたもの)
- emacs24 24.5+1-6ubuntu3
- pinentry-0.1.el, 2015-Jun-12, 15.8kB

## 結果

先に結果を書いておくと、
そのままだと動きませんでした。

`pinentry` を `--disable-pinentry-emacs --disable-inside-emacs` なしでリビルドして試すとうまくいきました。

## 設定反映

パッケージを入れ替えたり、
`~/.gnupg/gpg-agent.conf` を書き換えた後は、
`gpg-connect-agent killagent /bye` で `gpg-agent` を終了させました。
`pkill gpg-agent` でも同じようですが、
`gpg-connect-agent` の方が正式な手順のようです。
`gpgconf --kill gpg-agent` でも終了できるようです。

## オリジナルパッケージでの挙動

`~/.gnupg/gpg-agent.conf` に `allow-emacs-pinentry` を追加すると Emacs の中どころか、
コマンドライン直接でも使えなくなりました。

```
% cat ~/.gnupg/gpg-agent.conf
allow-emacs-pinentry
log-file /tmp/gpg-agent.log
```

のように `log-file` も指定してみると `gpg -c hoge` で暗号化したファイルを復号しようとしたときは

```
% gpg hoge.gpg
gpg: AES暗号化済みデータ
gpg: エージェントに問題: サポートされていません
gpg: 1 個のパスフレーズで暗号化
gpg: 復号に失敗しました: 秘密鍵がありません
```

となって、ログには

```
2017-03-25 13:XX:XX gpg-agent[XXXXX] gpg-agent (GnuPG) 2.1.15 started
2017-03-25 13:XX:XX gpg-agent[XXXXX] command 'GET_PASSPHRASE' failed: サポートされていません <Pinentry>
```

と出ていました。

公開鍵で暗号化したファイルを復号しようとしたときには

```
% gpg fuga.gpg
gpg: 2048-ビットRSA鍵, ID XXXXXXXXXXXXXXXX, 日付2017-03-24に暗号化されました
      "test@example.com"
gpg: 公開鍵の復号に失敗しました: サポートされていません
gpg: 復号に失敗しました: 秘密鍵がありません
```

となって、ログには

```
2017-03-25 13:XX:XX gpg-agent[XXXXX] gpg-agent (GnuPG) 2.1.15 started
2017-03-25 13:XX:XX gpg-agent[XXXXX] failed to unprotect the secret key: サポートされていません
2017-03-25 13:XX:XX gpg-agent[XXXXX] failed to read the secret key
2017-03-25 13:XX:XX gpg-agent[XXXXX] command 'PKDECRYPT' failed: サポートされていません <Pinentry>
```

と出ていました。

## パッケージのリビルド

変更点としては `debian/rules` の `--disable-pinentry-emacs --disable-inside-emacs` を外して `pinentry-curses` を入れ替えただけですが、
全体の手順もメモしておきます。

- `apt-get source pinentry-curses`
- `sudo apt-get build-dep pinentry`
- `sudo apt-get install devscripts`
- `cd pinentry-0.9.7`
- `vi debian/rules` で `SHARED_CONFIGS = --disable-rpath --without-libcap --disable-pinentry-emacs --disable-inside-emacs` を `SHARED_CONFIGS = --disable-rpath --without-libcap` に変更
- `debuild -uc -us -rfakeroot`
- `cd ..`
- `sudo dpkg -i pinentry-curses_0.9.7-5_amd64.deb`

## Emacs の設定

`M-x package-install RET pinentry RET` などで `pinentry.el` をインストールしておきます。

[GNU ELPA - pinentry](https://elpa.gnu.org/packages/pinentry.html) の説明にはありませんが、
`INSIDE_EMACS` 環境変数も設定しないと `pinentry-curses` に Emacs を開いている端末を乗っ取られて操作できなくなってしまいました。
(`pkill pinentry` で復帰できました。 `gpg-agent` も終了させたり `C-l` で再描画も必要かもしれません。)

`INSIDE_EMACS` 環境変数は `M-x shell` では `Emacsのバージョン,comint` に自動で設定されるようなので、
`pinentry.el` はそういう環境での使用を前提として作られたのかもしれません。

`(package-initialize)` は `(pinentry-start)` を呼ぶのに必要だったので追加しています。
起動後に `M-x pinentry-start` する時には `M-x package-initialize` は必要なかったので、
初期化の順番の問題なのだと思います。

```
% cat ~/.emacs.d/init.el
(setenv "INSIDE_EMACS" "t")
(package-initialize)
(pinentry-start)
```

## pinentry.el の動作

共通鍵で暗号化した `hoge.gpg` を開こうとするとミニバッファで

```
パスフレーズを入力:
```

と聞いてきて、間違ったパスフレーズを入力すると `Opening input file: Decryption failed,` になりました。

正しいパスフレーズを入力すると開けました。
編集して保存は新しいパスフレーズをきいてきました。
保存した時のパスフレーズがキャッシュされているらしく、
`C-x C-v` (`find-alternate-file`) での開き直しはパスフレーズ入力なしでできました。

公開鍵で暗号化した `fuga.gpg` を開こうとするとミニバッファで

```
OpenPGPの秘密鍵のロックを解除するためにパスフレーズを入力してください:
"test@example.com"
2048ビットRSA鍵, ID XXXXXXXXXXXXXXXX,
作成日付 2017-03-24 (主鍵ID YYYYYYYYYYYYYYYY).:
```

のようにきいてきました。

何も入力せずに `RET` を押すと `Opening input file: Decryption failed,` になりました。
パスフレーズを 3 回間違えても `Opening input file: Decryption failed,` になりました。

正しいパスフレーズを入力すると開けました。
編集して保存や `C-x C-v` (`find-alternate-file`) での開き直しもパスフレーズ入力なしでできました。

## パスフレーズの入力漏れに注意

オリジナルの `pinentry-curses` でも発生した問題です。

パスフレーズを入力せずに `RET` を押してしまうと `Opening input file: Decryption failed,` になった後、
再度開こうとしてもパスフレーズをきいてこなくて、
すぐに `Opening input file: Decryption failed,` になるようになってしまいました。
間違ったパスフレーズの場合は再度きいてくるので、
普通はそのまま `RET` は避けるようにして、
やってしまったら `gpg-connect-agent killagent /bye` などで `gpg-agent` を再起動するのが良さそうです。

`gpg hoge.gpg` で直接空欄で OK を押してしまったときも再度きいてこなくなってすぐに

```
% gpg hoge.gpg
gpg: AES暗号化済みデータ
gpg: gcry_kdf_derive failed: 無効なデータです
gpg: 1 個のパスフレーズで暗号化
gpg: 復号に失敗しました: 秘密鍵がありません
```

になるようになってしまったので、
`pinentry.el` に限らず注意した方が良さそうです。

## Debian パッケージで disable されている理由

debian/changelog には

```
pinentry (0.9.5-2) unstable; urgency=medium

  * disable emacs and emacs-fallback until we get a better description of
    them in the upstream documentation
```

と書いてあって、
意図的に無効にされているようです。

経緯としては
https://bugs.gnupg.org/gnupg/issue2034
や
https://bugs.debian.org/854797
をみるのが良さそうです。

upstream のドキュメント不足ということで、
ちゃんとドキュメントがあれば再度有効にしてもらえそうですが、
英語ドキュメントが書ける人じゃないと根本的な解決は難しそうです。

## まとめ

現状の Debian や Ubuntu では `pinentry.el` をそのまま使うことは難しそうです。

この記事に書いたように自前でリビルドしてセキュリティアップデートなどがあれば頑張ってリビルドし直すようにするか、
[pinentry-emacsを使ってみた](/blog/2017-03-23-pinentry-emacs.html)記事に書いたように、
セキュリティや Emacs の外での使い勝手などを犠牲にして `pinentry-emacs` を使うか、
ということになりそうです。

現状で一番良い方法は Emacs のミニバッファでの入力は諦めて、
GUI のダイアログが出てくる `pinentry` を使うことのようなので、
`ssh` で入った先の `emacs` で完結しないといけないなどの制限がなければ、
手元の Emacs と GUI の `pinentry` を組み合わせて、
リモートのファイルは TRAMP 経由で開く、
というのが良いのかもしれません。
