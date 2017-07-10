---
layout: post
title: "GitLabと連携するDokkuの初期設定"
date: 2017-07-10 21:30:00 +0900
comments: true
categories: gitlab linux ubuntu dokku
---
GitLab と Dokku (と一部 Heroku) を使って CI/CD (Continuous Integration / Continuous Deployment) 環境を作ってみた話の続きです。
今回は Dokku の設定の話になります。

[gitlab カテゴリー](/blog/categories/gitlab/)で一覧が見えます。

<!--more-->

## 対象バージョン

- Ubuntu 16.04.2 LTS
- Omnibus GitLab 9.1.4-ce.0 (インストール時) から 9.3.5-ce.0 (記事執筆時)
- gitlab-ci-multi-runner 9.2.0 (インストール時) から 9.3.0 (記事執筆時)
- Dokku 0.9.4 (インストール時) から 0.10.2 (記事執筆時)

## Dokku とは?

[Dokku - The smallest PaaS implementation you've ever seen](http://dokku.viewdocs.io/dokku/)は bash で書かれた OSS の PaaS です。

Heroku のように git push でデプロイするというのが基本的な使い方になります。

環境変数の操作などのような heroku コマンドで操作に相当することは ssh 経由と Dokku ホスト上での dokku コマンドの両方でできることが多いです。
プラグインのインストールなど、一部の操作は Dokku ホスト上で直接実行する必要があります。

データベースなどはプラグインで対応しています。

Dokku の更新が止まっていた時にできた [Dokku Alternative](https://github.com/dokku-alt/dokku-alt) という fork もありましたが、メンテナンスが止まっているので使うべきではありません。

最近はコアプラグインから少しずつ Go 言語への移行を進めているようです。

## インストール

[Dokku](http://dokku.viewdocs.io/dokku/) に書いてあるように `bootstrap.sh` を使うと自動で `get.docker.com` からの docker のインストールも含めて、 apt から dokku をインストールしてくれます。

## 初期設定

debconf であらかじめ設定しておくか、`web_config` を使って設定します。

デフォルトのままなどで `web_config` が有効な場合、 Dokku をインストールしたホストをブラウザーで開くと初期設定画面が出てきます。

そこでデプロイや ssh 経由での操作に使う ssh の公開鍵の登録とホスト名の設定をします。
ホスト名はアプリケーションごとに個別のバーチャルホストを別途設定できるので、とりあえず xip.io や nip.io を使っておくのが手軽だと思います。

## プラグインのインストール

[プラグイン一覧](http://dokku.viewdocs.io/dokku/community/plugins/)から Compatibility を確認して必要なプラグインをインストールします。

Heroku と似た感じで使いたいのなら [dokku postgres](https://github.com/dokku/dokku-postgres) を入れておくと良いと思います。

Dokku ホストで

    sudo dokku plugin:install https://github.com/dokku/dokku-postgres.git postgres

でインストールできます。

[dokku-maintenance](https://github.com/dokku/dokku-maintenance) のように Heroku だと標準で対応していることがプラグインになっていたり、
[dokku-apt](https://github.com/F4-Group/dokku-apt) のように Heroku だと複数 buildpack で対応していたようなことがプラグインになっていたりすることもあります。

## ssh の公開鍵設定

後で GitLab CI の secret variables に秘密鍵を設定するので、専用の鍵ペアを作成して、 [User Management](http://dokku.viewdocs.io/dokku/deployment/user-management/) の方法で Dokku に公開鍵を登録しておきます。

```
% ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa): /home/vagrant/.ssh/id_gitlab
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/vagrant/.ssh/id_gitlab.
Your public key has been saved in /home/vagrant/.ssh/id_gitlab.pub.
The key fingerprint is:
(略)
% cat ~/.ssh/id_gitlab.pub | ssh dokku@dokku.example.jp ssh-keys:add gitlab
```

## 動作確認

データベースの必要な例を試すなら、[Deploy tutorial](http://dokku.viewdocs.io/dokku/deployment/application-deployment/)を試してみると良いと思います。

データベースの不要な例を試すなら、[Deploying Rack-based Apps | Heroku Dev Center](https://devcenter.heroku.com/articles/rack) の `heroku create` を `git remote add dokku dokku@dokku.example.test:hello` に変えて、 `git push heroku master` の代わりに `git push dokku master` で試してみたりすると良いと思います。

## まとめ

Dokku のインストールから簡単な動作確認方法まで紹介しました。
次は `.gitlab-ci.yml` を作成して連携する話です。

[gitlab カテゴリー](/blog/categories/gitlab/)で一覧が見えます。
