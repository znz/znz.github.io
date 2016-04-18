---
layout: post
title: "公開鍵認証 + libpam-google-authenticator による二要素認証を特定のユーザーだけ対象に導入する"
date: 2016-04-18 21:26:42 +0900
comments: true
categories: linux debian
---
普通に `libpam-google-authenticator` を PAM の設定に追加するだけだと公開鍵認証の時に使われなくて二要素認証として嬉しくなかったのと、
`libpam-google-authenticator` による二要素認証をいきなり全ユーザーに導入してしまうと `google-authenticator` コマンドによるトークン作成をしていないユーザーが入れなくなってしまったり、リモートバックアップ処理の自動実行などで入れなくなったりして困るので、
一部のユーザーだけ二要素認証が必須になる設定を考えてみました。

<!--more-->

## 対象バージョン

- Debian GNU/Linux 8.4 (jessie)
- openssh-server 1:6.7p1-5+deb8u1
- libpam-google-authenticator 20130529-2

## 設定時の注意

PAM の設定変更は失敗するとログインできなくなって危険なので、設定を戻したりできるシェルを最低一個は残した状態で設定を変更することをおすすめします。

## PAM の設定

PAM の設定では `@include common-auth` の代わりに `pam_unix.so` を `pam_google_authenticator.so` に置き換えた設定を `/etc/pam.d/sshd` に追加しました。

これで `keyboard-interactive` 認証では unix password による認証は使えなくなって `libpam-google-authenticator` による認証だけになります。

ワンタイムパスワードなので、入力している値を見られても困らないし、実際 GitHub などでの入力画面では隠されていないので、 `/usr/share/doc/libpam-google-authenticator/README.gz` にも書いてある `echo_verification_code` の設定も追加してエコーバックされるようにしてみました。

```plain /etc/pam.d/sshd
+auth [success=1 default=ignore] pam_google_authenticator.so echo_verification_code
+auth requisite pam_deny.so
+auth required pam_permit.so
 # Standard Un*x authentication.
-@include common-auth
+#@include common-auth
```

## sshd の設定

他の `pam_google_authenticator.so` 導入記事にも書いてあるように
`ChallengeResponseAuthentication` を `yes` に変更します。
この設定を変更しないと `Verification code:` の入力プロンプトが出てこなくて、
認証コードの入力ができません。

```plain /etc/ssh/sshd_config
ChallengeResponseAuthentication yes
```

最後に適当なグループ (今回は `/var/log/` のログファイルのグループなどに利用されている `adm` グループを利用しましたが `sudo` グループなどでも良いかもしれません) を `Match` で指定して、そのグループに属するユーザーの時だけ `AuthenticationMethods` で公開鍵認証と `keyboard-interactive` 認証の両方を必須にしました。

```plain /etc/ssh/sshd_config
Match Group adm
AuthenticationMethods publickey,keyboard-interactive
```

## トークンを生成しているユーザーだけ有効にする設定

`google-authenticator` コマンドで `~/.google-authenticator` を生成しているユーザーだけ有効にすることができたので、その方法もメモしておきます。

方法としては `pam_exec` を使ってファイルの存在チェックをすれば可能でした。
`pam_exec.so` の引数部分では直接環境変数展開ができなかったので、別途外部に実行ファイルを用意する方法がデバッグもしやすくておすすめです。

存在チェックが成功すればそのまま次の行に進んで、存在しなければ後続の 2 行を飛ばして `pam_permit.so` で許可するようにしました。

`pam_exec.so` に `quiet` をつけないと `~/.google-authenticator` がない場合に毎回 `/usr/local/bin/check_google_authenticator.sh failed: exit code 1` が出るので、 `quiet` をつけて抑制するようにしました。
`Authenticated with partial success.` というメッセージは `ssh` が出しているので消せませんでした。

PAM の設定の詳細については[大統一Debian勉強会 特大号 東京エリア/関西Debian勉強会のPDF](http://tokyodebian.alioth.debian.org/pdf/debianmeetingresume2012-natsu.pdf) か、[大統一Debian勉強会](http://gum.debian.or.jp/2012/) の「Linux-PAMの設定について」の発表資料を参考にしてください。

```plain /etc/pam.d/sshd
auth [success=ignore default=2] pam_exec.so quiet /usr/local/bin/check_google_authenticator.sh
auth [success=1 default=ignore] pam_google_authenticator.so echo_verification_code
auth requisite pam_deny.so
auth required pam_permit.so
```

存在のチェック用スクリプトは `pam_exec` 経由で実行された時には `HOME` 環境変数が設定されていなくて、代わりに `PAM_USER` などが設定されているのを利用して `HOME` を設定されていなければ設定するようにしました。

```bash /usr/local/bin/check_google_authenticator.sh
#!/bin/sh
: ${HOME:=$(getent passwd "$PAM_USER" | awk -F: '{print $6}')}
test -f "$HOME/.google_authenticator"
```

## /etc/pam.d/sshd にまとめる書き方

発表資料の PDF を確認して気付いたのですが、 `[ ]` でくくれば空白の入った引数も渡せるので、シェルを経由するようにすれば変数展開付きのコマンドを含められました。

```plain /etc/pam.d/sshd
auth [success=ignore default=2] pam_exec.so quiet /bin/sh -c [: ${HOME:=$(getent passwd "$PAM_USER" | awk -F: '{print $6}')}; test -f "$HOME/.google_authenticator"]
auth [success=1 default=ignore] pam_google_authenticator.so echo_verification_code
auth requisite pam_deny.so
auth required pam_permit.so
```

## google-authenticator コマンドによるトークンの作成

二要素認証を使うユーザーで `google-authenticator` コマンドを実行してトークンを作成して、 iOS なら Google Authenticator のアプリに、 Android なら Google 認証システムアプリに QR コードを読み込ませておきます。
`google-authenticator` コマンドの質問は全部 `y` で良いと思います。

設定は `~/.google_authenticator` に保存されています。

テスト環境では QR コードは読み込ませずに `emergency scratch codes` を使っていたのですが、
`emergency scratch codes` は使っていくと `~/.google_authenticator` からどんどん減っていくので、適当なタイミングで `google-authenticator` コマンドを使って再生成させないと入れなくなりそうでした。

## 失敗した設定例

`Match` で `ChallengeResponseAuthentication` を設定しようとしましたが、 `Directive 'ChallengeResponseAuthentication' is not allowed within a Match block` というエラーで設定できませんでした。

公開鍵ごとに二要素認証の設定ができないか、検討してみましたが、使えそうな設定項目が見つかりませんでした。
