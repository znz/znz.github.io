---
layout: post
title: "devise で電子メールアドレスのドメイン部分を省略してもログインできるようにする"
date: 2016-04-13 21:29:43 +0900
comments: true
categories: ruby rails devise
---
社内向けアプリケーションのように、特定のドメインのユーザーがほとんどの場合、メールアドレスの全体を入力させるのは、余計な手間をしいていることが多いです。

そこで省略可能にするように変更しました。

<!--more-->

## 対象バージョン

- ruby 2.2.4
- rails 4.2.6
- devise 3.5.6
- warden 1.2.6

## config/initializers/devise.rb での設定

直接は関係ないですが、 `config/initializers/devise.rb` では以下のような感じの設定でユーザー登録できるメールアドレスのドメインを制限しています。

特殊用途に別ドメインのユーザーを登録する必要があったので、そこは `|` (or) で繋げて許可しています。

```ruby config/initializers/devise.rb
  config.email_regexp = /\A[\w+\-.]+@example\.co\.jp\z\|\Aspecial@example\.com\z/i
```

## User クラスへの追加

[How To: Allow users to sign in using their username or email address](https://github.com/plataformatec/devise/wiki/How-To:-Allow-users-to-sign-in-using-their-username-or-email-address) を参考にして `User.find_first_by_auth_conditions(warden_conditions)` を定義すれば良いということがわかったので、以下のように定義しました。

```ruby app/models/user.rb
  def self.find_for_database_authentication(warden_conditions)
    if /@/ =~ warden_conditions[:email]
      super
    else
      super(warden_conditions.merge(email: "#{warden_conditions[:email]}@example.co.jp"))
    end
  end
```

メールアドレス全体が入力された時 (`@` を含む時) はデフォルトの挙動をそのまま使い、省略された時はデフォルトのドメイン (例では `example.co.jp`) を補ってデフォルトの挙動を呼び出すようにしました。
