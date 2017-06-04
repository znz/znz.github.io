---
layout: post
title: "cron(crontab)の代わりにsystemdのtimerを使う"
date: 2017-06-04 14:18:19 +0900
comments: true
categories: linux debian ubuntu
---
最近 [gitlab omnibus などの環境](https://github.com/znz/ansible-playbook-gitlab-dokku)を作っていて、[GitLab CE の role](https://github.com/znz/ansible-role-gitlab-ce) でバックアップ処理を定期実行するのに crontab ではなく systemd の timer を使ってみました。

<!--more-->

## 利点

- systemd 管理下で統一的に扱えるので、覚えれば楽
- ログも journald で統一されるので cron だといちいちメールが飛ぶと鬱陶しいような粒度でも簡単にログに残せる
- 環境変数なども含めた環境が本番と同じ状態ですぐに実行を試しやすい
- systemd 依存の機能が使える (後述の例では After と Requires)

などが利点に感じました。

## 欠点

- 情報が cron (crontab) に比べてまだ少ないので、何かあったときに調べにくい
- systemd に大きく依存してしまう

などが欠点に感じました。

## 確認環境

- Ubuntu 16.04.2 LTS (xenial)
- systemd 229-4ubuntu17

## 情報表示

- `systemctl list-timers` でタイマーの次回実行予定時刻、前回実行時刻などを含めて表示されます。
- `systemctl status systemd-tmpfiles-clean.timer` でタイマーの情報、`systemctl status systemd-tmpfiles-clean.service` で実行されるサービスの情報が表示されます。
- `journalctl -u systemd-tmpfiles-clean.timer` や `journalctl -u systemd-tmpfiles-clean.service` でログが表示されます。 `systemd-journal` グループに入っていない場合は `sudo` が必要かもしれません。 `systemd-journal` に入っていれば `systemctl status` でも最近のログが表示されます。
- Type=oneshot (後述) の場合、ログの Starting が実行開始時刻で Started が実行終了時刻になるようです。

## 設定ファイルの場所

`systemctl status` で `Loaded: loaded (/lib/systemd/system/systemd-tmpfiles-clean.timer; static; vendor preset: enabled)` のようにパスが出るので、システムのものは `/lib/systemd/system/` にあることがわかります。

タイマーではありませんが、 gitlab-ce では `/usr/lib/systemd/system/gitlab-runsvdir.service` に service が入っていたので、 `/usr/lib/systemd/system/` も参照されるようです。

自分で作成する場合は systemd の流儀に従って `/etc/systemd/system/` に作成すれば良いと思います。

## service 作成

`/etc/systemd/system/gitlab-backup.service` を以下の内容で作成しました。

```
[Unit]
Description=Backup gitlab
After=gitlab-runsvdir.service
Requires=gitlab-runsvdir.service

[Service]
Type=oneshot
ExecStart=/opt/gitlab/bin/gitlab-rake gitlab:backup:create CRON=1
```

- Unit の Description は適当にわかりやすい説明を書けば良いと思います。
- Service の Type は cron 代わりに使う場合は oneshot にするのが普通のようです。
- ExecStart に crontab で書いていたようにコマンドを書きます。 crontab と同じように、複雑な場合は無理にここに書こうとせずに別途シェルスクリプトなどを作成して実行する方が良さそうです。
- After と Requires はバックアップ処理を実行するのに postgresql などが実行されている必要がありそうだったので書きました。このあたりが必要かどうかは用途によると思います。

## timer 作成

`/etc/systemd/system/gitlab-backup.timer` は以下の内容で作成しました。

```
[Unit]
Description=Backup gitlab

[Timer]
OnCalendar=*-*-* 2,14:00
Persistent=true

[Install]
WantedBy=timers.target
```

- Unit の Description は適当にわかりやすい説明を書けば良いと思います。
- OnCalendar で毎日 2:00 と 14:00 に実行するように設定しています。ローカルタイムでの指定になります。詳細は systemd のドキュメントを参照してください。
- Persistent=true は[systemd/タイマー - ArchWiki](https://wiki.archlinuxjp.org/index.php/Systemd/%E3%82%BF%E3%82%A4%E3%83%9E%E3%83%BC "systemd/タイマー - ArchWiki")によると「システムの電源が切られていたなどの理由で、最後の起動時間を過ぎていた場合、すぐに実行されます」ということのようで、 anacron 的な動作が期待できるかと思って指定しています。
- Install の WantedBy=timers は `systemctl enable` や `systemctl disable` ができるようにするための定型句のようです。

## 設定反映

`sudo systemctl daemon-reload` で反映させます。
新規作成時などは必要ないかもしれませんが、実行しておくと確実です。

## 有効化

`sudo systemctl enable gitlab-backup.timer` で `/etc/systemd/system/timers.target.wants/gitlab-backup.timer` に `/etc/systemd/system/gitlab-backup.timer` へのシンボリックリンクが作成されて有効になります。

## 無効化

`sudo systemctl disable gitlab-backup.timer` で無効に戻せます。
timer を消したくなったときには disable してから timer ファイル (と service ファイル) を削除すると良いと思います。
(ファイル削除後は `sudo systemctl daemon-reload` もすると良いかもしれません。)

## テスト実行

`sudo systemctl start gitlab-backup.service` でテスト実行できます。

## 実行時間を散らす

Timer セクションに RandomizedDelaySec を設定するとランダムスリープをいれて実行時間をばらけさせることができます。
`certbot.timer` などで使われています。

試しに `RandomizedDelaySec=10min` といれてみると、これを使ったときには設定が反映されたタイミングや前回の実行終了後などの次の実行が決まった段階でランダムスリープの時間が決まるようで、 `journalctl -u gitlab-misc-backup.timer` で `gitlab-misc-backup.timer: Adding 6min 33.234976s random time.` と出て、 `systemctl list-timers` の NEXT も遅延後の時刻になっていて、実行された時のログの Starting もその時刻以降 (AccuracySec がデフォルト 1min なので NEXT に出ていた時刻よりちょっと遅かった) になっていました。

## atd

crond の crontab の代わりは service ファイルと timer ファイルを作成して反映させて有効にして、という操作が必要でした。

atd の at の代わりとしては systemd-run というコマンドがあるようです。
試しに使ったことしかないので、紹介だけに留めておきます。

## まとめ

設定ファイルが複数必要だったり、反映するのに一手間必要だったりして、使い始めは crontab より面倒ですが、 systemd との連携が必要だったり、ログ管理をまとめたかったり、 RandomizedDelaySec のように systemd の機能を使った方がすっきりする場合などは積極的に timer を使っていくと良いのではないかと思いました。
