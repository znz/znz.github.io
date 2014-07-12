---
layout: post
title: "devise で通常ログイン出来るユーザーが他の複数サービス連携でログインできるようにした"
date: 2014-07-07 23:30:00
comments: true
categories: ruby rails devise omniauth oauth2
---
`devise` でデータベース認証を実装している Rails アプリに Twitter などでもログインできるように
実装を追加してみました。

<!--more-->

## 対象バージョン

- ruby 2.1.2
- rails 4.1.4
- devise 3.2.4
- omniauth 1.2.1
- omniauth-facebook 1.6.0
- omniauth-github 1.1.2
- omniauth-google-oauth2 0.2.4
- omniauth-twitter 1.0.1
- (dotenv 0.11.1) (key と secret の管理)
- (font-awesome-sass 4.1.0) (view での `icon` メソッド)

## 前提条件

- devise で通常の User モデルにメールアドレスとパスワードによる認証は実装されている。
- 複数サービスとの連携をするために User モデルには連携サービスの情報は直接持たずに UserAuth モデルに分ける。
- 連携対象サービスは Facebook, GitHub, Google+, Twitter
- 連携サービスからの追加情報はメールアドレスなども含めて一切とらずに認証だけに利用する。
- どの連携サービス (provider) かと識別するための ID だけ保存する。
- key や secret は環境変数で渡す。 (今回は dotenv を使ったが、 config/secrets.yml 経由でも良さそう)
- 今回はテストは未実装 (連携部分のテストの書き方をまだ調べていないため)
- 今回の実装範囲では I18n は使わずにメッセージは日本語固定

## 実装

今回は以下のように実装を追加すると連携サービスでログインできるようになりました。

### Gemfile

OmniAuth と使用するサービスを追加します。

```ruby Gemfile
    gem 'omniauth'
    gem 'omniauth-facebook'
    gem 'omniauth-github'
    gem 'omniauth-google-oauth2'
    gem 'omniauth-twitter'
```

### initializer 追加

key と secret があれば provider 登録するようにしました。

view で使うために、
登録されている provider の情報の取り方がわからなかったのと
追加で名前や Font Awesome のアイコン名も入れたかったので、
`AUTH_PROVIDERS` という配列にハッシュを入れるようにしています。

```ruby config/initializers/omniauth.rb
    AUTH_PROVIDERS = []
    
    Rails.application.config.middleware.use OmniAuth::Builder do
      key, secret = ENV['FACEBOOK_KEY'], ENV['FACEBOOK_SECRET']
      if key && secret
        provider :facebook, key, secret
        AUTH_PROVIDERS << {
          provider: :facebook,
          name: 'Facebook',
        }
      end
    
      key, secret = ENV['GITHUB_CONSUMER_KEY'], ENV['GITHUB_CONSUMER_SECRET']
      if key && secret
        provider :github, key, secret
        AUTH_PROVIDERS << {
          provider: :github,
          name: 'GitHub',
        }
      end
    
      key, secret = ENV['GOOGLE_CLIENT_ID'], ENV['GOOGLE_CLIENT_SECRET']
      if key && secret
        provider :google_oauth2, key, secret
        AUTH_PROVIDERS << {
          provider: :google_oauth2,
          name: 'Google',
          icon: :google
        }
      end
    
      key, secret = ENV['TWITTER_CONSUMER_KEY'], ENV['TWITTER_CONSUMER_SECRET']
      if key && secret
        provider :twitter, key, secret
        AUTH_PROVIDERS << {
          provider: :twitter,
          name: 'Twitter',
        }
      end
    
      AUTH_PROVIDERS.each do |provider|
        provider[:icon] ||= provider[:provider]
      end
    end
```

### UserAuth モデル作成

`rails g model UserAuth user:references uid:string provider:string` で作成します。

モデルクラスは生成されたまま使いましたが、
validation を追加した方が良さそうです。

```ruby app/models/user_auth.rb
    class UserAuth < ActiveRecord::Base
      belongs_to :user
    end
```

データベースの方は `null: false` や index を追加しました。

```ruby db/migrate/*_create_user_auths.rb
    class CreateUserAuths < ActiveRecord::Migration
      def change
        create_table :user_auths do |t|
          t.references :user, index: true, null: false
          t.string :uid, null: false
          t.string :provider, null: false
    
          t.timestamps
        end
    
        add_index :user_auths, [:uid, :provider], unique: true
      end
    end
```

### `rake db:migrate`

`rake db:migrate` で反映しておきます。

### User クラス側

`has_many` を追加しておきます。

```ruby app/models/user.rb
      has_many :user_auths, dependent: :destroy
```

### ルーティング追加

まず、どう使われるのか把握するためにルーティングを追加します。

`auth_help` は後でプライバシーポリシーなどを書いています。

```ruby config/routes.rb
      get '/auth/help' => 'auth#help', as: :auth_help
      get '/auth/:provider/callback' => 'auth#create'
      delete '/auth/destroy/:provider' => 'auth#destroy', as: :destroy_connection
```

ここには出てきていませんが、
OmniAuth で `/auth/:provider` がルーティングされるようです。

### AuthController 実装

参考サイトの実装例を参考にして AuthController を実装しました。

help は静的な情報を表示するだけなので、実装は空です。

```ruby
    # -*- coding: utf-8 -*-
    class AuthController < ApplicationController
      skip_before_filter :authenticate_user!
    
      def create
        auth = request.env['omniauth.auth']
        uid = auth['uid']
        provider = auth['provider']
        auth = UserAuth.where(uid: uid, provider: provider).first
        if auth
          flash[:notice] = "#{provider}でログインしました。"
          sign_in_and_redirect auth.user, event: :authentication
        else
          authenticate_user!
          UserAuth.create!(uid: uid, provider: provider, user_id: current_user.id)
          redirect_to root_url, notice: "#{provider}と連携しました。"
        end
      end
    
      def destroy
        provider = params.require(:provider)
        authenticate_user!
        auth = UserAuth.where(provider: provider, user_id: current_user.id).first
        auth.destroy if auth
        redirect_to root_url, notice: "#{provider}と連携解除しました。"
      end
    
      def help
      end
    end
```

まだ連携を登録していない状態で

- ログアウト状態
- 連携サービスで認証・連携許可
- 戻ってきて通常ログイン
- `auth/failure` に飛ばされる

ということがおきているのですが、
`auth/failure`
を実装していないので、デフォルトの 404 エラー画面になってしまいます。

### view helpers 実装

以前から使っていた `link_to_sign_in_or_out` の実装も同じファイルに持ってきて、
`link_to_provider` という汎用的なメソッドを実装しました。
使い方は後述します。

`AUTH_PROVIDERS` のハッシュはここで使っています。

```ruby app/helpers/link_to_auth_helper.rb
    # -*- coding: utf-8 -*-
    module LinkToAuthHelper
      def link_to_sign_in_or_out(html_options={})
        if user_signed_in?
          body = icon(:'sign-out') + t(:"devise.shared.links.sign_out", default: "Sign out")
          html_options = { method: :delete }.merge(html_options)
          link_to body, :destroy_user_session, html_options
        else
          body = icon(:'sign-in') + t(:"devise.shared.links.sign_in", default: "Sign in")
          link_to body, :new_user_session, html_options
        end
      end
    
      def link_to_provider(provider, html_options={})
        if current_user
          if UserAuth.where(user_id: current_user.id, provider: provider[:provider]).exists?
            html_options = { method: :delete }.merge(html_options)
            link_to icon(provider[:icon])+"#{provider[:name]}との連携を解除", destroy_connection_path(provider: provider[:provider]), html_options
          else
            link_to icon(provider[:icon])+"#{provider[:name]}と連携", "/auth/#{provider[:provider]}", html_options
          end
        else
          link_to icon(provider[:icon])+"#{provider[:name]}でログイン", "/auth/#{provider[:provider]}", html_options
        end
      end
    end
```


### view に追加

ナビゲーションに以下のように追加しました。
認証連携がないときは従来のログイン・ログアウトだけ表示しています。

`AUTH_PROVIDERS` に登録されているときだけ連携のリンクを表示するようにしています。

```text app/views/layouts/_navigation.html.slim
        li.dropdown
          a.dropdown-toggle data-toggle="dropdown"
            = icon(:user) + 'ログイン管理'
            b.caret
          ul.dropdown-menu
            li= link_to_sign_in_or_out
            - unless AUTH_PROVIDERS.empty?
              li= link_to icon(:info)+"認証連携のヘルプ", auth_help_path
            - AUTH_PROVIDERS.each do |provider|
              li= link_to_provider(provider)
```

### twitter 連携

Twitter は複数アカウントを使うことも通常の使用範囲として想定されていて、
アプリ専用のアカウントもとりやすいので、
Twitter 連携から試してみました。

https://apps.twitter.com/ から `Create New App` で作成します。

- Name: Twitter 全体で一意になるアプリケーションの ID 的にも使われるもの。ツイートの時にツイートしたアプリ名としても埋め込まれるが、今回は認証のみなので、他とぶつからないような名前という以上はこだわらなかった。
- Description: 認証のときに出てくる説明。
- Website: たとえば `http://app.127.0.0.1.xip.io:3000/` など
- Callback URL: たとえば `http://app.127.0.0.1.xip.io:3000/auth/twitter/callback` のように `/auth/twitter/callback` にする。実サイトなら `https` にすべき。

作成後には `Allow this application to be used to Sign in with Twitter` のチェックを入れておきます。
チェックがないと Twitter 連携でのログインがうまくいかないようです。

アイコンや Organization なども必要に応じて変更します。

API keys タブで key と secret を取得します。

- API key : 環境変数 `TWITTER_CONSUMER_KEY` に設定
- API secret : 環境変数 `TWITTER_CONSUMER_SECRET` に設定

dotenv を使っているので `.env` に以下のような感じで設定しました。
ランダムな文字列のように見えます。

```text
    TWITTER_CONSUMER_KEY="xxxxXXxxXxXxXXXxXxXXXXxxx"
    TWITTER_CONSUMER_SECRET="xxXXxXXxxxxxxXXXXXxXxXXxxXXXxXXxxxxXXxxXxxXXxxXxxx"
```

### 動作確認

- 通常のログインをした状態で「Twitter と連携」で Twitter の許可画面に飛びます。
- 許可すると「Twitter 側に許可情報」と「UserAuth に連携情報」が保存されます。
- 「Twitter との連携を解除」で「UserAuth が削除」されます。
- 「ログアウト」して「Twitter でログイン」すると「ログイン画面」に戻ります。
- ログインすると `auth/failure` が 404 エラーになります。ここは後で実装する予定です。
- 再度ログイン後の画面を直接開きます。
- 再度「Twitter と連携」で「UserAuth に連携情報」が保存されます。「Twitter 側の許可情報」は古いままです。
- 「ログアウト」して「Twitter でログイン」でログインできます。
- https://twitter.com/settings/applications で許可を取り消します。
- 「ログアウト」して「Twitter でログイン」で Twitter の許可画面に飛びます。
- 許可すると「Twitter 側に許可情報」が保存されて、「UserAuth」は残っているので、そのままログインできます。

### GitHub 連携

GitHub 連携は作成後にすぐに使えるので、アプリケーション作成に使っても良いアカウントがあれば一番簡単です。

https://github.com/settings/applications から `Register new application` で作成します。

- Application name: 適切な名前を設定
- Homepage URL: たとえば `http://app.127.0.0.1.xip.io:3000/` など
- Application description: ユーザーが許可するときにわかりやすい説明
- Authorization callback URL: たとえば `http://app.127.0.0.1.xip.io:3000/auth/github/callback` のように `/auth/github/callback` にする。実サイトなら `https` にすべき。

右上に見えている Client ID と Client Secret を使います。

- Client ID : 環境変数 `GITHUB_CONSUMER_KEY` に設定
- Client Secret : 環境変数 `GITHUB_CONSUMER_SECRET` に設定

dotenv を使っているので `.env` に以下のような感じで設定しました。
十六進数の数字のように見えます。

```text
    GITHUB_CONSUMER_KEY="xxxxxxxxxxxxxxxxxxxx"
    GITHUB_CONSUMER_SECRET="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

## Facebook 連携

https://developers.facebook.com/ から上の `Apps` の中にある `Craete a New App` で作成します。
最初は作成前に開発者関連の規約などに同意する必要があるようです。

- Display Name: 適切な名前を設定します。
- Namespace: optional なので空欄のままで良いようです。
- カテゴリ: 適当に選択します。

セキュリティチェックがあるので、入力すると作成できます。

右上に見えている App ID と App Secret (Show を押すとパスワード認証の後に内容表示) を使います。

- App ID : 環境変数 `FACEBOOK_KEY` に設定
- App Secret : 環境変数 `FACEBOOK_SECRET` に設定

dotenv を使っているので `.env` に以下のような感じで設定しました。
十進数と十六進数の数字のように見えます。

```text
    FACEBOOK_KEY="xxxxxxxxxxxxxxx"
    FACEBOOK_SECRET="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

`Settings` の `Add Platform` で `Website` を選んで追加します。

- Site URL: たとえば `http://app.127.0.0.1.xip.io:3000/` など
- Mobile Site URL: 空欄のまま

callback URL は設定が不要だったので、
Site URL の下ならどこでも良さそうです。

### `google_oauth2` 連携

https://github.com/zquestz/omniauth-google-oauth2 に書いてあるように
https://code.google.com/apis/console/ を開きます。

飛ばされる先で `API & AUTH` の中の `Credentials` で `Create new Client ID` で作成します。

- Application type : Web application
- Authorized JavaScript origins : たとえば `http://app.127.0.0.1.xip.io:3000` など
- Authorized redirect URI : たとえば `http://app.127.0.0.1.xip.io:3000/auth/google_oauth2/callback` のように `/auth/google_oauth2/callback` にする。実サイトなら `https` にすべき。

右に見えている Client ID と Client secret を使います。

- Client ID : 環境変数 `GOOGLE_CLIENT_ID` に設定
- Client secret : 環境変数 `GOOGLE_CLIENT_SECRET` に設定

dotenv を使っているので `.env` に以下のような感じで設定しました。
ドメインっぽい文字列とランダムな文字列のように見えます。

```text
    GOOGLE_CLIENT_ID="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.apps.googleusercontent.com"
    GOOGLE_CLIENT_SECRET="XXxxxXxXxxxXXxXXxXxXXxxX"
```


さらに `Consent screen` での設定が必要です。
しかも全アプリ共通のようなので、名前を分けたい場合はグーグルアカウントごと違うものにしないといけないように見えました。

- Email address : グーグルアカウントのアドレス選択
- Product name : 適切な名前を設定
- Homepage URL 以下 : 必要に応じて設定

さらにエラーメッセージ (`"Access Not Configured. Please use Google Developers Console to activate the API for your project."`) で検索してわかったのですが、
[Google OAuth 2.0 loginが2014年9月に使えなくなる(Googleアカウントからのユーザ登録機能がある場合は注意)](http://qiita.com/aikyo02/items/459d03af304e1188f110 "Google OAuth 2.0 loginが2014年9月に使えなくなる(Googleアカウントからのユーザ登録機能がある場合は注意)")
に書いてあるように、
APIs で `Google+ API` も有効にする必要がありました。

## 参考サイト

- [Railsでログインとは別に複数のサービスとの連携を行う方法 - PILOG](http://xoyip.hatenablog.com/entry/2013/12/20/212109 "Railsでログインとは別に複数のサービスとの連携を行う方法 - PILOG")
  と[その実装](https://github.com/xoyip/multi-oauth)は非常に参考になりました。
- [Google OAuth 2.0 loginが2014年9月に使えなくなる(Googleアカウントからのユーザ登録機能がある場合は注意)](http://qiita.com/aikyo02/items/459d03af304e1188f110 "Google OAuth 2.0 loginが2014年9月に使えなくなる(Googleアカウントからのユーザ登録機能がある場合は注意)")
