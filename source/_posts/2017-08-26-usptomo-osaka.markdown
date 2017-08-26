---
layout: post
title: "第30回シェル芸勉強会　大阪サテライト＠さくらインターネット大阪本社に午後から参加しました"
date: 2017-08-26 13:00:00 +0900
comments: true
categories: event
---
[第30回シェル芸勉強会　大阪サテライト＠さくらインターネット大阪本社](https://atnd.org/events/90102)に午後から参加しました。

<!--more-->

## 会場

最近よくいっているグランフロントタワーAのさくらインターネットさんでしたが、今回は初めて部屋の中の方に参加しました。

## 第30回シェル芸勉強会

問題は [【問題のみ】jus共催 第30回危念シェル芸勉強会](https://blog.ueda.tech/?p=10188) にあります。

主に macOS の環境で確認しました。

### Q1

- `awk '/^Keywords:/{print FILENAME ":" $0;nextfile}' posts/*/*.md` としてみた。
- nextfile などを調べていたら時間がかかってしまってあまりちゃんとできず。
- ファイル名部分のパスなどの掃除が必要と気づいてなかったのでできていなかった。
- 解答例では `grep -m 1` を使っていたけど、 macOS の grep (BSD grep) 2.5.1-FreeBSD だと動作が違うようで、最初のファイルで止まってしまって、うまく動かなかった。

macOS Sierra 10.12.6 の例:

```
% grep --version
grep (BSD grep) 2.5.1-FreeBSD
% grep -m 1 : /etc/passwd /etc/group
/etc/passwd:nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false
```

Debian GNU/Linux 9.1 (stretch) の例:

```
$ grep --version
grep (GNU grep) 2.27
Copyright (C) 2016 Free Software Foundation, Inc.
ライセンス GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

作者 Mike Haertel および その他の方々は <http://git.sv.gnu.org/cgit/grep.git/tree/AUTHORS> を参照してください。
$ grep -m 1 : /etc/passwd /etc/group
/etc/passwd:root:x:0:0:root:/root:/bin/bash
/etc/group:root:x:0:
```

### Q2

- `perl -pe 's,(href|src)="(?!http|/)(\./)?,$1="/files/,g' url.html`
- `(?!)` を使いたかったので Perl で。
- 最初に試しに書いたのはゆるすぎていらないところにつきまくっていた。
- 正規表現が緩いと charset の属性まで変換してしまうという罠があった。
- 気にせず置き換えてあとで `/files/http://` などになった場合に戻すという方針の人が多かった。

### Q3

- `< list | awk 'BEGIN{print "...";print "..."};/^\* /{print "<li>" $2 "</li>"};END{print "...";print "..."}'`
- 前後の部分の内容を省略するとこんな感じになった。
- `cat list | pandoc -t html5 -s` して削るという手もありらしい
- Web サーバーにするには list.html の内容だけではなく `HTTP/1.1 200 OK` の行も追加する必要があった

### Q4

- hub コマンドを使ってよかったらしい
- がんばって API をたたくのもありっぽい

### Q5

- `sed -e 's/[ *]//g' -e '1s/^/p /' -e '$!s/$/*/' complex | ruby` はかっこ不足だった。(複素数リテラルが優先されると思ってしまっていた。)
- `sed -e 's/[ *]//g' -e '1s/^/p (/' -e '$!s/$/)*(/' -e '$s/$/)/' complex | ruby` でうまくいった。
- 別解として1行ごとに解釈して inject してみた : `ruby -e 'p ARGF.each_line.map{|s|eval(s.gsub(/(?<!\d)i/,"1i"))}.inject(:*)' complex`

### Q6

- 真面目に計算すると `ruby -e 'a=[1,1];f=->(n){a[n]||=f[n-1]+f[n-2]};1.step{|n|if f[n]==6765;p f[n-4];exit;end}'` になった。
- 自分で計算せずに出そうとすると `curl https://en.wikipedia.org/wiki/Fibonacci_number | grep -B4 6765 | head -n 1 | grep -Eo '[[:digit:]]+'` になった。英語版 Wikipedia を使っているのは URL が短くてわかりやすいからで、日本語版 Wikipedia の URL でも可能でした。
- 最初、curl のダウンロード状況出力の右に隠れてしまって1行見落としていて `grep -B5` にしてしまっていましたが、 `grep -B4` でした。

### Q7

- zsh の組み込みのみで `for n in {00..99}; [[ -z ${(FM)"$(< nums)":#*$n*} ]] && echo $n`
- 解説は [zshでgrepのようなことをする](/blog/2014-12-11-zsh-grep.html) を参照
- シーケンスは `seq -w 0 99` とか `{0..9}{0..9}` とか
- `{00..99}` は bash 3 だとダメそうだった。 (0 1 ... になった)
- `grep -o ..` は `echo 1234 | grep -o ..` で 23 が出てこないのでダメ
- `echo 1234 | sed p | sed '1s/.//' | grep -o ..` のようにする
- 解答例: `cat nums | sed p | sed '1s/.//' | grep -o .. | cat - <(seq -w 0 99) | sort | uniq -u`
- fold -w 2 とかこういう時に使ってみると良さそうなコマンドは色々あるっぽい

### Q8

- 問題 差し替え https://twitter.com/ryuichiueda/status/901330659527897088
- `ruby -e 'ARGF.each_line.map{|s|[(s[2].ord-s[0].ord).abs,s]}.max[1].display' alphabet`
- `z-v` という逆順のところも考慮して abs を入れたけどいらなかったかもしれない。
- 別解で36進数を使ってみた: `ruby -e 'ARGF.each_line.map{|s|[eval(s.gsub(/\w+/){$&.to_i(36)}).abs,s]}.max[1].display' alphabet`
- abs の代わりにソートして引くという方法もあるらしい
- `map` + `max[1]` でシュワルツ変換的なことをしなくても `max_by` でよかったらしい
- 解答例は bash のブレース展開を使っていた
- xxd で変換して計算する解答例も紹介されていた

## 午後の部終了

午後の部まで参加の人はここで帰っていました。

## LT 大会

LT は中継はどうなるんだろうと思っていたら、大阪は大阪で LT をしていました。

### シェル芸思考

- [第30回シェル芸勉強会LT シェル芸思考](https://www.slideshare.net/kunst1080/30lt)
- 問題を解く時にどう考えているのか
- 1: コマンド一撃でオプションを知っていればすぐにできるようなものかどうか考える
- 終わりそうにない場合続きを考える
- 2: 中間データの形式を設計する
- 3: コマンドを組み合わせて解いていく
- メインは 2
- 設計の考え方
- 形について考える
- 例: [【ファン迷惑】「響け！ユーフォニアム」という文字列だけで遊ぶシェル芸人達](https://togetter.com/li/1041621)
- 中間データ1: 11回繰り返す
- 中間データ2: 11文字で折り返す
- 中間データ3: 10文字で切り出す

### FORK 爆弾爆発中のロードアベレージを見る

- https://speakerdeck.com/msr_i386/cgroup
- 前回 SysRq でクラッシュさせてカーネルダンプで見た
- 今回 実行中にみたい
- uptime コマンド, w コマンド, top コマンド, カーネルダンプの解析
- カーネルダンプは1回のみでリアルタイムは無理
- uptime, w は起動できない
- top は反応が止まるので無理
- cgroup で制御: cpuset を 0-2 と memory を 1GB に制限
- root:bash に制限
- forkbomb というグループを用意する
- 仮想4コア, メモリ16GB の VM でデモ
- bash はランダムに kill されるのでデフォルトシェルを zsh にしておいて zsh から bash を起動
- キーボード配列がおかしかったのでメモ帳で入力して貼り付けしたら zsh で実行してしまう事故発生
- VM 再起動待ち
- 今度はちゃんと bash で実行
- メモリ 1GB だと 11700 個ぐらい bash が起動している
- ロードアベレージは 10000 を超えていた
- cgconfig.conf は CentOS 7 では非推奨で systemd 経由で使うようになっている

### 破壊的難読化シェル芸

- 難読化シェル芸
- 置換による難読化はあまりにも弱い
- 難読化シェル芸には新たな武器が必要
- 武器っぽいコマンド gunzip (ガンジップ)
- gun(銃)が弱いわけがない
- gzip -cf
- 入力できない
- xxd を通す
- xxd -r -p | gunzip
- 別のアプローチ
- gunzip の代わりに cut と組み合わせる
- いろいろ探してみつけた例を実行

### AWS API リクエストへの署名

- REST API
- リクエストに access key と secret access key で署名する必要がある
- 署名バージョン4: AWS4-HMAC-SHA256
- 署名のテストスイートも公開されている
- 署名のプロセスをシェル芸で追いかける
- CLI / SDK に存在しない API がある: Amazon RDS / DownloadCompleteDBLogFile
- CLI / SDK にあるのは download-db-log-file-portion のみで分割ダウンロードされる
- 例に使うのは AWS IAM / ListUsers
- 例に使う時刻も固定しておく
- 署名作成手順の説明
