---
layout: post
title: "webhook でサイトの git pull をする設定をした"
date: 2018-01-03 18:00:00 +0900
comments: true
categories: linux debian ubuntu gitlab lilo
---
GitLab.com に git push した時に webhook で通知を受け取って git pull という設定をしました。

方針としては Web サーバーの実行ユーザー権限の cgi で通知用のファイルを更新して、 systemd の `*.path` で監視して、別途ディレクトリの所有者権限でアップデートのシェルスクリプトを実行して、アップデートのログは journald に任せるという感じにしました。

<!--more-->

## GitLab.com の設定

Webhook が使えるシステムなら GitHub などでも同様に設定可能だと思います。

- Settings - Integrations で Webhooks 設定
- URL: `https://lilo.linux.or.jp/trigger/update.cgi` のような感じ
- Secret Token あり
- Trigger : Push events のみ
- Enable SSL verification はチェックありのまま

## trigger cgi

URL がわかっていても Secret Token がちゃんと設定されていないリクエストはエラーを返すようにしました。

内容はチェックせずに通知用のファイルにリクエスト内容をそのまま書き込んでデバッグ用に使えるようにしました。

`/home/www` は CGI の権限で書き込めるディレクトリです。

```sh
#!/bin/sh
set -e
if [ x"$HTTP_X_GITLAB_TOKEN" = x"XXXXXXXXXXXXXXXXXXXX" ]; then
  cat > /home/www/trigger_update_web
  echo "Content-Type: text/plain; charset=utf-8"
  echo
  echo OK
else
  echo "Status: 403"
  echo "Content-Type: text/plain; charset=utf-8"
  echo
  echo NG
fi
```

## systemd の設定追加

後述のファイルを以下のように追加して設定しました。

```
  sudo cp lilo_web_update.path /etc/systemd/system
  sudo cp lilo_web_update.service /etc/systemd/system
  sudo systemctl daemon-reload
  sudo systemctl start lilo_web_update.path
```

## `lilo_web_update.path`

`trigger_update_web` ファイルを `PathModified` で監視して変化があれば `lilo_web_update.service` を実行するようにしました。

```
[Unit]
Description=Trigger update web

[Path]
PathModified=/home/www/trigger_update_web

[Install]
WantedBy=muti-user.target
```

## `lilo_web_update.service`

Web コンテンツに書き込めるユーザーでシェルスクリプトを実行します。

```
[Unit]
Description=Update web

[Service]
Type=oneshot
ExecStart=/path/to/lilo_web_update.sh
User=someuser
Group=someuser
```

## `lilo_web_update.sh`

flock コマンドで同時実行を抑制 (同時実行は後の方が失敗終了) した上で、何か変更されていたらそれを捨てて、 `git pull` でリモートのコンテンツで上書きするようにしています。

とりあえず自分が知っているコマンドの中でクリーンにできるものとして `git checkout .` と `git clean -dfx` を使っているだけなので、もっと良い方法があるかもしれません。

```bash
#!/bin/bash
set -euxo pipefail
exec {lock_fd}<"$0"
flock --nonblock "${lock_fd}"
cd "$(dirname "$0")/.."
git checkout .
git clean -dfx
git pull
```

## apache2 設定

trigger ディレクトリに cgi-bin の設定を参考にして ExecCGI と AddHandler の設定をしました。

```
<Directory /path/to/trigger/>
        AllowOverride None
        Options +ExecCGI -MultiViews +SymLinksIfOwnerMatch
        Require all granted
        AddHandler cgi-script .cgi
</Directory>
```

`/etc/apache2/conf-enabled/security.conf` の `.svn*` へのアクセスを禁止する設定を参考にして、 `.git*` へのアクセスを禁止しました。

```
<DirectoryMatch "/\.git">
   Require all denied
</DirectoryMatch>
```

## 動作確認

- `sudo -u someuser /path/to/lilo_web_update.sh` で git pull の動作確認
- `touch /home/www/trigger_update_web` と `sudo systemctl status lilo_web_update.service` で PathModified 経由での実行確認
- `curl -H 'X-Gitlab-Token: XXXXXXXXXXXXXXXXXXXX' https://lilo.linux.or.jp/trigger/update.cgi` で webhook 経由での動作確認

## まとめ

git push で webhook 経由でコンテンツを更新して、 journald でログを確認できるシステムを構築しました。
Unix 的にそれぞれは大した設定はしていないのですが、組み合わせるとそれなりの設定量になりました。
