---
layout: post
title: "Mac OS X で owncloudcmd を使う"
date: 2014-05-02 22:18:39 +0900
comments: true
categories: owncloud osx
---
ownCloud のデスクトップクライアントには `owncloudcmd` というコマンドが含まれています。
昔は `ocsync` というコマンドが別途あったようですが、今は同梱されている `owncloudcmd` を使うのが主流のようです。

<!--more-->

## 対象バージョン

- クライアント : ownCloud client for Mac 1.5.4 に同梱の `owncloudcmd` で試しました。
- サーバー : 自前サーバーなら ownCloud 6.0.3 で、今回の例では [Teraクラウド](https://teracloud.jp/) を使ってみました。

## 概要

`path/to/owncloudcmd path/to/localdir ownclouds://example.jp/remote.php/webdav`
のようにローカルのディレクトリと `owncloud` の WebDAV 用の URL の先頭の `http` を `owncloud` に置き換えたもの (`https` なら `ownclouds`) を指定すると同期できます。

引数なしで実行するとオプションの説明が出ます。

## Mac OS X 版での修正

### `owncloudcmd` のリンクミス?

普通に使おうとすると `dyld: Library not loaded: libowncloudsync.0.dylib` で起動できません。

```console
% /Applications/owncloud.app/Contents/MacOS/owncloudcmd $HOME/teracloud ownclouds://teracloud.jp/remote.php/webdav
dyld: Library not loaded: libowncloudsync.0.dylib
  Referenced from: /Applications/owncloud.app/Contents/MacOS/owncloudcmd
  Reason: image not found
zsh: trace trap  /Applications/owncloud.app/Contents/MacOS/owncloudcmd
```

### ライブラリのパス修正

`otool -L` で確認すると相対パスがまずいとわかります。

```console
% otool -L /Applications/owncloud.app/Contents/MacOS/owncloudcmd
/Applications/owncloud.app/Contents/MacOS/owncloudcmd:
	libowncloudsync.0.dylib (compatibility version 0.0.0, current version 1.5.4)
	QtWebKit.framework/Versions/4/QtWebKit (compatibility version 4.9.0, current version 4.9.3)
	QtXmlPatterns.framework/Versions/4/QtXmlPatterns (compatibility version 4.8.0, current version 4.8.4)
	QtGui.framework/Versions/4/QtGui (compatibility version 4.8.0, current version 4.8.4)
	QtDBus.framework/Versions/4/QtDBus (compatibility version 4.8.0, current version 4.8.4)
	QtXml.framework/Versions/4/QtXml (compatibility version 4.8.0, current version 4.8.4)
	QtSql.framework/Versions/4/QtSql (compatibility version 4.8.0, current version 4.8.4)
	QtNetwork.framework/Versions/4/QtNetwork (compatibility version 4.8.0, current version 4.8.4)
	QtCore.framework/Versions/4/QtCore (compatibility version 4.8.0, current version 4.8.4)
	libocsync.0.dylib (compatibility version 0.0.0, current version 0.2.1)
	/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 125.2.11)
	/usr/local/opt/sqlite/lib/libsqlite3.0.dylib (compatibility version 9.0.0, current version 9.6.0)
	/usr/lib/libiconv.2.dylib (compatibility version 7.0.0, current version 7.0.0)
	/System/Library/Frameworks/CoreServices.framework/Versions/A/CoreServices (compatibility version 1.0.0, current version 44.0.0)
	/System/Library/Frameworks/Foundation.framework/Versions/C/Foundation (compatibility version 300.0.0, current version 751.63.0)
	/System/Library/Frameworks/AppKit.framework/Versions/C/AppKit (compatibility version 45.0.0, current version 1038.36.0)
	@loader_path/../Frameworks/Sparkle.framework/Versions/A/Sparkle (compatibility version 1.5.0, current version 1.5.0)
	/usr/local/lib/libqtkeychain.0.3.0.dylib (compatibility version 0.0.0, current version 0.3.0)
	/usr/local/opt/neon/lib/libneon.27.dylib (compatibility version 31.0.0, current version 31.0.0)
	/usr/lib/libstdc++.6.dylib (compatibility version 7.0.0, current version 7.9.0)
```

`install_name_tool -add_rpath` や `install_name_tool -delete_rpath` で試行錯誤したのですが、
`otool -l /Applications/owncloud.app/Contents/MacOS/owncloudcmd` で確認しても
`rpath` が後ろに追加されてきいていないようだったので、
`install_name_tool -change` で対応することにしました。

まず相対パスを修正しました。

```console
%  otool -L /Applications/owncloud.app/Contents/MacOS/owncloudcmd | awk '$1~"^lib"{print "install_name_tool -change " $1 " @executable_path/../MacOS/" $1 " /Applications/owncloud.app/Contents/MacOS/owncloudcmd" }' | sh -e
%  otool -L /Applications/owncloud.app/Contents/MacOS/owncloudcmd | awk '$1~"^Qt"{print "install_name_tool -change " $1 " @executable_path/../Frameworks/" $1 " /Applications/owncloud.app/Contents/MacOS/owncloudcmd" }' | sh -e
```

さらに `/usr/local/lib` をさしているものがあったのを修正しました。

```console
%  /Applications/owncloud.app/Contents/MacOS/owncloudcmd
dyld: Library not loaded: /usr/local/lib/libqtkeychain.0.3.0.dylib
  Referenced from: /Applications/owncloud.app/Contents/MacOS/owncloudcmd
  Reason: image not found
zsh: trace trap  /Applications/owncloud.app/Contents/MacOS/owncloudcmd
%  find /Applications/owncloud.app -name libqtkeychain.0.3.0.dylib
/Applications/owncloud.app/Contents/MacOS/libqtkeychain.0.3.0.dylib
%  install_name_tool -change /usr/local/lib/libqtkeychain.0.3.0.dylib /Applications/owncloud.app/Contents/MacOS/libqtkeychain.0.3.0.dylib /Applications/owncloud.app/Contents/MacOS/owncloudcmd
%  /Applications/owncloud.app/Contents/MacOS/owncloudcmd
dyld: Library not loaded: /usr/local/opt/neon/lib/libneon.27.dylib
  Referenced from: /Applications/owncloud.app/Contents/MacOS/owncloudcmd
  Reason: image not found
zsh: trace trap  /Applications/owncloud.app/Contents/MacOS/owncloudcmd
%  find /Applications/owncloud.app -name libneon.27.dylib
/Applications/owncloud.app/Contents/MacOS/libneon.27.dylib
%  install_name_tool -change /usr/local/opt/neon/lib/libneon.27.dylib /Applications/owncloud.app/Contents/MacOS/libneon.27.dylib /Applications/owncloud.app/Contents/MacOS/owncloudcmd
```

これで余計なメッセージは出ますが、使えるようになりました。

```console
%  /Applications/owncloud.app/Contents/MacOS/owncloudcmd
objc[40210]: Class WebCoreMovieObserver is implemented in both /System/Library/Frameworks/WebKit.framework/Versions/A/Frameworks/WebCore.framework/Versions/A/WebCore and /Applications/owncloud.app/Contents/Frameworks/QtWebKit.framework/Versions/4/QtWebKit. One of the two will be used. Which one is undefined.
objc[40210]: Class WebCoreSharedBufferData is implemented in both /System/Library/Frameworks/WebKit.framework/Versions/A/Frameworks/WebCore.framework/Versions/A/WebCore and /Applications/owncloud.app/Contents/Frameworks/QtWebKit.framework/Versions/4/QtWebKit. One of the two will be used. Which one is undefined.
objc[40210]: Class WebVideoFullscreenWindow is implemented in both /System/Library/Frameworks/WebKit.framework/Versions/A/Frameworks/WebCore.framework/Versions/A/WebCore and /Applications/owncloud.app/Contents/Frameworks/QtWebKit.framework/Versions/4/QtWebKit. One of the two will be used. Which one is undefined.
objc[40210]: Class WebVideoFullscreenController is implemented in both /System/Library/Frameworks/WebKit.framework/Versions/A/Frameworks/WebCore.framework/Versions/A/WebCore and /Applications/owncloud.app/Contents/Frameworks/QtWebKit.framework/Versions/4/QtWebKit. One of the two will be used. Which one is undefined.
objc[40210]: Class WebVideoFullscreenHUDWindowController is implemented in both /System/Library/Frameworks/WebKit.framework/Versions/A/Frameworks/WebCore.framework/Versions/A/WebCore and /Applications/owncloud.app/Contents/Frameworks/QtWebKit.framework/Versions/4/QtWebKit. One of the two will be used. Which one is undefined.
objc[40210]: Class WebVideoFullscreenHUDWindow is implemented in both /System/Library/Frameworks/WebKit.framework/Versions/A/Frameworks/WebCore.framework/Versions/A/WebCore and /Applications/owncloud.app/Contents/Frameworks/QtWebKit.framework/Versions/4/QtWebKit. One of the two will be used. Which one is undefined.
objc[40210]: Class WebWindowFadeAnimation is implemented in both /System/Library/Frameworks/WebKit.framework/Versions/A/Frameworks/WebCore.framework/Versions/A/WebCore and /Applications/owncloud.app/Contents/Frameworks/QtWebKit.framework/Versions/4/QtWebKit. One of the two will be used. Which one is undefined.
objc[40210]: Class WebWindowScaleAnimation is implemented in both /System/Library/Frameworks/WebKit.framework/Versions/A/Frameworks/WebCore.framework/Versions/A/WebCore and /Applications/owncloud.app/Contents/Frameworks/QtWebKit.framework/Versions/4/QtWebKit. One of the two will be used. Which one is undefined.
owncloudcmd - command line ownCloud client tool.

Usage: owncloudcmd <sourcedir> <owncloudurl>

A proxy can either be set manually using --httpproxy or it
uses the setting from a configured sync client.

Options:
  --confdir = configdir: Read config from there.
  --httpproxy = proxy:   Specify a http proxy to use.
                         Proxy is http://server:port

zsh: exit 1     /Applications/owncloud.app/Contents/MacOS/owncloudcmd
```

## 使い方

概要のところに書いたように、
普通に `owncloudcmd sourcedir owncloudurl` で実行するとユーザー名とパスワードを聞かれます。
URL に `ownclouds://user:password@example.jp/remote.php/webdav` のようにユーザー名とパスワードを埋め込むことも出来ますが、あまりコマンドラインにパスワードは書きたくないことが多いと思います。

しかし、
[owncloudcmd のオプション処理部分のソース](https://github.com/owncloud/mirall/blob/f72e1cc8375b72c97d6566c6875f7214415cea9c/src/owncloudcmd/owncloudcmd.cpp)
をみても `"--confdir"` は読み込むだけで使われていないようなので、
他の方法で指定することは出来なさそうでした。

## まとめ

`owncloudcmd` を使えば GUI クライアントで設定しているサーバー以外のサーバーと同期したり、
サーバーなどのコマンドラインのみの環境でも `owncloud` との同期が出来ることがわかりました。

しかし、クライアントの出来がまだあまりよくないので、使い勝手は良くないということもわかりました。
