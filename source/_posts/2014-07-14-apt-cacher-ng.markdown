---
layout: post
title: "apt-cacher-ngでliveイメージ作成を繰り返す時の無駄なダウンロードを減らす"
date: 2014-07-14 23:53:14 +0900
comments: true
categories: debian ubuntu live vagrant ruby
---
[rubylive-builder](https://github.com/znz/rubylive-builder "rubylive-builder")
で
[RubyLive](https://github.com/znz/rubylive "RubyLive")
という Debian wheezy ベースの Live イメージを作成するときに
`apt-get update` などで何度も無駄にダウンロードしてしまうので、
`apt-cacher-ng` で Live イメージ作成を繰り返す時の無駄なダウンロードを減らすことにしました。

<!--more-->

## Vagrant の provision でインストール

Vagrantfile では

```ruby Vagrantfile
  config.vm.provision :shell do |shell|
    shell.path = "provision.sh"
  end
```

のようにシェルスクリプトでプロビジョニングしているだけだったので、
その中で以下のようにインストールして設定するようにしました。

```bash provision.sh
apt-get install -y apt-cacher-ng
echo 'Acquire::http::Proxy "http://localhost:3142/";' >/etc/apt/apt.conf.d/02proxy
```

Vagrantfile で以下のようにポートフォワーディングを設定していれば
[第315回　apt-cacher-ngを使ってAPT用キャッシュプロキシの構築：Ubuntu Weekly Recipe｜gihyo.jp … 技術評論社](http://gihyo.jp/admin/serial/01/ubuntu-recipe/0315 "第315回　apt-cacher-ngを使ってAPT用キャッシュプロキシの構築：Ubuntu Weekly Recipe｜gihyo.jp … 技術評論社")
の2ページ目に説明があるようにヒット率などを確認できます。

```ruby Vagrantfile
  # apt-cacher-ng
  config.vm.network "forwarded_port", guest: 3142, host: 3142
```

## rake コマンドで環境変数を渡す

`APT_HTTP_PROXY=http://localhost:3142 rake` でも良かったのですが、
rake コマンドは引数の `FOO=bar` を `ENV` に設定してくれるので、
`rake APT_HTTP_PROXY=http://localhost:3142` で渡して、
Rakefile の中では以下のように受け取って `lb config` に渡しました。

```ruby Rakefile
desc "config RubyLive"
task :config => [:clean] do
  sh 'lb config'
  if ENV['APT_HTTP_PROXY']
    sh "lb config --apt-http-proxy #{ENV['APT_HTTP_PROXY']}"
  end
end
```

## live-build で apt-cacher-ng を使う

既に出てきたように
`lb config` の `--apt-http-proxy` オプションや `--apt-ftp-proxy` オプションで指定すると
Live イメージ作成の時に proxy を使ってくれるようになります。
今回は apt-line に `http` しか使っていないので
`--apt-http-proxy` だけ指定しています。

もちろん、作成後の Live イメージには proxy 設定は残りません。

## 感想

live-build は cache ディレクトリにも、かなりキャッシュしてくれるのですが、
`apt-get update` などの proxy じゃないとキャッシュしにくいものもあるので、
どんな場合でもダウンロード量削減に役に立ちそうだと思いました。
