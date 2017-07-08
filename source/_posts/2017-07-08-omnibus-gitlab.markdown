---
layout: post
title: "Omnibus GitLabをContainer RegistryやMattermostを有効にして使う"
date: 2017-07-08 16:30:00 +0900
comments: true
categories: gitlab linux ubuntu
---
GitLab と Dokku (と一部 Heroku) を使って CI/CD (Continuous Integration / Continuous Deployment) 環境を作ってみたので、何回かにわけてその話を書いていきます。
最初は Omnibus GitLab 自体の設定の話です。
certbot で https を有効にして、 GitLab Container Registry や GitLab Mattermost も有効にします。
GitLab Pages も有効にしますが、 certbot での自動化はできなかったので http のみの設定です。

[gitlab カテゴリー](/blog/categories/gitlab/)で一覧が見えます。

<!--more-->

## 対象バージョン

- Ubuntu 16.04.2 LTS
- Omnibus GitLab 9.1.4-ce.0 (インストール時) から 9.3.5-ce.0 (記事執筆時)
- gitlab-ci-multi-runner 9.2.0 (インストール時) から 9.3.0 (記事執筆時)
- Dokku 0.9.4 (インストール時) から 0.10.2 (記事執筆時)

## インストール方法選択

[Installation methods for GitLab](https://about.gitlab.com/installation/) にある方法のうち、Omnibus package installation (recommended) を選択しました。
理由としては以下のようなことを考慮しました。

- ソースインストールはインストールもバージョンアップも管理も大変そうなので除外
- Docker でのインストールはバージョンを上げて問題がおきた場合に戻しやすそうだが、バージョンアップ情報を自分で追いかけないといけない
- Omnibus package は apt で他のパッケージの更新と同様にバージョンアップが検知可能
- GitLab 自体は仮想マシン1台で動かすのでバージョンアップ時に停止時間が発生するのは許容

## カスタマイズ方法

Docker イメージは `GITLAB_OMNIBUS_CONFIG` で、 Omnibus package は `/etc/gitlab/gitlab.rb` でカスタマイズできます。
(設定できる項目は https://docs.gitlab.com/omnibus/docker/README.html からリンクされている [Omnibus GitLab template](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/files/gitlab-config-template/gitlab.rb.template) 参照)

自動構築したい場合は `gitlab_rails['initial_root_password']` や `gitlab_rails['initial_shared_runners_registration_token']` が設定できるというのを知っておくと良さそうです。

設定項目によっては Web UI で変更しても `sudo gitlab-ctl reconfigure` を実行すると再設定されてしまうものもあるようなので、影響しそうな設定を変更するときは気にしておくと良さそうです。
(気づいた範囲では GitLab Mattermost 関係の設定が再設定されました。)

## 最低限の設定

相対時間で表示されることも多いので、デフォルトのままでもあまり気にならないかもしれませんが、

    gitlab_rails['time_zone'] = 'Asia/Tokyo'

でタイムゾーンの設定をしておくと良いと思います。

## メール設定

[SMTP settings](https://docs.gitlab.com/omnibus/settings/smtp.html) を参考にしてメール送信の設定をしておきます。

以下は SMTP over TLS (smtps) で送信する例です。

    # https://docs.gitlab.com/omnibus/settings/smtp.html
    gitlab_rails['smtp_enable'] = true
    gitlab_rails['smtp_address'] = 'smtp.example.com'
    gitlab_rails['smtp_port'] = 465
    gitlab_rails['smtp_user_name'] = 'username'
    gitlab_rails['smtp_password'] = 'password'
    gitlab_rails['smtp_tls'] = true

Mattermost も使うなら一緒に設定しておきます。

    mattermost['email_smtp_username'] = 'username'
    mattermost['email_smtp_password'] = 'password'
    mattermost['email_smtp_server'] = 'smtp.example.com'
    mattermost['email_smtp_port'] = 465
    mattermost['email_connection_security'] = 'TLS'
    mattermost['email_feedback_name'] = 'GitLab Mattermost'
    mattermost['email_feedback_email'] = 'email@example.com'
    mattermost['support_email'] =  'support@example.com'

## 後で使う変数設定

config は ruby のコードなので、ローカル変数でドメインをまとめて設定できるようにしておきます。

    base_domain = 'example.test'
    gitlab_domain = "gitlab.#{base_domain}"
    registry_domain = "registry.#{base_domain}"
    mattermost_domain = "mattermost.#{base_domain}"

## https 設定

GitLab で [Certbot](https://launchpad.net/~certbot/+archive/ubuntu/certbot) で [Let's Encrypt - Free SSL/TLS Certificates](https://letsencrypt.org/) の証明書を使うので、そのファイルの有無で設定を分岐します。

これで、最初の証明書がない状態や vagrant などでの検証環境では http でつながり、証明書発行後に `sudo gitlab-ctl reconfigure` すれば https でつながるようになる、という設定が実現できます。

    if File.exist?("/etc/letsencrypt/live/#{gitlab_domain}/fullchain.pem")
      external_url "https://#{gitlab_domain}"
      nginx['redirect_http_to_https'] = true
      nginx['ssl_certificate'] = "/etc/letsencrypt/live/#{gitlab_domain}/fullchain.pem"
      nginx['ssl_certificate_key'] = "/etc/letsencrypt/live/#{gitlab_domain}/privkey.pem"
    else
      external_url "http://#{gitlab_domain}"
    end
    nginx['custom_gitlab_server_config'] = 'location ^~ /.well-known { root /var/www/letsencrypt; }'

## GitLab Container Registry 設定

Docker の Container Registry の [GitLab Container Registry](https://docs.gitlab.com/ce/user/project/container_registry.html) も設定します。
GitLab と同様に証明書の有無で分岐しています。

Container Registry は https を使わない場合、 insecure-registry で困ることになると思います。

    if File.exist?("/etc/letsencrypt/live/#{registry_domain}/fullchain.pem")
      registry_external_url "https://#{registry_domain}"
      registry_nginx['redirect_http_to_https'] = true
      registry_nginx['ssl_certificate'] = "/etc/letsencrypt/live/#{registry_domain}/fullchain.pem"
      registry_nginx['ssl_certificate_key'] = "/etc/letsencrypt/live/#{registry_domain}/privkey.pem"
    else
      registry_external_url "http://#{registry_domain}"
    end
    registry_nginx['custom_gitlab_server_config'] = 'location ^~ /.well-known { root /var/www/letsencrypt; }'

設定はしましたが、今のところ使っていません。

## GitLab Mattermost 設定

GitLab Mattermost も同様に https の設定をします。

    if File.exist?("/etc/letsencrypt/live/#{mattermost_domain}/fullchain.pem")
      mattermost_scheme = "https"
      mattermost_port = 443
      mattermost_external_url "#{mattermost_scheme}://#{mattermost_domain}"
      mattermost_nginx['redirect_http_to_https'] = true
      mattermost_nginx['ssl_certificate'] = "/etc/letsencrypt/live/#{mattermost_domain}/fullchain.pem"
      mattermost_nginx['ssl_certificate_key'] = "/etc/letsencrypt/live/#{mattermost_domain}/privkey.pem"
      mattermost['service_use_ssl'] = true
    else
      mattermost_scheme = "http"
      mattermost_port = 80
      mattermost_external_url "#{mattermost_scheme}://#{mattermost_domain}"
    end
    mattermost_nginx['custom_gitlab_mattermost_server_config'] = 'location ^~ /.well-known { root /var/www/letsencrypt; }'

[Email Batching](https://docs.gitlab.com/omnibus/gitlab-mattermost/#email-batching) にはポート番号付きで書いてありますが、ポート番号付きで設定していると Omnibus GitLab 9.2.0 の Mattermost で Bad token type error になってしまってログインできなくなってしまったので、ポート番号なしで設定しています。

    # required by Email Batching
    #mattermost['service_site_url'] = "#{mattermost_scheme}://#{mattermost_domain}:#{mattermost_port}"
    # port causes Bad token type error https://github.com/mattermost/platform/issues/6489
    mattermost['service_site_url'] = "#{mattermost_scheme}://#{mattermost_domain}"
	mattermost['email_enable_batching'] = true

Mattermost の方だけにアカウントを作れる必要はなかったので、 GitLab 連携のアカウントだけに制限しています。

    mattermost['email_enable_sign_in_with_email'] = false # gitlab accounts only
    mattermost['email_enable_sign_up_with_email'] = false # default

チーム作成も GitLab のグループ作成のときに連動して作ることを想定して制限しています。
ただし Admin Area の方からグループを作成しようとすると「Create a Mattermost team for this group」のチェックボックスが出てこないので、 Dashboard の Groups の方から New group で作成する必要があるようです。

    mattermost['team_enable_team_creation'] = false
    mattermost['team_enable_user_creation'] = true # default, required by GitLab SSO
    mattermost['team_restrict_creation_to_domains'] = 'example.com'

デフォルトの言語設定をしておきます。

    mattermost['localization_server_locale'] = 'ja'
    mattermost['localization_client_locale'] = 'ja'

Webhook を有効にします。
このあたりの設定は Web UI から変更しても `sudo gitlab-ctl reconfigure` すると、こちらの設定が優先されるようです。

    mattermost['service_enable_incoming_webhooks'] = true

## GitLab Pages

[GitLab Pages](https://docs.gitlab.com/ce/user/project/pages/index.html) の設定をします。

github.io と同様にグループやユーザーのサブドメインで見えるようになるので、 xip.io や nip.io などのようなワイルドカード DNS を使うのが簡単だと思います。

    pages_external_url 'http://10.0.0.10.nip.io'

この設定で `http://hello.10.0.0.10.nip.io/world/` などでみえるようになります。

https を使うには証明書と秘密鍵を Web UI から設定する必要があるようなので、自動化は難しそうな印象を受けました。

## 機能制限

デフォルトではグループを作成できないようにしました。

    # GitLab user privileges
    gitlab_rails['gitlab_default_can_create_group'] = false
    gitlab_rails['gitlab_username_changing_enabled'] = false

すべてのプロジェクトですべての機能を使うわけではないので、デフォルトでは無効にしました。

    # Default project feature settings
    gitlab_rails['gitlab_default_projects_features_issues'] = false
    gitlab_rails['gitlab_default_projects_features_merge_requests'] = false
    gitlab_rails['gitlab_default_projects_features_wiki'] = false
    gitlab_rails['gitlab_default_projects_features_snippets'] = false
    gitlab_rails['gitlab_default_projects_features_builds'] = false
    gitlab_rails['gitlab_default_projects_features_container_registry'] = false

## certbot 実行

Ubuntu 16.04.2 LTS にはまだ certbot が含まれていないので、 [Certbot PPA](https://launchpad.net/~certbot/+archive/ubuntu/certbot) から certbot パッケージをインストールして、

    sudo certbot certonly --webroot --webroot-path=/var/www/letsencrypt -d gitlab.example.jp
    sudo certbot certonly --webroot --webroot-path=/var/www/letsencrypt -d registry.example.jp
    sudo certbot certonly --webroot --webroot-path=/var/www/letsencrypt -d mattermost.example.jp

のように証明書を発行して、

    sudo gitlab-ctl reconfigure

で反映させました。

後は certbot の timer で 60 日ごとに証明書が自動更新されるはずです。

## Web から設定

以上で `/etc/gitlab/gitlab.rb` での設定や端末での作業は終わりなので、 Web から設定していきます。

## グループ作成

Mattermost のチーム作成に必要だったので、 GitLab の Administrator (@root) ユーザーには Web UI からグループ作成権限をつけて、 Dashboard の Groups の方から New group で「Create a Mattermost team for this group」のチェックを入れてグループを作成しました。

チェックを入れて作成しても、特にリンクなどはつかないようなので、 Group の Description に `https://mattermost.example.jp/` の URL を書いておきました。

## GitLab からの通知用 Webhook 設定

Mattermost のチャンネルのサイドバー3点マークから「統合機能」を選んで「内向きのウェブフック」を追加します。
事前に通知用にチャンネルを作成して、送信先を分離しておくと良いかもしれません。
追加したら URL を控えておきます。

GitLab の Admin area (上のレンチ) を開いて、右上の歯車から「Service Templates」を開きます。
「Mattermost notifications」を選んで「Active」にチェックを入れて、「Webhook」に先ほどの URL を貼り付けます。
できるだけ通知して欲しいので「Notify only broken pipelines」と「Notify only default branch」のチェックは外しました。
最後に「Save」で設定を保存するとプロジェクトを作成する時に自動で通知する設定が入るようになります。

## まとめ

以上で GitLab の基本的な機能は使えるようになります。
後は GitLab CI (GitLab Runner) の設定と Dokku の設定とバックアップの設定になります。
