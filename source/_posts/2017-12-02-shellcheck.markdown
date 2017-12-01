---
layout: post
title: "shellcheck を使おう"
date: 2017-12-02 02:02:02 +0900
comments: true
categories: shell
---
シェルスクリプトを書くのなら shellcheck という lint ツールでチェックしましょう、というお話です。
([Qiita の Shell Script Advent Calendar 2017 2日目](https://qiita.com/znz/items/63a3d581e8ed6ff11b8e) に投稿したものと同じ内容です。)

<!--more-->

## 題材

先月[Shell Script Advent Calendar 2016 8日目のプログラマーの君！ 騙されるな！ シェルスクリプトはそう書いちゃ駄目だ！！ という話](https://qiita.com/piroor/items/77233173707a0baa6360)がいいね数ランキングで上位にきていたこともあり、[piroor/tweet.sh](https://github.com/piroor/tweet.sh) を例に使いたいと思います。

## shellcheck の準備

macOS なら `brew install shellcheck` とか、 Debian や Ubuntu なら `sudo apt install shellcheck` とかでインストールできます。
ちょっと試すだけなら https://www.shellcheck.net/ でもできるようです。

## ソースの準備

clone して中に移動します。

- `git clone https://github.com/piroor/tweet.sh`
- `cd tweet.sh`

最新だと直っているものもあるので、2016年の11月末のバージョンにします。

- `git checkout 72c657e15c2cb3ea868d1a4e4061d80d0db6adb7`

detached HEAD になるため、いろいろメッセージがでますが、コミットするのでなければ気にしなくて構いません。
後で `git checkout master` か `git checkout -` で master ブランチに戻れます。

<!--
git checkout '@{2016-12-01}' にしていたらうまくいかなかったので変更したら、以下の文が無関係になってしまった。

個人的には、ブレース展開を避けるために `{}` を含む引数は常にクオートするようにしていますが、この例の場合は展開されないのでクオートしなくても良いようです。
大丈夫かどうか考えるのが面倒なら、そのまま渡したい場合は常にクオートする方が考えるのが楽です。
-->

## チェックしてみる

`shellcheck tweet.sh` で実行してみるとたくさん出ます。

`shellcheck --format=gcc tweet.sh | wc -l` でカウントしてみたところ、ちょうど 100 でした。

多すぎるので多いものはとりあえず無視していきます。

- `shellcheck -e SC2155 tweet.sh`
- `shellcheck -e SC2155,SC2086 tweet.sh`

まだ多いです。

頻出するのを無視していった結果、このくらい残りました。

```
%  shellcheck -e SC2155,SC2086,SC2196,SC2166,SC2164,SC2001,SC1090 tweet.sh

In tweet.sh line 38:
tmp="/tmp/$$"
^-- SC2034: tmp appears unused. Verify it or export it.


In tweet.sh line 257:
    while read -r tweet; do echo "found!: ${tweet}"; done
                                          ^-- SC2154: tweet is referenced but not assigned.


In tweet.sh line 642:
    [ $? != 0 ] && continue;
      ^-- SC2181: Check exit code directly with e.g. 'if mycmd;', not indirectly with $?.


In tweet.sh line 1173:
    if echo "$url" | egrep -i "$URL_REDIRECTORS_MATCHER" 2>&1 >/dev/null
                                                         ^-- SC2069: The order of the 2>&1 and the redirect matters. The 2>&1 has to be last.


In tweet.sh line 1337:
    kill $target_pid 2>&1 > /dev/null
                     ^-- SC2069: The order of the 2>&1 and the redirect matters. The 2>&1 has to be last.


In tweet.sh line 1347:
  trap 'kill_descendants $self_pid; exit 0' HUP INT QUIT KILL TERM
                                                         ^-- SC2173: SIGKILL/SIGSTOP can not be trapped.
```

このうち SC2154 は Usage の中で表示すべき文字がエスケープされずに変数展開されてしまっているという感じで、明らかにバグだったので、[pull request を送って取り込まれています](https://github.com/piroor/tweet.sh/pull/2)。

SC2034 や SC2173 は実害がないので、特に pull request は送りませんでした。

SC2069 は本当に標準出力を捨てて、標準エラー出力を標準出力に出したいだけかもしれないと思って、深追いしませんでした。

他にも正常時には問題なさそうなものだったり、意図的にやっているのかどうかソースコードを深追いしないとわからなさそうなものだったりしたので、詳しくはチェックしていません。

## よくある指摘

SC2155 と SC2086 は、 shellcheck を使っていなかったシェルスクリプトに対して shellcheck を実行するとよく見かける気がします。

```
In tweet.sh line 1330:
  local children=$(ps --no-heading --ppid $target_pid -o pid)
        ^-- SC2155: Declare and assign separately to avoid masking return values.
                                          ^-- SC2086: Double quote to prevent globbing and word splitting.
```

SC2086 は、一般的に変数展開は `""` でくくるべき、という指摘です。

SC2086 は本当に複数引数に展開して欲しい場合には指摘される必要はないので、そういう場合は直前の行に

    # shellcheck disable=SC2086

というコメントを書いておくと次の行だけ警告を抑制できます。

SC2155 はシェルスクリプトをよく書く人でも知らない人が多い内容で、 `export` や `local` などの返り値で `$()` の返り値が隠れてしまって、 `set -e` にしていても中断してくれない、という問題です。

shellcheck は Wiki に説明がしっかり書いてあるので、指摘の番号で検索すると [SC2155 Declare and assign separately to avoid masking return values.](https://github.com/koalaman/shellcheck/wiki/SC2155) という説明が簡単に見つかります。

## まとめ

SC2154 や SC2034 のような変更が重なると発生してしまいそうなバグの指摘から SC2086 や SC2155 のような知らなければやってしまいそうな間違いの指摘まで、幅広くチェックしてくれて、シェルスクリプトの上達の手助けをしてくれるので、シェルスクリプトを書くときは積極的に使うことをおすすめします。
