---
layout: post
title: "React On Rails の react-webpack-rails-tutorial を Dokku で試してみた"
date: 2016-10-02 22:24:55 +0900
comments: true
categories: dokku ruby rails react
---
react.js を rails と組み合わせて使うにはどうすればいいんだろうと思って調べてみると、`react-rails` gem の他に `react_on_rails` gem というもっとまとめていろんなことの面倒を見てくれるものがあったので、そのサンプルアプリである
[react-webpack-rails-tutorial](https://github.com/shakacode/react-webpack-rails-tutorial "react-webpack-rails-tutorial")
を [Dokku](https://github.com/dokku/dokku/ "Dokku") にデプロイできるか試してみました。

<!--more-->

## 対象バージョン

- OS X El Capitan 10.11.6
- VirtualBox 5.1.6
- Vagrant 1.8.6
- Dokku master (0.7.2 からちょっと変更が進んだもの)
- react-webpack-rails-tutorial master
- ruby 2.3.1
- rails 5.0.0
- node 6.7.0

## 環境構築

まず https://github.com/dokku/dokku を git clone したディレクトリで作業します。

vagrant の provision から何度も試すようなら、 apt で日本のミラーを使うように shell provisioning を追加しておきます。

```diff
diff --git a/Vagrantfile b/Vagrantfile
index 4f3fc6c..bccceb9 100644
--- a/Vagrantfile
+++ b/Vagrantfile
@@ -47,6 +47,7 @@ Vagrant::configure("2") do |config|
       vb.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
     end
 
+    vm.vm.provision :shell, :inline => "sed -i -e 's,//us\\.archive\\.ubuntu\\.com,//jp.archive.ubuntu.com,' /etc/apt/sources.list"
     vm.vm.provision :shell, :inline => "export DEBIAN_FRONTEND=noninteractive && apt-get update > /dev/null && apt-get -qq -y install git > /dev/null && cd /root/dokku && #{make_cmd}"
     vm.vm.provision :shell, :inline => "cd /root/dokku && make dokku-installer"
     vm.vm.provision :shell do |s|
```

`vagrant up` します。
gliderlabs/herokuish の docker イメージのダウンロードなどもあるので時間がかかります。

## Web UI での設定

http://dokku.me/ を開いて初期設定します。
`Hostname` を `dokku.me` に変更して `Use virtualhost naming for apps` にチェックを入れて `Finish Setup` を押します。

意図した動作かどうかはわかりませんが、この作業をしなくても `app-name.dokku.me` は使えました。
(`not-found-app.dokku.me` で初期設定画面は出てくるまま)

## 初期設定

`/vagrant/tmp/init.sh` に以下のファイルをおいて実行します。
`tmp` は `.gitignore` に入っていてローカルな作業ファイルをおくのに都合が良いです。

内容としては以下のようなことをしています。

- `docker` コマンドを `sudo` なしで呼べるように `docker` グループに `vagrant` ユーザーを追加
- `/home/dokku` を調べたりするときなどに都合が良いように `dokku` グループに `vagrant` ユーザーを追加
- ruby などのダウンロードでタイムアウトしないように `CURL_TIMEOUT` を増やす
- `dokku run` などで一時的に作成されるコンテナーをデフォルトで削除するように `DOKKU_RM_CONTAINER` を設定
- `~/.ssh/known_hosts` がハッシュ化されているとどの行がどのホストかわからなくなるので `HashKnownHosts no` で無効化
- あとで git push のときに使う `10.0.0.2` のホスト鍵を `~/.ssh/known_hosts` に追加
- ssh の鍵ペアを作成して `dokku ssh-keys:add` で登録
- ruby のビルドに必要なパッケージなどをインストール
- anyenv, rbenv, ndenv をインストール
- `~/.gemrc` を作成してデフォルトでドキュメントのインストールを無効化
- ndenv で最新の node をインストール

デプロイするだけなら anyenv などは不要ですが、あとで開発環境としても動かしたかったので入れています。

```sh
#!/bin/bash
set -euo pipefail
set -x
cd /home/vagrant
sudo usermod -aG docker vagrant
sudo usermod -aG dokku vagrant
dokku config:set --global CURL_TIMEOUT=120
dokku config:set --global DOKKU_RM_CONTAINER=1
if [[ ! -e "$HOME/.ssh/config" ]]; then
  echo "HashKnownHosts no" >"$HOME/.ssh/config"
fi
if [[ ! -e "$HOME/.ssh/known_hosts" ]]; then
  ssh-keyscan -t ecdsa-sha2-nistp256 10.0.0.2 | grep -v '#' > "$HOME/.ssh/known_hosts"
fi
if [[ ! -e "$HOME/.ssh/id_rsa" ]]; then
  ssh-keygen -N '' -f "$HOME/.ssh/id_rsa"
  sudo dokku ssh-keys:add vagrant "$HOME/.ssh/id_rsa.pub"
fi

if [[ -z "$(dpkg -l | grep libsqlite3-dev)" ]]; then
  sudo sed -i~ -e 's/us\.archive/jp.archive/' /etc/apt/sources.list
  sudo apt-get update
  sudo apt-get -y install autoconf bison build-essential libssl-dev libyaml-dev libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm3 libgdbm-dev
  sudo apt-get -y install libpq-dev
  sudo apt-get -y install libsqlite3-dev
  sudo apt-get -y install jq
fi
if [[ ! -d ~/.anyenv ]]; then
  git clone https://github.com/riywo/anyenv.git ~/.anyenv
  echo 'export PATH="$HOME/.anyenv/bin:$PATH"' >> ~/.bashrc
  echo 'eval "$(anyenv init -)"' >> ~/.bashrc
fi
if [[ -z "$(command -v anyenv)" ]]; then
  export PATH="$HOME/.anyenv/bin:$PATH"
  set +x
  eval "$(anyenv init - --no-rehash)"
  set -x
fi
if [[ ! -d ~/.anyenv/envs/rbenv ]]; then
  anyenv install rbenv
fi
if [[ ! -f ~/.gemrc ]]; then
  cat <<EOF >~/.gemrc
install: --no-rdoc --no-ri --format-executable
update: --no-rdoc --no-ri --format-executable
EOF
fi
if [[ ! -d "$HOME/.anyenv/envs/ndenv" ]]; then
  anyenv install ndenv
fi
node_version=$(ndenv install -l | grep '^ *v' | tail -n1 | xargs)
if ! ndenv versions | grep -q "$node_version"; then
  ndenv install "$node_version"
  ndenv global "$node_version"
fi
```

## react-webpack-rails-tutorial のデプロイ

一度 `exit` で抜けて `vagrant ssh` で入りなおしてグループの変更や anyenv などのインストールを反映します。

`/vagrant/tmp/react-webpack-rails-tutorial.sh` に以下のファイルをおいて実行します。

内容としては以下のようなことをしています。

- https://github.com/shakacode/react-webpack-rails-tutorial の取得
- react-webpack-rails-tutorial アプリの作成 (リンク作業に必要)
- dokku-postgres が入っていなければ入れる
- react-webpack-rails-tutorial-db を作ってリンク
- dokku という remote を追加
- https://github.com/heroku/heroku-buildpack-multi を使うため `.buildpacks` を作成
- `rake db:migrate` の自動実行のため `app.json` を作成
- デプロイ

```sh
#!/bin/bash
set -euo pipefail
set -x
cd
if [[ ! -d react-webpack-rails-tutorial ]]; then
  git clone https://github.com/shakacode/react-webpack-rails-tutorial
fi
if [[ ! -d /home/dokku/react-webpack-rails-tutorial ]]; then
  dokku apps:create react-webpack-rails-tutorial
fi
if [[ ! -d /var/lib/dokku/plugins/available/postgres ]]; then
  sudo dokku plugin:install https://github.com/dokku/dokku-postgres.git || :
  sudo docker pull gliderlabs/herokuish
fi
if [[ ! -f /home/dokku/react-webpack-rails-tutorial/DOCKER_OPTIONS_RUN ]]; then
  dokku postgres:create react-webpack-rails-tutorial-db || :
  dokku postgres:link react-webpack-rails-tutorial-db react-webpack-rails-tutorial
fi
cd react-webpack-rails-tutorial
if ! git remote | grep -q dokku; then
  git remote add dokku dokku@10.0.0.2:react-webpack-rails-tutorial
fi
cat >.buildpacks <<EOF
https://github.com/heroku/heroku-buildpack-nodejs
https://github.com/heroku/heroku-buildpack-ruby
EOF
git add .buildpacks
git commit -m 'Add .buildpacks' || :
cat <<EOF >app.json
{
  "scripts": {
    "dokku": {
      "predeploy": "bundle exec rake db:migrate"
    }
  }
}
EOF
git add app.json
git commit -m "Set script.dokku.predeploy to app.json" || :
git push dokku master
```

## 動作確認

ブラウザーで http://react-webpack-rails-tutorial.dokku.me/ を開いて動作確認します。

## .buildpacks の順番

`.buildpacks` で nodejs, ruby という順番で指定しましたが、逆の順番にすると以下のエラーで失敗しました。
`react_on_rails` で使っているので、 nodejs の方を先に入れる必要があるようです。

```text
-----> Preparing app for Rails asset pipeline
       Running: rake assets:precompile
       cd client && npm run build:production
       sh: 1: npm: not found
       rake aborted!
       Command failed with status (127): [cd client && npm run build:production...]
       /tmp/build/vendor/bundle/ruby/2.3.0/gems/react_on_rails-6.1.0/lib/tasks/assets.rake:33:in `block (3 levels) in <top (required)>'
       /tmp/build/vendor/bundle/ruby/2.3.0/gems/rake-11.2.2/exe/rake:27:in `<top (required)>'
       Tasks: TOP => assets:precompile => react_on_rails:assets:compile_environment => react_on_rails:assets:webpack
       (See full trace by running task with --trace)
       !
       !     Precompiling assets failed.
       !
```

## 開発環境設定

https://github.com/shakacode/react-webpack-rails-tutorial#basic-demo-setup を参考にして設定します。

`/vagrant/tmp/react-webpack-rails-tutorial-dev.sh` に以下のファイルをおいて実行します。

内容としては以下のようなことをしています。

- `.ruby-version` で指定されている ruby のインストール
https://github.com/thoughtbot/capybara-webkit/wiki/Installing-Qt-and-compiling-capybara-webkit#ubuntu-trusty-1404 のインストール (OS X 上で直接試したときにはこの依存をインストールする部分が大変でした)
- 余計な差分が出ないように `Gemfile.lock` に記録されているバージョンの bundler をインストール
- `bundle install` で依存している gem をインストール
- `npm install` で依存している node modules をインストール
- sqlite3 のデータベース作成

```sh
#!/bin/bash
set -euo pipefail
set -x
cd "$HOME/react-webpack-rails-tutorial"
ruby_version="$(<.ruby-version)"
if ! rbenv versions | grep -q "$ruby_version"; then
  rbenv install "$ruby_version"
fi
sudo apt-get -y install libqt4-dev libqtwebkit-dev
bundler_version=$(grep -A1 'BUNDLED WITH' Gemfile.lock | tail -n1 | tr -d ' ')
if ! gem list | grep -q bundler; then
  gem install bundler -v "$bundler_version"
fi
bundle install
npm install
rake db:setup
```

## 開発環境追加設定

`foreman start -f Procfile.hot` で起動して
http://dokku.me:5000/ で表示を確認すると「FATAL: Listen error: unable to monitor directories for changes. Visit https://github.com/guard/listen/wiki/Increasing-the-amount-of-inotify-watchers for info on how to fix this.」というエラーになるので、サイトに書いてある通り、
`echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p`
を実行しました。

実行前の `cat /proc/sys/fs/inotify/max_user_watches` は 8192 でした。

起動途中だと `ActionView::Template::Error (No such file or directory @ rb_file_s_mtime - app/assets/webpack/server-bundle.js):` というエラーになるので少し待てば良いようです。

しかし、次にアクセスしてみるとなぜか hot-assets が落ちてしまってうまく動きませんでした。
そして `ps x` でプロセスを確認するとちゃんと終了せずに残ってしまっているプロセスがあるので `pkill -f puma`, `pkill node` で終了させる必要がありました。

表示できても assets として `http://localhost:3500/` を参照しているため、ポートフォワーディングの設定追加が必要そうでした。

## Procfile.static

`foreman start -f Procfile.static` で起動して `http://dokku.me:5000/` を開いたところ、開けることもありましたが、落ちることも多くて安定しませんでした。

## まとめ

Vagrant の VM は開発環境としてはなぜか安定しませんでしたが、 Dokku をデプロイ先としては安定して使えたので、
`react_on_rails` は開発対象の選択肢として入れても良さそうな感じがしました。

開発環境として OS X 上で直接動かしたときは問題なく動いたので、`localhost` ではなく `10.0.2.2` を使ってしまったのが不具合の原因だったのかもしれませんが、もう少し調べてみないとなんとも言えません。
