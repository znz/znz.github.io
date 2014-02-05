---
layout: post
title: "rails_best_practices 1.15.0でtryが壊れた"
date: 2014-02-05 10:35:06 +0900
comments: true
categories: ruby rails gem
---
今日の `bundle update` をしてみたところ、開発環境での devise でのログイン時に `devise (3.2.2) lib/devise/hooks/csrf_cleaner.rb, line 3` で `wrong number of arguments (2 for 1)` という `ArgumentError` が発生するようになりました。

その原因調査方法の記録です。

<!--more-->

例外が発生した行は以下の内容で、今まで問題なく動いていたし、特に問題はなさそうだったので、他の gem の影響だということがわかりました。

```ruby lib/devise/hooks/csrf_cleaner.rb
Warden::Manager.after_authentication do |record, warden, options|
  if Devise.clean_up_csrf_token_on_authentication
    warden.request.session.try(:delete, :_csrf_token)
  end
end
```

ソースコードを追いかけるのは難しそうだったので、 better_errors の live shell を使って調査しました。

```console
>> env.keys
=> 略
>> env["warden"]
=> Warden::Proxy:70192217236640 @config={:default_scope=>:user, :scope_defaults=>{}, :default_strategies=>{:user=>[:rememberable, :database_authenticatable]}, :intercept_401=>false, :failure_app=>#<Devise::Delegator:0x007fadcc44d438>}
>> env["warden"].request
=> #<ActionDispatch::Request:0x007fadd10c4dc0 略>
>> env["warden"].request.session
=> {"session_id"=>"略", "user_return_to"=>"/", "_csrf_token"=>"略", "warden.user.user.key"=>[[8], "$2a$10$略"]}
>> env["warden"].request.session.try(:delete, :_csrf_token)
!! #<ArgumentError: wrong number of arguments (2 for 1)>
```

`env["warden"]` 経由で再現しました。

次に原因を探るために `arity` を調べました。

```console
>> env["warden"].request.session.methods.grep /delete/
=> [:delete, :delete_if]
>> env["warden"].request.session.method("delete")
=> #<Method: Rack::Session::Abstract::SessionHash#delete>
>> env["warden"].request.session.method("delete").arity
=> 1
>> env["warden"].request.session.method("try")
=> #<Method: Rack::Session::Abstract::SessionHash(Object)#try>
>> env["warden"].request.session.method("try").arity
=> 1
>> env["warden"].request.session.method("try").source_location
=> ["略/rails_best_practices-1.15.0/lib/rails_best_practices/core_ext/object.rb", 7]
>>
```

rails_best_practices 1.15.0 が原因とわかったので issues を確認すると既に同様の現象の報告があったので、[コメントしておきました](https://github.com/railsbp/rails_best_practices/issues/202#issuecomment-34129010)。
