---
layout: post
title: "require './file'とrequire_relative 'file'の違い"
date: 2016-07-26 21:16:07 +0900
comments: true
categories: ruby
---
最近 Debian の Perl が `CVE-2016-1238` ([#588017](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=588017 "#588017")) で更新されました。
似たような変更は Ruby だと [1.9.2 から `.` が `$:` (`$LOAD_PATH`) から削除された](https://www.ruby-lang.org/ja/news/2010/08/18/ruby-1-9-2-is-released/)ということがありました。
そして、明記はされていませんが `require_relative` が推奨されるようになったようです。

<!--more-->

## `require './file'` と `require_relative 'file'` の違い

`/var/tmp` と `/tmp` に確認用のファイルを以下のように準備して実行してみました。

```
$ head main.rb require.rb require_relative.rb /tmp/require.rb /tmp/require_relative.rb
==> main.rb <==
require './require'
require_relative 'require_relative'

==> require.rb <==
p __FILE__

==> require_relative.rb <==
p __FILE__

==> /tmp/require.rb <==
p __FILE__

==> /tmp/require_relative.rb <==
p __FILE__
$ ruby main.rb
"/var/tmp/require.rb"
"/var/tmp/require_relative.rb"
$ cd /tmp
$ ruby /var/tmp/main.rb
"/tmp/require.rb"
"/var/tmp/require_relative.rb"
```

結果を見ればわかるように `require './file'` はカレントディレクトリがどこなのかの影響を受けます。

このような書き方を使っていると `$LOAD_PATH` から `.` が取り除かれていても (Windows の) DLL hijacking vulnerability のような脆弱性の原因になるため、
`require_relative` を使う方が望ましいということになります。

## require_relative の歴史

いつから使えるようになったのか調べたので、ついでにメモしておきます。

- ruby 1.9.0-1 で `lib/require_relative.rb` が追加され、 `require 'require_relative'` すれば `require_relative` が使えるようになる。
- ruby 1.9.0-2 で `lib/require_relative.rb` から `prelude.rb` (ruby 本体に組み込まれて実行開始時に自動実行されるファイル) に移動して `require 'require_relative'` なしで `require_relative` が使えるようになる。
- ruby 1.9.2 で `doc/NEWS` に `Kernel#require_relative` が載る。 `require_relative` が `prelude.rb` から C 実装に置き換えられる。
