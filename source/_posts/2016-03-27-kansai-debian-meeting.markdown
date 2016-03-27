---
layout: post
title: "第 108 回 関西 Debian 勉強会に参加しました"
date: 2016-03-27 14:00:00 +0900
comments: true
categories: debian
---
[第 108 回 関西 Debian 勉強会](https://wiki.debian.org/KansaiDebianMeeting/20160327 "第 108 回 関西 Debian 勉強会")に参加しました。

<!--more-->

以下メモです。

## オープニングなど

- [ハッシュタグは #kansaidebian](https://twitter.com/search?q=%23kansaidebian&src=typd)
- [前回](https://wiki.debian.org/KansaiDebianMeeting/20160228) の事前配布資料がまだアップロードされていなかった。
- http://mozilla.debian.net/ から jessie にも firefox がインストールできる。
- [Application Binary Interface (ABI) Tracker](http://abi-laboratory.pro/tracker/)
- その他の話題
- [Badlock Bug](http://badlock.org/ "Badlock Bug")
- [WindowsおよびSambaの重大なバグ「Badlock」、4月12日のパッチリリースが告知される](http://security.srad.jp/story/16/03/25/2045223/ "WindowsおよびSambaの重大なバグ「Badlock」、4月12日のパッチリリースが告知される")
- Wheezy LTS
- [Debian 管理者ハンドブック](https://debian-handbook.info/browse/ja-JP/stable/ "Debian 管理者ハンドブック") のペーパーバック版が発売されているという話
- jessie で tdiary が消えた理由などを聞いた。
- [tdiary REMOVED from testing](https://packages.qa.debian.org/t/tdiary/news/20140624T163916Z.html "tdiary REMOVED from testing") にあるように ruby-imagesize が壊れたのが原因で upstream で ruby-fastimage に依存が変わっていたので、新規パッケージで入れて tdiary の更新をしようとしたら freeze されて jessie に入らなかったという感じの話だった。
- doorkeeper の申し込みで事前課題の入力必須だったのがわかりにくいという話
- doorkeeper で管理者側でアンケート結果の一覧が表示できないので不便という話
- doorkeeper の方は事前課題として「systemd で起動する Jessie 環境を御用意下さい．幾つか設定を変更することがありますので，VM 環境の方が無難かもしれません．」というのが出ていたのに debian wiki の方には書いていなかった話

## 休憩

- tdiary で org-mode

## systemd に浸ってみた

- `systemctl`
- `systemctl status`
- `systemd-analyze`
- `systemd-analyze blame`
- `systemd-cgls`
- `timedatectl`
- `systemctl status systemd-timesyncd`
- `sudo timedatectl set-ntp true`
- `systemctl status -l systemd-timesyncd`
- `/etc/systemd/timesyncd.conf`
- `timesyncd.conf(5)` の man がない (jessie だと)
- sid だと設定が変わっているが jessie と同じ設定でも動いている
- `/usr/lib/systemd/ntp-units.d/`
- `/usr/share/doc/systemd/changelog.Debian.gz` に `* Run timesyncd in virtual machines. (Closes: #762343)` という行があって仮想環境では upstream の timesyncd は動かないようになっている
- `dmesg` と `dmesg -T`
- [How systemd-timesyncd handles leap seconds](https://0xstubs.org/systemd-timesyncd-and-leap-seconds/ "How systemd-timesyncd handles leap seconds")
- `systemctl status systemd-resolved`
- 使うには有効にした後、 `/etc/nsswitch.conf` の `dns` を `resolve` に変更、または `/run/systemd/resolve/resolv.conf` を `/etc/resolv.conf` にシンボリックリンク
- `/etc/systemd/resolved.conf` に設定
- `systemctl status systemd-networkd`
- `/etc/systemd/network/` 以下で設定
- debian では udev に設定が入っていて eth0 とかになる
- `systemd.network(5)` の man を見る
- `Driver=` が有効になるのは systemd 216 で jessie では使えない
- `[DHCP]` セクションに `UseDNS=` という設定がある
- `systemd.link(5)` や `systemd.netdev(5)` も参照
- `ConditionHost=` などを使えば単一の設定ファイルを複数ホストで共有できる
- 無線を使う場合には向かない
- NetworkManager とは排他
- `sudo apt-get install systemd-cron`
- cron, anacron とは conflict していて入れ替わる
- `systemctl list-unit-files | grep cron`
- `cat /lib/systemd/system/cron-hourly.timer`
- `crontab -e` は動かなくなる
- `~/.config/systemd/user/` 以下にユーザーごとの設定を置ける
- `systemctl --user list-unit-files`
- GNOME や KDE だと systemd を使うしかなさそう
- Xfce だと systemd はハイバネーションで失敗するのでやめた方が良い
- systemd の情報を見る時はバージョン番号を確認しましょう
