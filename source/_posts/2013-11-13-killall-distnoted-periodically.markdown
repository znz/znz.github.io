---
layout: post
title: "launchdでdistnotedを定期的に終了させる"
date: 2013-11-13 18:23
comments: true
categories: osx mavericks
---
Mac OS X 10.9 Mavericks で気がつくと
`distnoted` というプロセスのメモリ消費が増えて
大変なことになっていることがあって、
気がついた時は手動で `killall distnoted` で対処したり、
OS 自体を再起動したりしていました。

たまに気がつかないうちに大量にメモリを消費して、
確認のためのアクティビティモニタを開くのも大変なことがあったので、
さすがにまずいと思って `launchd` で定期的に実行するようにしました。

2013-12-28 追記:
続きとして
[コメントにあったパッチを試してみた話](/blog/2013-12-27-emacs-inline-patch.html)
を書きました。

<!--more-->

## distnoted とは?

man によると `distributed notification server` というものらしいのですが、
詳細はよくわかりませんでした。

man には自動で起動するものなので、
手動で起動するものではないとは書いてありました。

必要に応じて自動で起動してくるので、
`killall` などで止めてしまっても問題が無いという情報は
どこかでみかけました。

`ps` でプロセスを確認すると `root` 権限で
` /usr/sbin/distnoted daemon` が動いていて、
他にいくつかの
`/usr/sbin/distnoted agent`
がユーザー権限で動いていました。
そのうちの1個がログインしたユーザーの権限で動いていて、
それがメモリを大量に消費していて、
`killall distnoted`
ではそのプロセスだけを終了させています。

## launchd による定期実行

`launchd` で定期的に実行するには


```xml ~/Library/LaunchAgents/local.killall.distnoted.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>local.killall.distnoted</string>
	<key>ProgramArguments</key>
	<array>
		<string>/usr/bin/killall</string>
		<string>distnoted</string>
	</array>
	<key>StartInterval</key>
	<integer>600</integer>
</dict>
</plist>
```

という内容のファイルを
`~/Library/LaunchAgents/local.killall.distnoted.plist`
に作成して
`launchctl load ~/Library/LaunchAgents/local.killall.distnoted.plist`
で反映します。

設定を変更したときは

```
launchctl unload ~/Library/LaunchAgents/local.killall.distnoted.plist
launchctl load ~/Library/LaunchAgents/local.killall.distnoted.plist
```

で反映します。

`StartInterval` は秒単位なので `600` だと10分間隔です。
