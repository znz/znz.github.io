---
layout: post
title: "bundle updateした記録"
date: 2014-02-03 17:00:00 +0900
comments: true
categories: ruby rails gem
---
`bundle update` で更新された gem などのメモを残すことにしました。

<!--more-->

## jquery-rails 3.1.0 (was 3.0.4)

jQuery 1.11.0 にあがったのと jquery-ujs が更新されていました。

```text CHANGELOG.md
## 3.1.0 (29 January 2014)

  - Updated to jQuery 1.11.0
  - Updated to latest jquery-ujs
  - Added development rake task for updating jQuery
```

## omniauth-oauth2 1.1.2 (was 1.1.1)

oauth2 が 0.8.1 から 0.9.3 に一気にあがって驚きましたが、
omniauth-oauth2 の依存関係が更新されたのが原因でした。

他にもたくさん変更されていました。

## turbolinks 2.2.1 (was 2.2.0)

https://github.com/rails/turbolinks/blob/master/CHANGELOG.md#turbolinks-221-january-30-2014
によるとバグ修正のみのようです。

## rails_admin 0.6.1 (was 0.6.0)

paper_trail gem が 3.0.0 で `Version` から `PaperTrail::Version` に変わった
[#165](https://github.com/airblade/paper_trail/pull/165)
影響を受けて、
`config/initializers/rails_admin.rb`
を以下のように変更する必要がありました。

```diff config/initializers/rails_admin.rb.diff
--- a/config/initializers/rails_admin.rb
+++ b/config/initializers/rails_admin.rb
@@ -18,7 +18,7 @@ RailsAdmin.config do |config|
   # config.audit_with :history, 'User'

   # Or with a PaperTrail: (you need to install it first)
-  config.audit_with :paper_trail, 'User'
+  config.audit_with :paper_trail, 'User', 'PaperTrail::Version'

   # Display empty fields in show views:
   # config.compact_show_view = false
```

あまりカスタマイズしていないのなら、
`rails g rails_admin:install`
で生成し直して、設定し直した方が良いかもしれません。

最初に rails_admin を更新した rails アプリでは以下のように
devise と pundit と paper_trail の設定をしただけで、
actions などはそのままにしています。

```ruby config/initializers/rails_admin.rb
RailsAdmin.config do |config|

  ### Popular gems integration

  ## == Devise ==
  config.authenticate_with do
    warden.authenticate! scope: :user
  end
  config.current_user_method(&:current_user)

  ## == Cancan ==
  # config.authorize_with :cancan

  require 'rails_admin/extensions/pundit'
  config.authorize_with :pundit

  ## == PaperTrail ==
  config.audit_with :paper_trail, 'User', 'PaperTrail::Version' # PaperTrail >= 3.0.0

  ### More at https://github.com/sferik/rails_admin/wiki/Base-configuration

  config.actions do
    dashboard                     # mandatory
    index                         # mandatory
    new
    export
    bulk_delete
    show
    edit
    delete
    show_in_app

    ## With an audit adapter, you can add:
    # history_index
    # history_show
  end
end
```

