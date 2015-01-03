---
layout: post
title: "doorkeeper gem を 1.4 系から 2.0 系にアップデートした"
date: 2014-12-18 16:18:00 +0900
comments: true
categories: ruby rails doorkeeper
---
doorkeeper gem がメジャーバージョンアップして非互換があったので、
対応したときのメモです。

<!--more-->

## 対象バージョン

- rails 4.1.8
- doorkeeper 1.4.0 から 2.0.1

## 変更点確認

[CHANGELOG.md](https://github.com/doorkeeper-gem/doorkeeper/blob/master/CHANGELOG.md)
にまとまっていたので、参考にしました。

セキュリティアップデートも含まれるようなので、
2.0 系にあげられないときは 1.4.1 にあげておくのが良さそうです。

## `doorkeeper_for` の書き換え

`doorkeeper_for :all` は `before_action :doorkeeper_authorize!` に書き換えました。

```diff
-    doorkeeper_for :all
-    respond_to     :json
+    before_action :doorkeeper_authorize!
+    respond_to    :json
```

```diff
-    doorkeeper_for :index,  :show,   scopes: %w"public"
-    doorkeeper_for :create, :update, scopes: %w"admin write"
+    before_action -> { doorkeeper_authorize! :public }, only: [:index, :show]
+    before_action only: [:create, :update] do
+      doorkeeper_authorize! :admin, :write
+    end
```

## `config/initializers/doorkeeper.rb` の更新

`rails g doorkeeper:install` で生成し直して変更していた部分を再適用しました。

- `resource_owner_authenticator` で `devise` との連携
- scope 関連
- 認可の画面のスキップ
- 動的な query parameter の許可

```ruby
  resource_owner_authenticator do
    current_user || warden.authenticate!(scope: :user)
  end

  default_scopes  :public
  optional_scopes :admin, :write

  skip_authorization do |resource_owner, client|
    true
  end

  wildcard_redirect_uri true
```

## `config/locales/doorkeeper.en.yml` の更新

`rails g doorkeeper:install` で `config/initializers/doorkeeper.rb` と一緒に更新されていました。

認可の画面をスキップしている関係で、翻訳はせずに en を ja に置き換えただけで
そのまま使っていたので、同様に置き換えてそのまま使いました。

## `redirect urimust be an HTTPS/SSL URI.` 対策

テストの中で
`let!(:application) { Doorkeeper::Application.create!(name: 'MyApp', redirect_uri: 'http://api.test') }`
のようにしていると
`redirect urimust be an HTTPS/SSL URI.`
で失敗していたので、
`https` の URL に変更するか、
`config/initializers/doorkeeper.rb` で
`force_ssl_in_redirect_uri false`
にする必要がありました。

## Missing column: `applications.scopes`

テストを実行したときに

    [doorkeeper] Missing column: `applications.scopes`. If you are using ActiveRecord run `rails generate doorkeeper:application_scopes && rake db:migrate` to add it.

というメッセージが出るので、メッセージ通り
`rails generate doorkeeper:application_scopes && rake db:migrate`
で追加しました。
