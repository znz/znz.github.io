---
layout: post
title: "bundled gem と default gem の違いの具体例"
date: 2016-09-13 23:33:33 +0900
comments: true
categories: ruby
---
[bundled gem と default gem の違い](/blog/2016-09-10-bundled-gem-and-default-gem.html)で概要は説明しましたが、具体的にどうなっているのか試してみました。

<!--more-->

## 動作確認環境

- Mac OS X Yosemite (10.10.5)
- homebrew
- rbenv 1.0.0
- ruby-build v20160913
- ruby 2.4.0-preview2, 2.3.1
- bundler 1.13.0, 1.12.5
- activesupport 5.0.0.1
- rdoc 4.2.1, 5.0.0.beta2

## クリーンな 2.4.0-preview2 を準備

homebrew で入れた rbenv + ruby-build を使って 2.4.0-preview2 をインストールしました。
bundler も必要なのでインストールして、普通の gem の例として activesupport も入れておきました。

```
% rbenv install 2.4.0-preview2
Downloading ruby-2.4.0-preview2.tar.bz2...
-> https://cache.ruby-lang.org/pub/ruby/2.4/ruby-2.4.0-preview2.tar.bz2
Installing ruby-2.4.0-preview2...
Installed ruby-2.4.0-preview2 to /Users/kazu/.rbenv/versions/2.4.0-preview2
% rbenv shell 2.4.0-preview2
% gem list

*** LOCAL GEMS ***

bigdecimal (default: 1.2.8)
did_you_mean (1.0.2)
io-console (default: 0.4.6)
json (default: 2.0.2)
minitest (5.9.0)
net-telnet (0.1.1)
openssl (default: 2.0.0.beta.2)
power_assert (0.3.1)
psych (default: 2.1.1)
rake (11.2.2)
rdoc (default: 5.0.0.beta2)
test-unit (3.2.1)
xmlrpc (0.1.1)
% gem install bundler
Fetching: bundler-1.13.0.gem (100%)
Successfully installed bundler-1.13.0
1 gem installed
% gem install activesupport
Fetching: i18n-0.7.0.gem (100%)
Successfully installed i18n-0.7.0
Fetching: thread_safe-0.3.5.gem (100%)
Successfully installed thread_safe-0.3.5
Fetching: tzinfo-1.2.2.gem (100%)
Successfully installed tzinfo-1.2.2
Fetching: concurrent-ruby-1.0.2.gem (100%)
Successfully installed concurrent-ruby-1.0.2
Fetching: activesupport-5.0.0.1.gem (100%)
Successfully installed activesupport-5.0.0.1
5 gems installed
```

## bundler なしの環境での require

普通の状態ではどれも問題なく require できます。

```
% rbenv exec irb -r irb/completion --simple-prompt
>> require 'json' # default gem
=> true
>> require 'xmlrpc' # bundled gem
=> true
>> require 'uri' # stdlib
=> true
>> require 'active_support/all' # normal gem
=> true
```

## bundler 環境でのテスト

bundler 環境下では bundled gem は普通の gem と同じように読み込めないことがわかります。

```
% mkdir /tmp/test
% cd /tmp/test
% bundle init
Writing new Gemfile to /private/tmp/test/Gemfile
% bundle exec irb -r irb/completion --simple-prompt
>> require 'json' # default gem
=> true
>> begin; require 'xmlrpc'; rescue LoadError; $!; end # bundled gem
=> #<LoadError: cannot load such file -- xmlrpc>
>> require 'uri' # stdlib
=> false
>> begin; require 'active_support/all'; rescue LoadError; $!; end # normal gem
=> #<LoadError: cannot load such file -- active_support/all>
>>
```

## uninstall

default gem は uninstall ができなくて、bundled gem は uninstall できることがわかります。

```
% gem uninstall json
ERROR:  While executing gem ... (Gem::InstallError)
    gem "json" cannot be uninstalled because it is a default gem
% gem uninstall xmlrpc
Successfully uninstalled xmlrpc-0.1.1
```

## Gemfile でバージョン指定

`Gemfile` でバージョン指定していれば default gem の代わりに指定したバージョンの gem が使われることがわかります。

```
% echo "gem 'json', '2.0.0'" >> Gemfile
% bundle install
Fetching gem metadata from https://rubygems.org/.
Fetching version metadata from https://rubygems.org/
Resolving dependencies...
Installing json 2.0.0 with native extensions
Using bundler 1.13.0
Bundle complete! 1 Gemfile dependency, 2 gems now installed.
Use `bundle show [gemname]` to see where a bundled gem is installed.
% bundle exec irb -r irb/completion --simple-prompt
>> require 'json'
=> true
>> JSON::VERSION
=> "2.0.0"
```

## おかしなことが起こる例

ruby 2.4 だと古い json 1.x が入らない関係で rdoc 4.x が入らなかったので、
ここだけ使用中の ruby 2.3.1 で検証しました。

### 準備

rubygems が古いため、 `default:` は付いていませんが、 4.2.1 が default gem です。

```
% rbenv shell 2.3.1
% echo "gem 'rdoc', '= 5.0.0.beta2'" >> Gemfile
% bundle install
Warning: the running version of Bundler is older than the version that created the lockfile. We suggest you upgrade to the latest version of Bundler by running `gem install bundler`.
Fetching gem metadata from https://rubygems.org/
Fetching version metadata from https://rubygems.org/
Resolving dependencies...
Installing json 2.0.0 with native extensions
Installing rdoc 5.0.0.beta2
Using bundler 1.12.5
Bundle complete! 2 Gemfile dependencies, 3 gems now installed.
Use `bundle show [gemname]` to see where a bundled gem is installed.
% gem list rdoc

*** LOCAL GEMS ***

rdoc (5.0.0.beta2, 4.2.1)
```

### rdoc 5 で減ったファイルを探す

探してみたところ、 `rdoc/markdown/literals_1_9.rb` が `rdoc/markdown/literals.rb` に改名されていてなくなっていたのを見つけたので、
これを使っておかしな挙動を確かめてみました。

### bundler 環境下で確認

bundler 環境下では `Gemfile` で指定した 5.0.0.beta2 が読み込まれるのがわかります。

ここまでは良いのですが、 5.0.0.beta2 ではなくなっているファイルを `require 'rdoc/markdown/literals_1_9'` で読み込もうとすると、
default gem の 4.2.1 のファイルが (組み合わせがおかしいので警告が出つつ) 読み込めてしまいます。

`$LOAD_PATH` を確認してみると、そういう挙動になる理由はわかるのですが、増減するファイルによっては何か気づきにくい問題が起きるかもしれません。

```
% bundle exec irb -r irb/completion --simple-prompt
>> require 'rdoc'
=> true
>> RDoc::VERSION
=> "5.0.0.beta2"
>> require 'rdoc/markdown/literals_1_9'
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/2.3.0/rdoc/markdown/literals_1_9.rb:413: warning: already initialized constant RDoc::Markdown::Literals::Rules
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/gems/2.3.0/gems/rdoc-5.0.0.beta2/lib/rdoc/markdown/literals.rb:409: warning: previous definition of Rules was here
=> true
>> puts $LOAD_PATH
/usr/local/Cellar/rbenv/1.0.0/rbenv.d/exec/gem-rehash
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/gems/2.3.0/gems/bundler-1.12.5/lib
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/gems/2.3.0/gems/rdoc-5.0.0.beta2/lib
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/gems/2.3.0/gems/json-2.0.0/lib
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/gems/2.3.0/extensions/x86_64-darwin-14/2.3.0-static/json-2.0.0
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/site_ruby/2.3.0
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/site_ruby/2.3.0/x86_64-darwin14
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/site_ruby
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/vendor_ruby/2.3.0
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/vendor_ruby/2.3.0/x86_64-darwin14
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/vendor_ruby
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/2.3.0
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/2.3.0/x86_64-darwin14
=> nil
```

### gem which で確認

`gem which` でどのファイルが `require` で読み込まれるのか確認できるので、参考になるかもしれません。

```
% gem which rdoc/markdown/entities
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/2.3.0/rdoc/markdown/entities.rb
% bundle exec gem which rdoc/markdown/entities
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/gems/2.3.0/gems/rdoc-5.0.0.beta2/lib/rdoc/markdown/entities.rb
% bundle exec gem which rdoc/markdown/literals_1_9
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/2.3.0/rdoc/markdown/literals_1_9.rb
% bundle exec gem which rdoc/markdown/literals
/Users/kazu/.rbenv/versions/2.3.1/lib/ruby/gems/2.3.0/gems/rdoc-5.0.0.beta2/lib/rdoc/markdown/literals.rb
```

## 普通は問題にならないであろう問題点

基本的には Ruby コミッターしか影響しないはずなので、単なるおまけです。

どこに報告すればいいのかわからなかったので、とりあえず gist にメモを書いた ([rubygems with multi default gem problem](https://gist.github.com/znz/62946cbb55de4fe58a5101d7875a2ba1)) のですが、Ruby コミッターのように開発版を上書きインストールし続けている環境を持っていると、
default gem として複数バージョンの gemspec を持ってしまうことがあり、実際には後から `make install` した方しか入っていないので、ダミーの gemspec だけ残っているバージョンを指定した時におかしなことになるという話です。

クリーンインストールしなおすなり、 `$(gem env gemdir)/specifications/default` の古い gemspec を消すなりすれば良いだけなので、そんなに困る問題でもないです。

## まとめ

bundled gem と default gem の違いを実際の動作を元に比べてみました。
また、 bundler と組み合わせて問題が起きる可能性がある例をみてみました。
通常の使い方では問題が起きることはないと思いますが、トリビア的に知っておくとおもしろいかもしれません。
