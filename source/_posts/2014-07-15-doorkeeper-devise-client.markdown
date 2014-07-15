---
layout: post
title: "doorkeeper providerサンプルアプリに対応するOAuthクライアントをdeviseで作成した"
date: 2014-07-15 18:50:40 +0900
comments: true
categories: ruby rails doorkeeper devise oauth2
---

[doorkeeper-provider-app](https://github.com/znz/doorkeeper-provider-app)
を使って SSO (Single Sign On) のように使うクライアントアプリを作成しました。
[doorkeeper-devise-client-app](https://github.com/znz/doorkeeper-devise-client-app)
で公開しています。

SSO は OAuth 2.0 の本来の使い方ではないので、不便な部分もありますが、
クライアント側の例として参考になると思います。

<!--more-->

## 動作確認バージョン

- ruby 2.1.2
- rails 4.1.4
- bootstrap 3.2.0
- devise 3.2.4
- omniauth 1.2.2
- omniauth-oauth2 1.2.0
- bootstrap-sass 3.2.0.0
- dotenv-rails 0.11.1

## 簡単な役割解説

provider は doorkeeper gem を入れている rails アプリ側で認証や認可を受け持ちます。
(OAuth の仕様的には認証と認可が別々のサーバーのこともあります。)

ここでいう OAuth クライアントは devise + omniauth + omniauth-oauth2 を使った rails アプリのことです。
ブラウザーなどのユーザー側にあるクライアントではなく、ユーザーから見れば、これもサーバーです。

詳しいことは OAuth 2.0 の仕様を調べてください。

## 大まかな流れ

1. `lib/omniauth/strategies/doorkeeper.rb` 作成
2. `config/initializers/devise.rb` で `config.omniauth :doorkeeper, ...`
3. `app/controllers/users/omniauth_callbacks_controller.rb` 作成
   (`uid` でユーザーを自動作成したり、 `access_token` (`credentials.token`) を保存したり)
4. `config/routes.rb` に設定
5. `user` に `provider` を追加
6. `app/models/user.rb` で `devise :omniauthable` や `uid` を使った処理を実装
7. `OAuth2::AccessToken` を生成
8. それを使って API アクセス

`access_token` を session に保存するかデータベースに保存するかは
アプリケーションのポリシー次第になります。
このアプリでは session に保存しています。

別途 callback uri として `http://localhost:3000/users/auth/doorkeeper/callback` のような URL を指定して
doorkeeper 側の `oauth/applications` に登録しておく必要があります。

## `lib/omniauth/strategies/doorkeeper.rb` 作成

例として
[Dookreeper Devise+Omniauth Client](https://github.com/doorkeeper-gem/doorkeeper-devise-client "Dookreeper Devise+Omniauth Client")
と比較して `info` に `name` を増やしています。

コントローラーを `users` の下の `Users::OmniauthCallbacksController` にしたので、
戻り先の `authorize_path` は `'/oauth/authorize'` ではなく `'/users/oauth/authorize'` になっています。

info のハッシュはサーバーから受け取れていて、後の処理でもっと欲しい情報があれば自由に増やせます。

```ruby lib/omniauth/strategies/doorkeeper.rb
module OmniAuth
  module Strategies
    class Doorkeeper < OmniAuth::Strategies::OAuth2
      option :name, :doorkeeper

      option :client_options, {
        site: 'http://localhost:4000',
        authorize_path: '/users/oauth/authorize'
      }

      uid do
        raw_info['id']
      end

      info do
        {
          email: raw_info['email'],
          name: raw_info['name'],
        }
      end

      def raw_info
        @raw_info ||= access_token.get('/api/v1/me.json').parsed || {}
      end
    end
  end
end
```

## `config/initializers/devise.rb` で設定

[サンプルアプリ](https://github.com/doorkeeper-gem/doorkeeper-devise-client)
では

```ruby config/initializers/devise.rb
  config.omniauth :doorkeeper,  DOORKEEPER_APP_ID, DOORKEEPER_APP_SECRET, :client_options =>  {:site => DOORKEEPER_APP_URL}
```

となっていました。

`scope` も追加すると以下のようになります。
`dotenv` を使って `ENV` から取るようにしました。

```ruby config/initializers/devise.rb
  config.omniauth :doorkeeper, ENV['DOORKEEPER_APP_ID'], ENV['DOORKEEPER_APP_SECRET'], client_options: {site: ENV['DOORKEEPER_APP_URL'] }, scope: 'public write'
```

`scope` は `'public,write'` だと `The requested scope is invalid, unknown, or malformed.` というエラーになってしまったので、
`,` 区切りではなくスペース区切りにしています。

## `app/controllers/users/omniauth_callbacks_controller.rb` 作成

callback で認証結果を受け取る部分を作成します。
ここで認証結果を受け取って、ユーザーを必要に応じてひも付けたり、
後で API アクセスに使うアクセストークンを保存したりします。

認証に失敗した時はログイン画面 (あれば) か `root_path` に戻すようにしています。

```ruby app/controllers/users/omniauth_callbacks_controller.rb
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  # https://github.com/plataformatec/devise/issues/2432
  protect_from_forgery except: :doorkeeper
  skip_filter :auto_authenticate_omniauth_user!, only: :doorkeeper

  def doorkeeper
    # You need to implement the method below in your model (e.g. app/models/user.rb)
    oauth_data = request.env['omniauth.auth']
    @user = User.find_or_create_for_doorkeeper_oauth(oauth_data)
    session[:doorkeeper_token] = oauth_data['credentials']['token']

    if @user.persisted?
      sign_in_and_redirect @user, :event => :authentication #this will throw if @user is not activated
      if is_navigational_format?
        set_flash_message(:notice, :success, kind: ENV['DOORKEEPER_APP_NAME'] || 'Doorkeeper')
        # hide flash message after auto sign in
        #flash.delete(:notice)
      end
    else
      session['devise.doorkeeper_data'] = request.env['omniauth.auth']
      if respond_to?(:new_user_registration_url)
        redirect_to new_user_registration_url
      else
        redirect_to root_url
      end
    end
  end

  def after_omniauth_failure_path_for(scope)
    if respond_to?(:new_session_path)
      new_session_path(scope)
    else
      root_path
    end
  end
end
```

自動ログイン後のメッセージが不要なら `set_flash_message` の部分を `flash.delete(:notice)` に置き換えます。
「Doorkeeper でログインしました」だとどのサイトか区別がつかないので、
`ENV['DOORKEEPER_APP_NAME']` で表示用の名前を設定できるようにしています。

## `config/routes.rb` に設定

`config/routes.rb` で `omniauth_callbacks` として独自のものを使うように設定します。

今回は認証必須なので不要ですが、
例として `sign_in` と `sign_out` の URL も入れました。
実際に試してみるとすぐに自動ログインで再ログインしてしまいます。

`sign_out` が `get` か `delete` か違うことがあるので、
`sign_out_via` を使ってどちらでも対応できるようにしました。

```ruby config/routes.rb
  devise_for :users, controllers: { omniauth_callbacks: 'users/omniauth_callbacks' }
  devise_scope :user do
    get 'sign_in',  to: 'devise/sessions#new',     as: :new_user_session
    __send__ Devise.sign_out_via, 'sign_out', to: 'devise/sessions#destroy', as: :destroy_user_session
  end
```

## timeoutable 設定


SSO 的に使うのは
OAuth 2.0 の本来の目的ではないので、
ログアウトは難しい問題です。
たとえば
doorkeeper
と連携するアプリが複数あるときにまとめてログアウト出来ないなどの問題があります。

そのため、このアプリでは一定時間で再ログインが必要になるように `timeoutable` を使って、
こまめに認証し直すようにしてログアウト問題を緩和しています。

その副作用として入力に時間のかかるフォームがあると入力途中でタイムアウトしてしまって
投稿に失敗するなどの問題も起きるので、その点を考慮しておく必要があります。

```ruby config/initializers/devise.rb
  # Default is 30 minutes.
  config.timeout_in = 1.minutes if Rails.env.development?
  config.expire_auth_token_on_timeout = true
```

## app/models/user.rb に実装

`find_or_create_for_doorkeeper_oauth` の実装は `concerns` に分けてみました。
`omniauthable` に `omniauth_providers` も設定して余計な route が生成されないようにしています。
`timeoutable` も入れています。

```ruby app/models/user.rb
  devise :omniauthable, omniauth_providers: [:doorkeeper]
  devise :timeoutable
  include DoorkeeperOauthFinder
```

ログインしたときに `name` や `email` が変わっていたら反映するようにしています。

`id` を統一したいのなら、 `create` のときに `id` まで指定すると
doorkeeper gem による OAuth provider 側とユーザーの ID を統一できます。

SSO 的に使うのならパスワードは不要なので、
ここではコメントアウトしています。

```ruby app/models/concerns/doorkeeper_oauth_finder.rb
module DoorkeeperOauthFinder
  extend ActiveSupport::Concern

  module ClassMethods
    def find_or_create_for_doorkeeper_oauth(oauth_data)
      uid = oauth_data.uid.to_s
      id = uid.to_i
      user = self.where(provider: oauth_data.provider, uid: uid).first
      if user
        user.name = oauth_data.info.name
        user.email = oauth_data.info.email
        user.save! if user.changed?
      else
        user = self.create!({
          id: id, # use same id
          name: oauth_data.info.name,
          provider: oauth_data.provider,
          uid: uid,
          email: oauth_data.info.email,
          #password: Devise.friendly_token[0,20]
        })
      end
      user
    end
  end
end
```

データベースの migration の方でも削除して unique index の制約なども不要なものは外しておきます。

```ruby db/migrate/*_devise_create_users.rb
      ## Database authenticatable
      t.string :email,              null: false, default: ""
      # t.string :encrypted_password, null: false, default: ""
```

代わりに `provider` と `uid` と `name` を追加しました。
このアプリは Doorkeeper 専用なので、直接 `users` に追加していますが、
複数プロバイダに対応するには `provider` と `uid` の組を別テーブルにします。

```ruby db/migrate/*_add_omniauth_columns_to_users.rb
class AddOmniauthColumnsToUsers < ActiveRecord::Migration
  def change
    add_column :users, :provider, :string
    add_column :users, :uid, :string
    add_column :users, :name, :string
    add_index :users, [:provider, :uid], unique: true
  end
end
```

## `OAuth2::AccessToken` を生成

`OAuth2::Client` と保存しておいた `access_token` を引数にして `OAuth2::AccessToken` を生成します。
ここでは `concerns` に分けて必要なコントローラーでだけ `include DoorkeeperApiV1` するようにしました。
全体で使いたいのなら `ApplicationController` に `include` すれば良いと思います。

```ruby app/controllers/concerns/doorkeeper_api_v1.rb
module DoorkeeperApiV1
  private

  def access_token
    return @access_token if defined?(@access_token)
    config = Devise.omniauth_configs[:doorkeeper]
    strategy = config.strategy_class.new(*config.args)
    token = session[:doorkeeper_token]
    @access_token = OAuth2::AccessToken.new(strategy.client, token)
  end

  def get_me
    access_token.get("/api/v1/me.json").parsed
  end

  def get_microposts
    access_token.get("/api/v1/microposts.json").parsed
  end

  MICROPOST_CONTENT_MAX_LENGTH = 140

  def post_micropost(micropost)
    micropost[:content] = micropost[:content].truncate(MICROPOST_CONTENT_MAX_LENGTH)
    access_token.post("/api/v1/microposts", params: { micropost: micropost }).parsed
  end
end
```

使い方は `get_me` などを呼び出すだけなので省略します。

## ログインを強制する

常に Doorkeeper の方でログインさせておきたいアプリの場合は、
User クラスを使っている場合の devise での戻り先の `session[:user_return_to]` に URL を保存しておいて、
`user_omniauth_authorize_path(:doorkeeper)` に強制的にリダイレクトしています。
`main_app.` をつけているのは route で mount している engine の中で問題が起きたことがあったためです。

```ruby app/controllers/application_controller.rb
  include AuthDoorkeeper
  before_action :auto_authenticate_omniauth_user!
```

```ruby app/controllers/concerns/auth_doorkeeper.rb
module AuthDoorkeeper
  private

  def auto_authenticate_omniauth_user!
    return if current_user
    session[:user_return_to] = request.original_url
    redirect_to main_app.user_omniauth_authorize_path(:doorkeeper)
  end
end
```

## テストについて

API 呼び出しの部分の対処が出来ていなくて、まだテストが通る状態には出来ていません。
