---
layout: post
title: "octopressをアップデートしてisolateを使い始めた"
date: 2013-12-21 22:43:36 +0900
comments: true
categories: octopress git zsh
---
気が向いたので、また octopress の更新をしてみました。
そして `rake isolate` を使い始めました。

記事中では `git fetch --prune` とか、
zsh の `(mm-1)` を使っているので、
カテゴリに `git` と `zsh` も付けています。

<!--more-->

## octopress のアップデート

`git fetch` します。
最近は `--prune` を付けてリモートで削除されたブランチは手元でも削除されるようにしていることが多いので、
今回も付けています。

```console
% git remote -v
octopress	git://github.com/imathis/octopress.git (fetch)
octopress	git@github.com:imathis/octopress.git (push)
origin	git@github.com:znz/znz.github.io (fetch)
origin	git@github.com:znz/znz.github.io (push)
origin	ssh://git@bitbucket.org/znz/blog.n-z.jp.git (push)
% git fetch --prune octopress
remote: Counting objects: 33, done.
remote: Compressing objects: 100% (18/18), done.
remote: Total 24 (delta 16), reused 14 (delta 6)
Unpacking objects: 100% (24/24), done.
From git://github.com/imathis/octopress
 * [new branch]      jekyll-1-3 -> octopress/jekyll-1-3
   64ba603..78defff  master     -> octopress/master
%
```

差分を確認してからマージしました。

```console
% git diff source...octopress/master
% git merge octopress/master
Auto-merging Rakefile
Merge made by the 'recursive' strategy.
 .themes/classic/sass/base/_theme.scss                     |  2 +-
 .themes/classic/source/_includes/archive_post.html        |  2 +-
 .themes/classic/source/_includes/custom/category_feed.xml |  4 +-
 .themes/classic/source/_includes/disqus.html              |  2 +-
 .themes/classic/source/_includes/head.html                |  4 +-
 .themes/classic/source/atom.xml                           |  4 +-
 README.markdown                                           |  2 +-
 Rakefile                                                  |  6 +--
 plugins/category_generator.rb                             |  3 +-
 plugins/code_block.rb                                     |  2 +-
 plugins/date.rb                                           | 53 +++++++++----------------
 plugins/gist_tag.rb                                       |  8 +++-
 plugins/include_code.rb                                   |  2 +-
 plugins/jsfiddle.rb                                       |  2 +-
 14 files changed, 41 insertions(+), 55 deletions(-)
```


themes も更新されているので、
[Updating Octopress](http://octopress.org/docs/updating/)
の手順に従ってアップデートしました。

```console
 % bundle install
 Using rake (0.9.2.2)
 Using RedCloth (4.2.9)
 Using chunky_png (1.2.5)
 Using fast-stemmer (1.0.1)
 Using classifier (1.3.3)
 Using fssm (0.2.9)
 Using sass (3.2.9)
 Using compass (0.12.2)
 Using directory_watcher (1.4.1)
 Using haml (3.1.7)
 Using kramdown (0.13.8)
 Using liquid (2.3.0)
 Using syntax (1.0.0)
 Using maruku (0.6.1)
 Using posix-spawn (0.3.6)
 Using yajl-ruby (1.1.0)
 Using pygments.rb (0.3.4)
 Using jekyll (0.12.0)
 Using rack (1.5.2)
 Using rack-protection (1.5.0)
 Using rb-fsevent (0.9.1)
 Using rdiscount (2.0.7.3)
 Using rubypants (0.2.0)
 Using sass-globbing (1.0.0)
 Using tilt (1.3.7)
 Using sinatra (1.4.2)
 Using stringex (1.4.0)
 Using bundler (1.3.5)
 Your bundle is complete!
 Use `bundle show [gemname]` to see where a bundled gem is installed.
 % bundle exec rake update_source
 mkdir source.old
 cp -r source/. source.old
 ## Copied source into source.old/
 cp -r --remove-destination .themes/classic/source/. source
 cp -r --remove-destination source.old/_includes/custom/. source/_includes/custom/
 cp source.old/favicon.png source
 ## Updated source ##
 % bundle exec rake update_style
 mv sass sass.old
 ## Moved styles into sass.old/
 cp -r .themes/classic/sass/ sass
 cp -r sass/custom/. sass.old/custom
 ## Updated Sass ##
 %
```

`git diff` で確認するとローカルでの変更が消えてしまっていたので、
`git checkout ファイルパス` で戻したり、
再適用したりしてからコミットしました。

## rake isolate

[Octopressでチェック用に高速にgenerate/previewする](http://rcmdnk.github.io/blog/2013/10/07/blog-octopress/)
で紹介されていた
`rake isolate[filename]`
を使い始めてみました。

### ファイル名の指定を省略する

最終更新時刻でソートして最新だけ、という絞り込みをしたかったのですが、
zsh の機能だけでうまく表現できなかったので、
[zsh で find を使わずに簡単にファイルを絞り込む](http://qiita.com/mollifier/items/1c4a4930a89aa75e5ced)
を参考にして1分以内に変更されたファイルだけという絞り込みで妥協することにしました。

```console
 bundle exec rake "isolate[$(echo source/_posts/*.markdown(mm-1))]"
 bundle exec rake preview
 bundle exec rake integrate
```

### rake integrate 忘れを防ぐ

isolate した状態で deploy する事故を防ぐために
[check stash dir before deploy](https://github.com/imathis/octopress/pull/1444)
という変更を入れてから使うようにしてみました。

Rakefile をみてみると

```ruby Rakefile
desc "Generate website and deploy"
task :gen_deploy => [:integrate, :generate, :deploy] do
end
```

となっているので、
`bundle exec rake deploy` ではなく
`bundle exec rake gen_deploy` を使うようにしていれば
大丈夫そうですが、
念のためチェックした方が安全だと思いました。

octopress の fork で作成したこの変更は

```console
git pull git@github.com:znz/octopress.git check-stash-before-deploy
```

で手元の source ブランチに取り込みました。
