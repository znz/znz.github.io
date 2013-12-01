---
layout: post
title: "Ruby 2.1.0の新機能のException#cause"
date: 2013-12-03 00:00
comments: true
categories: ruby
---
[Ruby 2.1.0-preview2 リリース](https://www.ruby-lang.org/ja/news/2013/11/22/ruby-2-1-0-preview2-is-released/)
では
[2013-11-10 の大きめの変更](http://d.hatena.ne.jp/nagachika/20131110/ruby_trunk_changes_43615_43636)
が気になっていて、
ここでは
`Exception#cause`
というメソッドを紹介します。

`"literal"f` のように書く freeze された文字列リテラル機能が
2.1.0-preview1 には入っていたのですが、それが削除されて
`"literal".freeze` がコンパイル時に特別扱いされるようになった、
というのも気になっています。

<!--more-->

## Exception#cause の例

Ruby 2.0.0 までは `rescue` や `ensure` の中で別の例外が発生すると、
別途保存しておかない限り、
以前に発生した例外がわからなくなってしまっていましたが、
Ruby 2.1.0(-preview2) からは別の例外を `raise` した時に
以前の例外が自動で保存されて
`Exception#cause` でたどれるようになりました。
`cause` は `raise` のタイミングで設定されるので、
例外オブジェクト自体は `rescue` や `ensure`
の外で生成していても良いようです。

```ruby
#!/usr/bin/env ruby
def foo
  raise "foo"
end

def bar
  e = Exception.new("bar")
  foo
rescue
  raise e
end

def baz
  bar
ensure
  raise "baz"
end

begin
  baz
rescue
  p $!                   #=> #<RuntimeError: baz>
  p $!.cause             #=> #<Exception: bar>
  p $!.cause.cause       #=> #<RuntimeError: foo>
  p $!.cause.cause.cause #=> nil
end
```

## 終了時のバックトレース

ちなみに、例外が保存されていても
`rescue` せずにプログラムが終了した時のバックトレースは
今まで通り最後の例外だけ表示されるようです。

```console
% cat t.rb
#!/usr/bin/env ruby
def foo
  raise "foo"
end

def bar
  foo
rescue
  raise "bar"
end

bar
% ruby t.rb
t.rb:9:in `rescue in bar': bar (RuntimeError)
	from t.rb:7:in `bar'
	from t.rb:12:in `<main>'
```
