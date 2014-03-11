---
layout: post
title: "Ruby 2.1.1のhash#rejectの不具合対策"
date: 2014-03-11 14:22:28 +0900
comments: true
categories: ruby rails
---
[Ruby 2.1.1 に含まれる Hash#reject の不具合について](https://www.ruby-lang.org/ja/news/2014/03/10/regression-of-hash-reject-in-ruby-2-1-1/)
という話があって、
自分で書くコードでは普通は Hash を継承することはない (is-a にせずに has-a にすることが多い) ので影響はないのですが、
Rails の中の ActiveSupport が影響を受けるということで対処を入れました。

他にもバックポート漏れのコミットをあてた独自ビルドを使うとか、
Ruby 2.1.0 を使う、という対処もあるようです。

<!--more-->

[Rails の Issue #14188](https://github.com/rails/rails/issues/14188)
で話があるようにセキュリティ修正ではないため、
Rails 3.2 系にこの修正は入らないので、
モンキーパッチを入れることにしました。

bugs.ruby-lang.org では
https://github.com/ruby/bugs.ruby-lang.org/blob/ro-2-5/lib/redmine/core_ext.rb
のように対処していたのを参考にして、
以下のようなコードを
`config/initializers/hash_reject.rb`
に置いて対処することにしました。

```ruby config/initializers/hash_reject.rb
# monkey patch for regression of Hash#reject in Ruby 2.1.1
# see https://www.ruby-lang.org/ja/news/2014/03/10/regression-of-hash-reject-in-ruby-2-1-1/
# or https://www.ruby-lang.org/en/news/2014/03/10/regression-of-hash-reject-in-ruby-2-1-1/
require 'active_support/hash_with_indifferent_access'
require 'active_support/ordered_hash'

module ActiveSupport
  class HashWithIndifferentAccess
    def select(*args, &block)
      dup.tap { |hash| hash.select!(*args, &block) }
    end

    def reject(*args, &block)
      dup.tap { |hash| hash.reject!(*args, &block) }
    end
  end

  class OrderedHash
    def select(*args, &block)
      dup.tap { |hash| hash.select!(*args, &block) }
    end

    def reject(*args, &block)
      dup.tap { |hash| hash.reject!(*args, &block) }
    end
  end
end
```

ここのコードが実行される前に読み込まれていれば問題ないのですが、
ここでクラス定義してしまうと元々の定義が autoload されなくなるので、
念のため require しています。
