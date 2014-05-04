---
layout: post
title: "homebrew cask で sylpheed test1 をインストールした"
date: 2014-04-21 23:00:00 +0900
comments: true
categories: osx homebrew sylpheed
---
[Sylpheed 3.4.1-test1 for Mac OS X (テスト版)](http://sylpheed.sraoss.jp/ja/news.html)
がリリースされていたので、
homebrew cask を使ってインストールしてみました。

dmg で配布されているアプリケーションを homebrew cask に対応させる例としても参考になると思います。

<!--more-->

## homebrew cask のインストール

[Homebre](http://brew.sh/)
と
[homebrew-cask](https://github.com/phinze/homebrew-cask)
をあらかじめインストールしておきます。

## cask 作成

`brew cask create sylpheed-test` で作成して、
内容を以下のように変更しました。

```ruby
class SylpheedTest < Cask
  url 'http://sylpheed.sraoss.jp/sylpheed/macosx/Sylpheed_3.4.1-test1.dmg'
  homepage 'http://sylpheed.sraoss.jp/'
  version '3.4.1-test1'
  sha256 '52505ab7913fd58fa921a9c8b574286f08073321b440b7aa23dba3896b4b2ddc'
  link 'Sylpheed.app'
end
```

`sha256` は
[Cask Stanzas](https://github.com/phinze/homebrew-cask/blob/master/CONTRIBUTING.md#cask-stanzas)
の説明にあるように別途 `url` に指定したファイルをダウンロードして `shasum -a 256 <file>` で調べました。
もし upstream のリリースアナウンスに書いてあれば、それをそのまま使えば良いと思います。

編集途中で閉じてしまった場合は
`brew cask edit sylpheed-test`
で再編集できます。

実体は `$(brew --prefix)/Library/Taps/phinze-cask/Casks/sylpheed-test.rb` にあります。

## インストール

`sylpheed-test.rb` が作成できたら `brew cask install sylpheed-test` でインストールできます。

```console
% brew cask install sylpheed-test
==> Downloading http://sylpheed.sraoss.jp/sylpheed/macosx/Sylpheed_3.4.1-test1.dmg
######################################################################## 100.0%
==> Symlinking App 'Sylpheed.app' to '/Users/kazu/Applications/Sylpheed.app'
🍺  sylpheed-test installed to '/opt/homebrew-cask/Caskroom/sylpheed-test/3.4.1-test1' (314 files, 32M)
```

## 起動

Launchpad から起動したり、Spotlight で検索したりして普通に起動します。

初回起動で
`~/Library/Application Support/Sylpheed/Mailboxes/Mail`
を作るかきかれましたが、その時点で既に
`~/Library/Application Support/Sylpheed`
は作成されていました。

## まとめ

テスト版なので、まだ変なところはあるようですが (Command+V がメッセージビューのトグルになってペーストできないとか)
Mac OS X 版特有の問題があるだけのようなので、今後の正式版に期待しています。

まだ test 版なので pull request はしていないのですが、
正式版が出たら homebrew cask に pull request を送るかもしれません。
beta 版なども対応するなら
[homebrew-cask-versions](https://github.com/caskroom/homebrew-versions)
に pull request を送ると良さそうです。
