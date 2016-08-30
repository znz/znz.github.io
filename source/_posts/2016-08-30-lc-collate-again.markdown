---
layout: post
title: "lsで丸数字で始まるファイル名の順番が変だった"
date: 2016-08-30 22:58:34 +0900
comments: true
categories: osx linux debian ubuntu
---
OS X で丸数字から始まるファイル名のファイルが入ったフォルダーを Finder で開くと丸数字の数字順に並んでいたのに、 `ls` で表示すると別の順番になっていたので原因を調べてみました。

<!--more-->

## 動作確認環境

- OS X Yosemite (10.10.5)
- ls, uniq バージョン不明
- sort (GNU coreutils) 5.93

## 動作確認

Unicode 的に連続している 丸1 から 丸20 までのファイル名のファイルと、それに適当な ASCII の数字をつけたファイル名のファイルを作成して `ls` で表示してみました。

```
%  touch $(ruby -e 'puts ("\u{2460}".."\u{2473}").to_a')
%  touch $(ruby -e 'puts ("\u{2460}".."\u{2473}").to_a.shuffle.map.with_index{|e,i|"#{e}#{i}"}')
%  ls
①      ⑤      ⑨      ⑬      ⑰      ⑧0     ⑲12    ⑮16    ⑥2     ⑬6
②      ⑥      ⑩      ⑭      ⑱      ⑤1     ⑱13    ⑳17    ④3     ⑫7
③      ⑦      ⑪      ⑮      ⑲      ⑨10    ②14    ⑪18    ⑭4     ⑦8
④      ⑧      ⑫      ⑯      ⑳      ⑯11    ⑩15    ③19    ⑰5     ①9
```

丸数字のみだと数字順に並んでいるのに、後ろに ASCII の数字をつけた部分は ASCII の数字順に並んでいます。
(numeric sort じゃないので 1,10,2 という順番になっているのがちょっとわかりにくいかもしれませんが。)

## Jessie での動作確認

比較のために Debian GNU/Linux 8.5 (jessie) でも同様のファイルを作成して `ls` してみると丸数字のみのところもバラバラの順番でした。
何度か実行しても同じ結果なので、ランダムというわけではなくなんらかの基準がありそうですが、どういう順番なのかはわかりませんでした。

```
% ls
⑧  ⑯  ⑥  ⑬  ⑫  ⑦  ①  ③  ⑮  ⑰  ④0  ⑨10  ⑳12  ⑫14  ⑭16  ①18  ⑩2  ⑥4  ⑯6  ⑱8
⑱  ⑩  ⑨  ⑪  ⑲  ⑤  ②  ④  ⑭  ⑳  ⑪1  ⑦11  ⑲13  ⑬15  ⑮17  ⑧19  ⑤3  ②5  ⑰7  ③9
```

## LC_COLLATE

[LC_COLLATEの問題でuniqで丸数字が同一視されてしまう](http://blog.n-z.jp/blog/2013-10-31-lc-collate-uniq.html "LC_COLLATEの問題でuniqで丸数字が同一視されてしまう")のと同じ話かと思って、 `sort` や `uniq` も試してみたところ、同じ話のように見えました。
OS X では locale data が GNU/Linux とは違うようで `uniq` で同一視されるということは起きませんでした。

```
% rbenv exec irb -r irb/completion --simple-prompt
>> IO.popen("uniq", "r+"){|io| io.puts ("\u{2460}".."\u{2473}").to_a; io.close_write; puts io.read }
①
②
③
④
⑤
⑥
⑦
⑧
⑨
⑩
⑪
⑫
⑬
⑭
⑮
⑯
⑰
⑱
⑲
⑳
=> nil
>> IO.popen("sort", "r+"){|io| io.puts ("\u{2460}".."\u{2473}").to_a.shuffle; io.close_write; puts io.read }
①
②
③
④
⑤
⑥
⑦
⑧
⑨
⑩
⑪
⑫
⑬
⑭
⑮
⑯
⑰
⑱
⑲
⑳
=> nil
>> IO.popen("sort", "r+"){|io| io.puts ("\u{2460}".."\u{2473}").to_a.shuffle.map.with_index{|e,i|"#{e}#{i}"}; io.close_write; puts io.read }
⑬0
⑥1
⑰10
⑱11
⑤12
⑮13
⑦14
④15
③16
⑪17
⑩18
①19
⑧2
⑲3
⑫4
⑳5
⑭6
②7
⑯8
⑨9
=> nil
>> IO.popen({"LC_COLLATE"=>"C"}, "sort", "r+"){|io| io.puts ("\u{2460}".."\u{2473}").to_a.shuffle.map.with_index{|e,i|"#{e}#{i}"}; io.close_write; puts io.read }
①6
②5
③0
④16
⑤19
⑥18
⑦7
⑧2
⑨8
⑩3
⑪12
⑫15
⑬4
⑭9
⑮14
⑯10
⑰1
⑱17
⑲13
⑳11
=> nil
```

## 一番自然に感じる並び順

ruby の sort での結果は `LC_COLLATE=C` と同じように文字コード順になり、意味自然な並び順に感じました。
`LC_COLLATE=C ls` も同じ並び順でした。

```
>> puts Dir['*'].sort
①
①9
②
②14
③
③19
④
④3
⑤
⑤1
⑥
⑥2
⑦
⑦8
⑧
⑧0
⑨
⑨10
⑩
⑩15
⑪
⑪18
⑫
⑫7
⑬
⑬6
⑭
⑭4
⑮
⑮16
⑯
⑯11
⑰
⑰5
⑱
⑱13
⑲
⑲12
⑳
⑳17
=> nil
```

```
% LC_COLLATE=C ls
①      ③      ⑤      ⑦      ⑨      ⑪      ⑬      ⑮      ⑰      ⑲
①9     ③19    ⑤1     ⑦8     ⑨10    ⑪18    ⑬6     ⑮16    ⑰5     ⑲12
②      ④      ⑥      ⑧      ⑩      ⑫      ⑭      ⑯      ⑱      ⑳
②14    ④3     ⑥2     ⑧0     ⑩15    ⑫7     ⑭4     ⑯11    ⑱13    ⑳17
```

## Finder での並び順

Finder での並び順は `LC_COLLATE=C` での結果と同じかと思いきや、丸1 の後に 丸10 がきて、丸19, 丸2, 丸20, 丸3 のように並んでいたので、独特な感じでした。

```
①
①9
⑩
⑩15
⑪
⑪18
⑫
⑫7
⑬
⑬6
⑭
⑭4
⑮
⑮16
⑯
⑯11
⑰
⑰5
⑱
⑱13
⑲
⑲12
②
②14
⑳
⑳17
③
③19
④
④3
⑤
⑤1
⑥
⑥2
⑦
⑦8
⑧
⑧0
⑨
⑨10
```
