---
layout: post
title: "Ubuntu 14.04.5 (trusty) から 16.04.3 (xenial) に更新したら subversion が壊れた話"
date: 2018-01-22 21:17:00 +0900
comments: true
categories: ubuntu linux
---
subversion のレポジトリを置いているサーバーを Ubuntu 14.04.5 (trusty) から 16.04.3 (xenial) に更新したら [Old repository can't be read: svn: E125012: Invalid character in hex checksum](https://bugs.launchpad.net/ubuntu/+source/subversion/+bug/1639406) の影響でコミットできなくなったので、他の環境にコピーして直した話です。

<!--more-->

## 環境

- 壊れた環境: Ubuntu 14.04.5 (trusty) から 16.04.3 (xenial)
- svnadmin dump に使った環境: Debian GNU/Linux 9

## 現象

レポジトリを置いているサーバーの Ubuntu のバージョンをあげたら `svn: E125012: Invalid character in hex checksum` とでるようになって svn ci などができなくなりました。

[Old repository can't be read: svn: E125012: Invalid character in hex checksum](https://bugs.launchpad.net/ubuntu/+source/subversion/+bug/1639406) によると subversion の upstream ではなおっていて、 Ubuntu xenial では1年以上も放置されていて、修正される望みは薄いと思いました。

## 復旧作業

svnadmin dump/load で復旧できるとあったので、 svnadmin dump しようとしましたが、壊れた環境では svnadmin dump もできなかったので、一時的に他の環境に持っていって dump することにしました。

- 壊れた環境で tarball 作成: `time tar acvf ~/repo.tar.xz repo`
- 別環境にコピー: `scp svn@server:repo.tar.xz .`
- 別環境でダンプ: `tar xf repo.tar.xz; time svnadmin dump repo > repo.svndump`
- ダンプをコピー: `xz -k repo.svndump; ls -lh repo.svndump*; scp repo.svndump.xz svn@server:`
- load: `mv repo repo-2017; rm -rf repo-2017; svnadmin create repo; xzcat repo.svndump.xz | svnadmin load repo`
- (一度 mv しているのは履歴で最新のレポジトリを消してしまう事故を防ぐため)


- 以下のエラーが出たのでオプションを追加して再実行: `mv repo repo-2017; rm -rf repo-2017; svnadmin create repo; xzcat repo.svndump.xz | svnadmin load --bypass-prop-validation repo`

```
    svnadmin: E125005: Invalid property value found in dumpstream; consider repairing the source or using --bypass-prop-validation while loading.
    svnadmin: E125005: Cannot accept 'svn:log' property because it is not encoded in UTF-8
```

## まとめ

E125012 の復旧方法についての日本語の記事が見当たらなかったので、まとめると svnadmin dump/load するというだけですが、書いてみました。
