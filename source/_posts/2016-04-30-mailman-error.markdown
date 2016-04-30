---
layout: post
title: "wheezy から jessie にあげたら mailman でエラーが起きていた"
date: 2016-04-30 23:52:04 +0900
comments: true
categories: debian linux
---
wheezy から jessie にあげた VPS の環境のうちの 1 個で mailman を使っていたのですが、ちゃんと確認していなかったらエラーが起きてちゃんとメールが配送されていなかったので、修正しました。

<!--more-->

## 環境

- Debian GNU/Linux amd64 の wheezy から jessie にあげた環境
- [mailman](http://packages.debian.org/mailman) 1:2.1.15-1+deb7u1 から 1:2.1.18-2

## エラーの内容

`/var/log/mailman/error` を見ると以下のようなエラーが出てメールの配送がされていませんでした。

```
Apr 30 17:02:36 2016 (17947) Uncaught runner exception: 'utf8' codec can't decode byte 0xcb in position 5: invalid conti
nuation byte
Apr 30 17:02:36 2016 (17947) Traceback (most recent call last):
  File "/var/lib/mailman/Mailman/Queue/Runner.py", line 119, in _oneloop
    self._onefile(msg, msgdata)
  File "/var/lib/mailman/Mailman/Queue/Runner.py", line 190, in _onefile
    keepqueued = self._dispose(mlist, msg, msgdata)
  File "/var/lib/mailman/Mailman/Queue/IncomingRunner.py", line 130, in _dispose
    more = self._dopipeline(mlist, msg, msgdata, pipeline)
  File "/var/lib/mailman/Mailman/Queue/IncomingRunner.py", line 153, in _dopipeline
    sys.modules[modname].process(mlist, msg, msgdata)
  File "/var/lib/mailman/Mailman/Handlers/CookHeaders.py", line 179, in process
    i18ndesc = uheader(mlist, mlist.description, 'Reply-To')
  File "/var/lib/mailman/Mailman/Handlers/CookHeaders.py", line 65, in uheader
    return Header(s, charset, maxlinelen, header_name, continuation_ws)
  File "/usr/lib/python2.7/email/header.py", line 183, in __init__
    self.append(s, charset, errors)
  File "/usr/lib/python2.7/email/header.py", line 267, in append
    ustr = unicode(s, incodec, errors)
UnicodeDecodeError: 'utf8' codec can't decode byte 0xcb in position 5: invalid continuation byte

Apr 30 17:02:36 2016 (17947) SHUNTING: 1462003356.244866+ea73a2d0f0636f691f515c4b199b5e8c21436142
```

## 調査

エラーメッセージで適当に切り出していろいろ検索してみたところ、
[qrunner crashes on invalid unicode sequence](https://bugs.launchpad.net/mailman/+bug/1462755)
と同じ現象だとわかりました。

コメントにあるように調査してみたところ、確かに description に問題がありそうでした。
Web の設定画面で見てみると info も化けていたので、元の設定を調査するために `withlist` の環境で表示しておきました。

```
# withlist lilo
lilo のリストを読み込中 (ロック解除)
変数 `m' が lilo の MailList インスタンスです
>>> m.preferred_language
'ja'
>>> m.description
'LILO \xcb\xdc\xc2\xceML'
>>> m.info
'LILO ( \xa4\xea\xa4\xed : Linux Install Learning Osaka ) \xa4\xcf\xb4\xd8\xc0\xbe\xa4\xce Linux \xa5\xe6\xa1\xbc\xa5\xb6\xb2\xf1\xa4\xc7\xa4\xb9\xa1\xa3 \xbc\xe7\xa4\xcb\xb4\xd8\xc0\xbe\xa4\xce Linux \xa5\xe6\xa1\xbc\xa5\xb6\xa4\xce\xb8\xf2\xce\xae\xa1\xa2\xbe\xf0\xca\xf3\xb8\xf2\xb4\xb9\xa4\xce\xbe\xec\xa4\xf2\xc4\xf3\xb6\xa1\xa4\xb9\xa4\xeb\xa4\xbf\xa4\xe1\xa4\xcb\xb3\xe8\xc6\xb0\xa4\xb7\xa4\xc6\xa4\xa4\xa4\xde\xa4\xb9\xa1\xa3'
>>>
最終処理中
#
```

## 変換

何でも良かったのですが、使い慣れているという理由で ruby の nkf で変換しました。
変換した結果を Web の設定画面から設定し直しました。

```
% rbenv exec irb -r irb/completion --simple-prompt
>> require 'nkf'
=> true
>> NKF.nkf('-w',"LILO \xcb\xdc\xc2\xceML")
=> "LILO 本体ML"
>> NKF.nkf('-w',"'LILO ( \xa4\xea\xa4\xed : Linux Install Learning Osaka ) \xa4\xcf\xb4\xd8\xc0\xbe\xa4\xce Linux \xa5\xe6\xa1\xbc\xa5\xb6\xb2\xf1\xa4\xc7\xa4\xb9\xa1\xa3 \xbc\xe7\xa4\xcb\xb4\xd8\xc0\xbe\xa4\xce Linux \xa5\xe6\xa1\xbc\xa5\xb6\xa4\xce\xb8\xf2\xce\xae\xa1\xa2\xbe\xf0\xca\xf3\xb8\xf2\xb4\xb9\xa4\xce\xbe\xec\xa4\xf2\xc4\xf3\xb6\xa1\xa4\xb9\xa4\xeb\xa4\xbf\xa4\xe1\xa4\xcb\xb3\xe8\xc6\xb0\xa4\xb7\xa4\xc6\xa4\xa4\xa4\xde\xa4\xb9\xa1\xa3'")
=> "'LILO ( りろ : Linux Install Learning Osaka ) は関西の Linux ユーザ会です。 主に関西の Linux ユーザの交流、情報交換 の場を提供するために活動しています。'"
>>
```

## 失敗したメールの再配送

設定し直した後、しばらく待ってみても再配送はされなかったので、メールキューの強制再実行が必要かと思って、 `/var/lib/mailman` 以下を調べてみたところ、 `/var/lib/mailman/qfiles/shunt/` の中にファイルが溜まっていることがわかりました。

さらに調べていて見つけた
http://docs.python.jp/contrib/mailman/siteadmin.html#unshunt
によると qrunner が処理中にエラーを発生させて処理できなかったメールは `<prefix>/qfiles/shunt` に保存されていて、`unshunt` コマンドで処理できるらしいということなので、試してみたところ、ちゃんと溜まっていたメールが配送されました。

## NEWS.Debian 確認

復旧を優先して、ちゃんと見るのを忘れていたのですが、 `/usr/share/doc/mailman/NEWS.Debian.gz` によると `mailman (1:2.1.16-1exp1)` で UTF-8 化したから description を webinterface などから設定しなおせと書いてありました。

```
mailman (1:2.1.16-1exp1) experimental; urgency=low

  This version has changed the encoding of most strings, templates
  and pages to UTF-8 to meet the Debian release goal of full UTF-8
  support in all packages. It also no longer automatically converts
  mails to ISO-8859-1.

  If you have been using any nōn-ASCII strings in places such as
  the mailing list description, these were be stored wrongly in the
  list configuration file (config.pck), so you will need to change
  those (e.g. via the webinterface) again in order to have them be
  displayed correctly.

 -- Thorsten Glaser <tg@mirbsd.de>  Sun, 29 Dec 2013 14:35:50 +0000
```

## まとめ

mailman が UTF-8 対応で description や info などに日本語などの ASCII 以外の文字を使っているとエラーが起きるという話でした。

実行例で気付いた人もいると思いますが、 lilo.linux.or.jp のサーバーでの話で、他にも管理用の ML などがあるのですが、それらは description や info が空だったので問題が起きていなかったようです。

教訓としては、急いでいても apt-listchanges で最低限 NEWS だけはちゃんと読むようにした方が良いということでした。
