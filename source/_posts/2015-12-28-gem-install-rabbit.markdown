---
layout: post
title: "gem install rabbit に失敗したのを解決した"
date: 2015-12-28 18:48:40 +0900
comments: true
categories: ruby
---
Mac OS X 上で `gem install rabbit` でひっかかったので、その解決方法のメモです。

<!--more-->

## gobject-introspection

http://www.cozmixng.org/~w3ml/index.rb/rabbit-shocker/msg/1283
( https://gist.github.com/znz/59ae1b8785a640916ce9 )
のようなエラーで gobject-introspection-3.0.7 のインストールで失敗したので、
ML で教えてもらったように
`PKG_CONFIG_PATH=/usr/local/opt/libffi/lib/pkgconfig gem install rabbit`
でインストールすると進むようになりました。

## poppler

次に poppler gem (3.0.7) のエラーでひっかかったのですが ( http://www.cozmixng.org/~w3ml/index.rb/rabbit-shocker/msg/1285 )、
https://bugs.launchpad.net/poppler-python/+bug/1528489 で同じエラーが python binding で報告されていて、原因は poppler-0.39.0 での変更ということだったので、
poppler-0.38.0 に戻すことにした。

まず homebrew のディレクトリに移動して `Formula/poppler.rb` のログをみて poppler 0.38.0 に戻せる commit hash を調べて checkout して、インストールし直して、 poppler は 0.38.0 で止めておきました。

```
    cd /usr/local
    git log Library/Formula/poppler.rb
	git checkout 6f7b9d57b25451e811e8901c560d17abd7a464f9
	brew reinstall poppler
    git checkout master
    brew pin poppler
```

問題が解決したら `brew unpin poppler` でまた最新に戻せます。

ruby-gnome2 の方でも https://github.com/ruby-gnome2/ruby-gnome2/commit/3dda85661515d71101f1028dc7d68d4e53de45b1 でなおっているので、
poppler gem の新しいバージョンがリリースされたら解決しそうです。

## 成功

poppler をダウングレードして `PKG_CONFIG_PATH=/usr/local/opt/libffi/lib/pkgconfig gem install rabbit` で rabbit のインストールは成功しました。
