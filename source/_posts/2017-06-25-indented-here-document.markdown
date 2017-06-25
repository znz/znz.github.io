---
layout: post
title: "Ruby と Perl 5.26 の &lt;&lt;~ の挙動の違い"
date: 2017-06-25 22:22:22 +0900
comments: true
categories: ruby perl
---
Ruby には 2.3.0 から入っていた indented here document が[「Perl 5.26」リリース、「@INC」の仕様が変更される | OSDN Magazine](https://mag.osdn.jp/17/06/02/161500) によると、Perl 5.26 にも入ったようなのですが、挙動が若干違うようなので、調べてみました。
(最初に調べたのは OSDN Magazine の記事をみた直後だったのですが、調べた結果を残し忘れていたので、今日調べなおしたものになります。)

<!--more-->

## 動作確認環境

Perl の最新を試すために archlinux を使ってみました。

- OS は vagrant で [terrywang/archlinux](https://atlas.hashicorp.com/terrywang/boxes/archlinux) の box を使って `sudo pacman -Syu` した環境
- ruby 2.4.1p111 (2017-03-22 revision 58053) [x86_64-linux]
- This is perl 5, version 26, subversion 0 (v5.26.0) built for x86_64-linux-thread-multi

## Ruby での基本動作

Ruby では基本的に indented here document の中で一番インデントが浅いところを基準に削られます。

Perl では `<<` の代わりに `<<~` を使っただけでは削られません。

```
vagrant@archlinux:~$ cat /tmp/a
print <<~END;
    4
 1
  2
END
vagrant@archlinux:~$ ruby /tmp/a
   4
1
 2
vagrant@archlinux:~$ perl /tmp/a
    4
 1
  2
```

## Perl での基本動作

Ruby では `<<-` と同様に閉じる識別子のインデントは意味を持ちませんが、
Perl では閉じる識別子のインデント分が削られます。

```
vagrant@archlinux:~$ cat /tmp/b
print <<~END;
   3
    4
  2
 END
vagrant@archlinux:~$ ruby /tmp/b
 3
  4
2
vagrant@archlinux:~$ perl /tmp/b
  3
   4
 2
```

## Perl でのエラー例

Perl では閉じる識別子のインデントより浅い部分があるとエラーになります。

```
vagrant@archlinux:~$ cat /tmp/c
print <<~END;
 1
    4
  2
  END
vagrant@archlinux:~$ ruby /tmp/c
1
   4
 2
vagrant@archlinux:~$ perl /tmp/c
Indentation on line 1 of here-doc doesn't match delimiter at /tmp/c line 1.
```

## Ruby で一番浅い行頭に空白を入れたい場合

Ruby では一番浅い行頭に空白を入れたい場合はエスケープする必要があります。

Perl では最初の2例のように閉じる識別子のインデントの方を浅くするだけです。

```
vagrant@archlinux:~$ cat /tmp/d
print <<~END;
 \ x
    4
  2
 END
vagrant@archlinux:~$ ruby /tmp/d
 x
   4
 2
vagrant@archlinux:~$ perl /tmp/d
 x
   4
 2
```

## 感想

- Ruby の仕様の方が内容を開始の行と終了の行よりインデントしたい時には都合が良さそうと感じました。
- Perl の仕様の方がパーサーは単純になって速そうという印象を受けました。 (実際に速いかどうかは調べていません。)
- Perl の仕様の方が行頭にある程度の空白を残したい場合は都合が良さそうと感じました。
