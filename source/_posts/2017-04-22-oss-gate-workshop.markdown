---
layout: post
title: "OSS Gate大阪ワークショップ2017-04-22に参加した"
date: 2017-04-22 23:00:00 +0900
comments: true
categories: event oss-gate
---
[OSS Gate大阪ワークショップ2017-04-22 - OSS Gate | Doorkeeper](https://oss-gate.doorkeeper.jp/events/58579 "OSS Gate大阪ワークショップ2017-04-22 - OSS Gate | Doorkeeper")にサポーター (旧称: メンター) として参加しました。

<!--more-->

## 全体の感想

最後のアンケートにも書きましたが、前回はデモである程度メモをどの程度とるのかなどの方向性が示されていたのと、やる内容がインストールから初めて、ひっかかったところのドキュメントの改善をフィードバックする、というのが多かったのに対して、今回はデモがほぼなくて、サポーター (旧称: メンター) として応募したけど、人数の都合でビギナーになった人が多かったからか、いろんなことに挑戦していて、難しい感じでした。

作業メモの粒度としては、[過去のビギナーの作業ログ](https://github.com/oss-gate/workshop/issues?q=is%3Aissue+is%3Aclosed)から特にコメント数が多いものを参考にしてみると、こういう細かい思考過程までメモすると良いのか、というのが、 oss-gate に限らず普段の作業メモのとり方としても、参考になるかと思います。

## メモ

以下、今回の雑多なメモです。

### フォント

途中のふりかえりのときのビギナーの人がブラウザーで変わったフォントを使っていたのできいてみたところ、「スマートフォントUI」というのを使っていると教えてもらいました。

### Chef DK のアンインストール

参考のため、自分の環境にも Chef DK を入れてみていたのですが、 https://docs.chef.io/install_dk.html にアンインストールの手順があったので、それに従ってアンインストールしました。

symlink は削除前に確認してみたら、たくさんありました。

```
%  sudo rm -rf /opt/chefdk
Password:
%  sudo pkgutil --forget com.getchef.pkg.chefdk
Forgot package 'com.getchef.pkg.chefdk' on '/'.
%  find /usr/local/bin -lname '/opt/chefdk/*'
/usr/local/bin/berks
/usr/local/bin/chef
/usr/local/bin/chef-apply
/usr/local/bin/chef-client
/usr/local/bin/chef-shell
/usr/local/bin/chef-solo
/usr/local/bin/chef-vault
/usr/local/bin/cookstyle
/usr/local/bin/dco
/usr/local/bin/delivery
/usr/local/bin/foodcritic
/usr/local/bin/inspec
/usr/local/bin/kitchen
/usr/local/bin/knife
/usr/local/bin/ohai
/usr/local/bin/push-apply
/usr/local/bin/pushy-client
/usr/local/bin/pushy-service-manager
/usr/local/bin/rubocop
%  sudo find /usr/local/bin -lname '/opt/chefdk/*' -delete
%  find /usr/bin -lname '/opt/chefdk/*'
```

### 辞書

macOS の辞書ではなく、Logophile という辞書ソフトを使っているのをみました。
COBUILD のシソーラスが便利だそうです。

たぶん [Logophile](http://dicwizard.jp/logophile/ "Logophile") で、シェアウェアのようです。

### fish shell

検索しにくい名前ですが、[全訳！fishシェル普及計画【コマンドラインは怖くない】](http://fish.rubikitch.com/ "全訳！fishシェル普及計画【コマンドラインは怖くない】") に翻訳されたドキュメントがあるので、使い始めやすそうです。

個人的には rvm を入れたら環境を壊された (`zsh` なら `chpwd_function` を使えばいいのに使っていなかったらしく `cd` のカスタマイズが壊れた) ぐらいシェルはカスタマイズをしているので、乗り換える可能性は低いですが、ちょっと試してみたところ、 `echo ` のオートサジェスチョンで `$BASH_VERSION` がでてきたので、`.bash_history` をみているようです。
ちょっと試した後は、 `rm -rf ~/.local/share/fish` でクリーンな状態に戻して、また最初から試せるようにしておきました。
