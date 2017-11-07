---
layout: post
title: "gitlab-ci-multi-runner パッケージから gitlab-runner パッケージへの更新"
date: 2017-11-06 23:00:00 +0900
comments: true
categories: gitlab linux ubuntu
---
GitLab Runner が 10.0.0 だとパッケージ名が gitlab-runner に変わってしまって、そのままだと 9.5.1 から更新されなくなってしまったので、対応しました。

<!--more-->

## 対象バージョン

- Ubuntu 16.04.3 LTS
- gitlab-ci-multi-runner (9.5.1) から gitlab-runner (10.1.0)

## GitLab Runner とは?

[GitLab Runner](https://docs.gitlab.com/runner/) とは GitLab CI のジョブを実行する部分のことです。

詳細は[インストール時の記事](/blog/2017-07-09-gitlab-runner.html)を参照してください。

## インストール方法

[Install GitLab Runner using the official GitLab repositories - GitLab Documentation](https://docs.gitlab.com/runner/install/linux-repository.html) が gitlab-runner に改名後のインストール方法になっているので、参考にして移行しました。

## 移行方法

簡単にまとめると apt の設定を変えて `gitlab-runner` パッケージを入れ直すだけでした。

GitLab (GitLab CI) への登録や `/etc/gitlab-runner/config.toml` はそのままで大丈夫でした。

## apt-line の更新

`/etc/apt/sources.list.d/` 以下に入っている

    deb https://packages.gitlab.com/runner/gitlab-ci-multi-runner/ubuntu xenial main

の apt-line を削除して、

    deb https://packages.gitlab.com/runner/gitlab-runner/ubuntu xenial main

を追加しました。

## pinning 設定変更

`/etc/apt/preferences.d/pin-gitlab-runner.pref` で `gitlab-ci-multi-runner` パッケージを `packages.gitlab.com` のものを優先するように設定していたのを、 `gitlab-runner` パッケージに変更しました。

## パッケージのインストール

`gitlab-runner` パッケージをインストールすると自動的に `gitlab-ci-multi-runner` パッケージと置き換わりました。

`gitlab-ci-multi-runner` パッケージを purge しても `/etc/gitlab-runner/config.toml` が消えたりすることもないので、特に注意するような点はなさそうでした。

## ansible での例

[Use gitlab-runner instead of gitlab-ci-multi-runner](https://github.com/znz/ansible-role-gitlab-runner/commit/616a9da561360fbae940940aec49483a5ee1ce9b) のように変更しました。

移行措置として、 gitlab-ci-multi-runner の apt-line を消す処理も入れています。

## まとめ

`gitlab-ci-multi-runner` パッケージから `gitlab-runner` パッケージへの移行は GitLab (GitLab CI) への登録し直しが必要だと面倒そうと思って、 10.0.x の間は躊躇してそのままにしてしまっていましたが、 vagrant 環境で確認したところ、パッケージの更新だけで大丈夫ということがわかったので、問題なく上げることができました。
