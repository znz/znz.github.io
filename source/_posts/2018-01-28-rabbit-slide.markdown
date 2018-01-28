---
layout: post
title: "rabbitでのスライド作成から公開まで"
date: 2018-01-28 23:30:00 +0900
comments: true
categories: rabbit
---
[Rabbit](https://rabbit-shocker.org/ja/) でのスライドの作り方などの話があまり見当たらない気がするので、スライド作成から公開までの流れをまとめてみました。

<!--more-->

## 確認バージョン

- Ubuntu 16.04 LTS (xenial)
- ruby 2.4.3
- rabbit 2.2.1

## 雛形作成

`rabbit-slide new` にいくつかのオプションを指定して、雛形を作成します。

id と base name と markup language は複数の場所に影響するので、最初からしっかり決めておくか、変更する場合は `rabbit-slide new` からやり直してスライドの内容をコピーした方が簡単かもしれません。

他のオプションは config.yaml, スライドの1ページ目, README.md に影響するだけなので、後からでも簡単に変更できます。

GitHub が RD に対応してくれないので、 README を markdown にするために markup language を指定します。
スライドの記述もそれに合わせて markdown で書くことになります。

いくつかのオプションは `~/.rabbit/author.yaml` に保存されて次からは指定しなくてもデフォルトが変わります。
特に markup language も変わることは注意が必要かもしれません。

tags は `,` 区切りで複数指定できます。

allotted time はスライドの下の亀が左から右に行くまでの時間になります。
(うさぎはスライドの枚数)
テーマによっては表示されなかったり、別のものが表示されたりします。

他のオプションは見た目通りの意味です。

```
vagrant@vagrant:/vagrant$ rabbit-slide new --id about-boot2docker-format-me --base-name boot2docker-format-me --markup-language markdown --title "boot2docker の format-me の話" --tags linux --allotted-time 10m --presentation-date 2018/01/28 --name "Kazuhiro NISHIYAMA" --email "zn@mbf.nifty.com" --rubygems-user znz --slideshare-user znzjp --speaker-deck-user znz
[INFO]
Creating directory: about-boot2docker-format-me
[INFO]
Creating file:      about-boot2docker-format-me/.gitignore
[INFO]
Creating file:      about-boot2docker-format-me/.rabbit
[INFO]
Creating file:      about-boot2docker-format-me/config.yaml
[INFO]
Creating file:      about-boot2docker-format-me/README.md
[INFO]
Creating file:      about-boot2docker-format-me/Rakefile
[INFO]
Creating file:      about-boot2docker-format-me/boot2docker-format-me.md
[INFO]
Creating directory: /home/vagrant/.rabbit
[INFO]
Creating file:      /home/vagrant/.rabbit/author.yaml
```

## 表紙やテーマの設定

見出しレベル1 (HTML でいう h1, markdown だと `# ` で始まる行) ごとにページが分かれるので、作成されたディレクトリに入って `base-name.md` ファイルを編集していきます。

1ページ目は表紙とメタデータで、テーマを変更する場合は markdown なら

```
theme
:   lightning-simple
```

のように書きます。

オプションで指定したものの他には

```
content-source
:   第1回なんとか勉強会
```

のように `content-source` をつけることが多いです。

## 表示確認

`rake` で表示を確認できます。
`rabbit slide.md` のように `rabbit` コマンドにスライドファイルを指定しても同じように表示できます。

最大化でスクリーンサイズがあわないときに `rabbit -S 1920,1080 slide.md` のようにオプションを指定して実行するという使い方をしたこともあります。

## スライド記述

プレゼンにふさわしい書き方で内容を書いていきます。

個人的には箇条書きを使うことが多いです。

rabbit のスライドは `rabbit-slide-` で始まる名前で gem としても公開できるので、[gem を rabbit-slide で検索](https://rubygems.org/search?utf8=%E2%9C%93&query=rabbit-slide)して、参考にすると良いと思います。
(ダウンロード数がある程度あって、こういう gem を誰がダウンロードしてるんだろう? と思うのですが、全 gem 調査とかでダウンロードされてる回数が割とあるのではないかと推測しています。)

## 表示しながらスライド記述

スライドのファイルを書き換えると何もしなくても自動で反映されます。
残念ながら亀は戻ってしまうので、発表練習をしながら途中で気づいた点を直すのは、後回しにするかタイマーを諦める必要がありそうです。

## ページごとの追加データ

見出しレベル2 はページにプロパティなどをつけるのに使います。
書き方は表紙ページのメタデータと同様です。

見出しの行の内容は後述の「note」以外なら「プロパティ」でも「property」でもなんでも良いようです。
試した感じだと markdown では以下のような感じで設定できるようです。
何が設定できるのかはテーマ次第だと思います。

```
## プロパティ

background-color
:   gray
```

## 情報ウィンドウと note

情報ウィンドウはスライド実行中に `I` で出てきます。
プロジェクターにスライドを出して、手元に情報ウィンドウを出すという使い方が想定されているようです。
残り時間のタイマーも付いています。

たとえば以下のようにメモを書いておくと情報ウィンドウにメモが表示されて、情報ウィンドウの表示のされ方も変わるようです。

```
## note

メモです。
```

note を使ってみた例としては、昔のスライドなので markup language が RD ですが https://github.com/znz/rubykaigi2014-ruby-removed-features/blob/master/ruby-removed-features.rab があります。

## 本番

`rake` か `rabbit` コマンドで表示してプレゼンを実施します。

スライドの内容によっては右クリックメニューから `Cache all slides` でキャッシュしておくとページの切り替えがはやくなって良いかもしれません。

クリック、`→`、`f` などで次のページ、`←`、`b` などで前のページです。

表紙の間はまだタイマーは止まっていて、1ページ目に切り替えた瞬間からカウントダウンが始まります。

## スライドの公開

`rake pdf` で PDF ファイルを生成できます。
`pdf/id-base-name.pdf` という形式のファイル名で作成されます。

```
$ rake pdf
mkdir -p pdf
$ ls pdf
about-boot2docker-format-me-boot2docker-format-me.pdf
```

`rake publish:slideshare` はエラーでうまくいかず、 `rake publish:speaker_deck` は speaker deck 側で API が未実装なので、生成された PDF を https://www.slideshare.net/ や https://speakerdeck.com/ でアップロードします。
情報を埋めて puslish します。

公開されたスライドの URL の `/` で区切られた末尾の部分をそれぞれ `config.yaml` の `slideshare_id:` と `speaker_deck_id:` に設定します。

たとえば https://www.slideshare.net/znzjp/boot2docker-formatme と https://speakerdeck.com/znz/boot2docker-false-format-me-falsehua なら

```
slideshare_id: boot2docker-formatme
speaker_deck_id: boot2docker-false-format-me-falsehua
```

になります。

両方に公開してもいいですし、片方だけでも、または後述の [Rabbit Slide Show](https://slide.rabbit-shocker.org/) だけでも良いと思います。

## gem での公開

https://rubygems.org/ に gem としてスライド (のソース) を公開します。
gem を公開して、しばらく待つと [Rabbit Slide Show](https://slide.rabbit-shocker.org/) に同期されてスライドが公開されます。
以下の例だと https://slide.rabbit-shocker.org/authors/znz/about-boot2docker-format-me/ に公開されます。
SlideShare と Speaker Deck へのリンクもここにあります。

gem の公開は `rake publish:rubygems` でできます。
認証情報は `~/.gem/credentials` に保存されるので、2回目以降はパスワードの入力は不要です。

```
$ rake publish:rubygems
mkdir -p pkg
WARNING:  licenses is empty, but is recommended.  Use a license identifier from
http://spdx.org/licenses or 'Nonstandard' for a nonstandard license.
WARNING:  open-ended dependency on rabbit (>= 2.0.2) is not recommended
  if rabbit is semantically versioned, use:
    add_runtime_dependency 'rabbit', '~> 2.0', '>= 2.0.2'
WARNING:  See http://guides.rubygems.org/specification-reference/ for help
  Successfully built RubyGem
  Name: rabbit-slide-znz-about-boot2docker-format-me
  Version: 2018.01.28
  File: rabbit-slide-znz-about-boot2docker-format-me-2018.01.28.gem
mv rabbit-slide-znz-about-boot2docker-format-me-2018.01.28.gem pkg/rabbit-slide-znz-about-boot2docker-format-me-2018.01.28.gem
Enter password on RubyGems.org [znz]: 
/home/vagrant/.anyenv/envs/rbenv/versions/2.4.3/bin/ruby -S gem push pkg/rabbit-slide-znz-about-boot2docker-format-me-2018.01.28.gem --key znz
Pushing gem to https://rubygems.org...
Successfully registered gem: rabbit-slide-znz-about-boot2docker-format-me (2018.01.28)
```

## github.com にソースを公開

github.com に repository を作成して push します。

```
%  hub create
Updating origin
created repository: znz/about-boot2docker-format-me
%  git push -u origin master
Counting objects: 17, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (15/15), done.
Writing objects: 100% (17/17), 3.82 KiB | 1.27 MiB/s, done.
Total 17 (delta 6), reused 0 (delta 0)
remote: Resolving deltas: 100% (6/6), done.
To github.com:znz/about-boot2docker-format-me.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
```

修正予定がなければ必要ないかもしれませんが、タグも `rake tag` で作成しておくと発表時点のものが記録できて良いと思います。

```
$ rake tag
git tag -a 2018.01.28 -m Publish 2018.01.28
git push --tags
Counting objects: 1, done.
Writing objects: 100% (1/1), 177 bytes | 0 bytes/s, done.
Total 1 (delta 0), reused 0 (delta 0)
To git@github.com:znz/about-boot2docker-format-me.git
 * [new tag]         2018.01.28 -> 2018.01.28
```

## まとめ

rabbit を使ってのスライドの作成から公開までの流れを紹介しました。

公式サイトの[スライドの作り方](https://rabbit-shocker.org/ja/how-to-make/)は概要しか書いていないので、色々なスライドの書き方は https://slide.rabbit-shocker.org/ で気になったものがあれば、そのソースを取得して参考にしてみると良いと思います。
細かい機能については [news](https://rabbit-shocker.org/ja/news.html) ページが参考になるかもしれません。
