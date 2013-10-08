---
layout: post
title: "doorkeeper gem の API のクライアント"
date: 2013-10-08 21:06
comments: true
categories: ruby rails doorkeeper devise omniauth oauth2
---
[doorkeeper gem](http://rubygems.org/gems/doorkeeper)
で API を作る方は
[doorkeeper-provider-app](https://github.com/applicake/doorkeeper-provider-app)
というサンプルの
`app/controllers/api/`
以下などをみればすぐにわかったのですが、
API を呼び出す方は
[OmniAuth の中でユーザーの情報を取り出す](https://github.com/applicake/doorkeeper/wiki/Create-a-OmniAuth-strategy-for-your-provider)
だけならすぐに出来たのですが、
コントローラーの中など呼び出す方法は
[doorkeeper-devise-client](https://github.com/applicake/doorkeeper-devise-client)
を見てもよくわからなかったので、まとめてみました。

<!--more-->

## 動作確認バージョン

* provider 側
  * rails 3.2.14
  * doorkeeper 0.7.3
* client 側
  * rails 4.0.0
  * devise 3.1.1
  * omniauth 1.1.4
  * omniauth-oauth2 1.1.1
  * oauth2 0.8.1

## 準備

まず
[Create a OmniAuth strategy for your provider](https://github.com/applicake/doorkeeper/wiki/Create-a-OmniAuth-strategy-for-your-provider)
を参考にして、
OmniAuth の中で
`access_token.get('/api/v1/me.json').parsed`
は出来るところまでは準備しておきます。

目的としては、
この
`access_token`
が認証より後で呼ばれる他のコントローラーの中で取得できれば良いということになります。

余談ですが、
doorkeeper の wiki は
[Supported Ruby & Rails versions](https://github.com/applicake/doorkeeper/wiki/Supported-Ruby-&-Rails-versions)
のように情報が古いまま放置されているページもあるようなので、
[README](https://github.com/applicake/doorkeeper)
などのソースコード側のドキュメントも参照した方が良さそうです。

## 必要なもの

`access_token`
は
`OAuth2::AccessToken`
クラスのオブジェクトです。

生成するには

* `OAuth2::Client` のオブジェクト
* 認証で取得した `token`

が必要になります。

`OAuth2::Client`
の生成には

* `client_id`
* `client_secret`
* URL

が必要になります。

## token の保存

まず
`OAuth2`
の認証で取得した
`token`
を保存しておく必要があります。

`Users::OmniauthCallbacksController#doorkeeper`
で
`session[:doorkeeper_token] = request.env["omniauth.auth"]["credentials"]["token"]`
のようにしてセッションなどの後で使える場所に保存しておきます。

後で調べてわかったのですが、
[doorkeeper-devise-client の Users::OmniauthCallbacksController](https://github.com/applicake/doorkeeper-devise-client/blob/master/app/controllers/users/omniauth_callbacks_controller.rb)
では
`request.env["omniauth.auth"].credentials.token`
を
`user.doorkeeper_access_token`
でデータベースに保存していました。

## OAuth2::Client の作成

[doorkeeper-devise-client の ApplicationController](https://github.com/applicake/doorkeeper-devise-client/blob/master/app/controllers/application_controller.rb)
では必要な情報は定数経由で受け取るようになっていました。

今回は
`devise`
と
`omniauth-oauth2`
を使っているので、
その情報を使って生成するようにしました。
要点だけまとめると以下のコードになります。

```ruby
    config = Devise.omniauth_configs[:doorkeeper]
    strategy = config.strategy_class.new(*config.args)
    client = strategy.client
```

## OAuth2::AccessToken の生成

ここまで準備ができれば後は
`OAuth2::AccessToken.new`
するだけです。

まとめると以下のコードになります。

```ruby
  def access_token
    return @access_token if defined?(@access_token)
    config = Devise.omniauth_configs[:doorkeeper]
    strategy = config.strategy_class.new(*config.args)
    token = session[:doorkeeper_token]
    @access_token = OAuth2::AccessToken.new(strategy.client, token)
  end
```

## API 呼び出し

`access_token`
が出来たら後は呼び出しに使うだけです。

単純な情報取得は
`get`
して
JSON
なら
`parsed`
を呼び出すだけです。

```ruby
  access_token.get("/api/v1/me.json").parsed
  access_token.get("/api/v1/posts.json").parsed
```

今回は連携して書き込みたいというのが目的だったため、
`post`
も使いました。

モデルの例としては
`rails g scaffold post title body:text`
で API の提供側では以下の実装とします。

```ruby app/controllers/api/v1/posts_controller.rb
module Api::V1
  class PostsController < ApiController
    doorkeeper_for :index
    doorkeeper_for :create
    respond_to     :json

    def index
      respond_with Post.all
    end

    def create
      post = Post.new(params[:post])
      post.user = current_resource_owner
      post.save!
      respond_with post
    end
```

呼び出し側は以下のようになります。

```ruby
access_token.post("/api/v1/posts", params: { post: { title: title, body: body } })
```

`params`
による指定は
[OAuth2::AccessToken](https://github.com/intridea/oauth2/blob/master/lib/oauth2/access_token.rb)
のソースをみて推測しました。

## scope 付き API 提供

書き込みも許可すると
`scope`
を分けたくなります。

doorkeeper 側では
[Using Scopes](https://github.com/applicake/doorkeeper/wiki/Using-Scopes)
を参考にして

* initializers に scopes 追加
* 翻訳追加
* API に scopes 追加

をしておきます。

```ruby config/initializers/doorkeeper.rb
  default_scopes  :public
  optional_scopes :admin, :write
```

```ruby app/controllers/api/v1/posts_controller.rb
    doorkeeper_for :index,  :show,   scopes: [:public]
    doorkeeper_for :create, :update, scopes: [:admin, :write]
```

参考のため、この API の rspec も載せておきます。
複数の `scopes` を設定する時に `,` 区切りだとうまくいかないところがあったので、
スペース区切りにしています。

```ruby spec/controllers/api/v1/posts_controller_spec.rb
require 'spec_helper'

describe Api::V1::PostsController do
  describe "GET 'index'" do
    let!(:application) { Doorkeeper::Application.create!(name: "MyApp", redirect_uri: "http://app.com") }
    let!(:user) { FactoryGirl.create(:normal_user) }
    let!(:token) { Doorkeeper::AccessToken.create! application_id: application.id, resource_owner_id: user.id, scopes: "public" }
    subject { response }

    context "valid token" do
      before do
        get 'index', format: :json, access_token: token.token
      end
      it { should be_success }
      its(:status) { should eq(200) }
      its(:body) { should == Post.all.to_a.to_json }
    end

    context "invalid token" do
      before do
        get 'index', format: :json, access_token: token.token.succ
      end
      it { should_not be_success }
      its(:status) { should eq(401) }
    end
  end

  describe "GET 'index' without scopes" do
    let!(:application) { Doorkeeper::Application.create!(name: "MyApp", redirect_uri: "http://app.com") }
    let!(:user) { FactoryGirl.create(:normal_user) }
    let!(:token) { Doorkeeper::AccessToken.create! application_id: application.id, resource_owner_id: user.id, scopes: "write" }
    subject { response }

    context "valid token" do
      before do
        get 'index', format: :json, access_token: token.token
      end
      it { should_not be_success }
      its(:status) { should eq(401) }
      its(:body) { should == " " }
    end
  end

  describe "POST 'create'" do
    let!(:application) { Doorkeeper::Application.create!(name: "MyApp", redirect_uri: "http://app.com") }
    let!(:user) { FactoryGirl.create(:normal_user) }
    let!(:token) { Doorkeeper::AccessToken.create! application_id: application.id, resource_owner_id: user.id, scopes: "public write" }
    subject { response }

    context "valid token" do
      before do
        post 'create', format: :json, access_token: token.token, post: { title: "title", body: "some content" }
      end
      it { should be_success }
      its(:status) { should eq(201) } # 201 Created
      its(:body) { should == Post.last.to_json }
    end

    context "invalid token" do
      before do
        post 'create', format: :json, access_token: token.token.succ
      end
      it { should_not be_success }
      its(:status) { should eq(401) }
    end
  end
end
```

## scope 付き呼び出し

`devise`
の設定で
`omniauth`
の設定に
`scope`
を追加するだけです。

```ruby config/initializers/devise.rb
  config.omniauth :doorkeeper, ENV['DOORKEEPER_APP_ID'], ENV['DOORKEEPER_APP_SECRET'], { scope: 'public write' }
```

rspec のところでもちょっと書きましたが、
区切りが `,` だとうまくいかないことがあったので、
スペース区切りにしています。

原因は
[lib/doorkeeper/oauth/scopes.rb](https://github.com/applicake/doorkeeper/blob/master/lib/doorkeeper/oauth/scopes.rb)
で
`string.split`
のように無引数の
`String#split`
を使っているからではないかと推測していますが、確認はしていません。

## まとめ

`client_id` と `client_secret` と provider の URL はあらかじめ用意しておいて、
client 側の rails アプリに設定しておきます。

`token`
は OAuth2 で取得したものを
`session`
やデータベースなどに保存しておいて使います。

必要なら
`scopes`
も設定できます。
