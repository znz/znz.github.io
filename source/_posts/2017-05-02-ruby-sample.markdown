---
layout: post
title: "rubyのsample/cbreak.rbをSolarisで試した"
date: 2017-05-02 21:23:14 +0900
comments: true
categories: ruby
---
ruby の [sample/cbreak.rb](https://github.com/ruby/ruby/blob/3692fd69ca10fb921db5cc74a6da5eaa66808f38/sample/cbreak.rb) は Linux で試しても動かなくて、ファイルの頭に `# ioctl example works on Sun` と書いてあったので、
Solaris で試してみました。

<!--more-->

## vagrant box 探し

「vagrant solaris」で検索して
https://vagrantcloud.com/boxes/search?q=solaris&utf8=%E2%9C%93
の中から
[plaurin/solaris-11_3](https://vagrantcloud.com/plaurin/boxes/solaris-11_3)
が比較的新しくて良さそうかなと思って使うことにしました。

## VM 作成

適当なディレクトリを作成して `vagrant init` をしました。

    mkdir solaris-11_3
	cd solaris-11_3
    vagrant init plaurin/solaris-11_3

説明ページに書いてあったので `vi Vagrantfile` で

	config.ssh.password = "1vagrant"

を追加しました。

    vagrant up

してダウンロードなどを待ちます。

後は

    vagrant ssh

で入って VM の中で作業しました。

## git インストール

とりあえず git をインストールすることにしました。

https://git-scm.com/download/linux に書いてあったように

    sudo pkg install developer/versioning/git

でインストールできました。

## autoconf インストール

必要になるのがわかっているので autoconf もインストールしました。
パッケージ名は適当に指定してみたらインストールできました。

    sudo pkg install autoconf

## git clone

履歴はなくても良いので、 `--depth 1` で最新だけとってきました。

    git clone --depth 1 https://github.com/ruby/ruby

## とりあえず configure

とりあえず configure まで実行するとエラーになりました。

    vagrant@solaris:~$ cd ruby
    vagrant@solaris:~/ruby$ autoconf
    vagrant@solaris:~/ruby$ mkdir build
    vagrant@solaris:~/ruby$ cd build
    vagrant@solaris:~/ruby/build$ ../configure --prefix=$HOME/opt/ruby
    checking for ruby... false
    configure: error: cannot run /bin/sh ../tool/config.sub

## baseruby をインストール

リリースされたアーカイブではないので、baseruby となる ruby が必要ということで ruby をインストールしてやり直しました。

    vagrant@solaris:~/ruby/build$ sudo pkg install ruby
	(略)
	vagrant@solaris:~/ruby/build$ ../configure --prefix=$HOME/opt/ruby
    checking for ruby... /usr/bin/ruby
    downloading config.guess ... done
    downloading config.sub ... done
    checking build system type... i386-pc-solaris2.11
    checking host system type... i386-pc-solaris2.11
    checking target system type... i386-pc-solaris2.11
    checking for gcc... no
    checking for cc... no
    checking for cl.exe... no
    configure: error: in `/export/home/vagrant/ruby/build':
    configure: error: no acceptable C compiler found in $PATH
    See `config.log' for more details

## gcc をインストール

C compiler がなくて、何を入れればいいのかよくわからなかったので、とりあえず gcc を指定してみたら入りました。

    vagrant@solaris:~/ruby/build$ sudo pkg install gcc
    (略)
	vagrant@solaris:~/ruby/build$ ../configure --prefix=$HOME/opt/ruby
    (略)
    ---
    Configuration summary for ruby version 2.5.0
    
    head: illegal option -- c
    usage: head [-n #] [-#] [filename...]
    /export/home/vagrant/opt/ruby
    head: illegal option -- c
    usage: head [-n #] [-#] [filename...]
    ${prefix}
    head: illegal option -- c
    (略)
	head: illegal option -- c
    usage: head [-n #] [-#] [filename...]
    yes
    head: illegal option -- c
    usage: head [-n #] [-#] [filename...]
    man
    
    ---

configure 自体は問題がなかったようですが、サマリーの表示部分でエラーになりました。

## head -c を書き換え

head -c は [POSIX](http://pubs.opengroup.org/onlinepubs/9699919799/utilities/head.html) になくて、代わりの手段を探してみたところ、[どの環境でも使えるシェルスクリプトを書くためのメモ ver4.51 - Qiita の headコマンド](http://qiita.com/richmikan@github/items/bd4b21cf1fe503ab2e5c#head%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89) にあったので `head -c26` を `dd bs=1 count=26 2>/dev/null` に書き換えました。

    vagrant@solaris:~/ruby/build$ cd ..
    vagrant@solaris:~/ruby$ vi configure.in
	vagrant@solaris:~/ruby$ autoconf
	vagrant@solaris:~/ruby$ cd build
	vagrant@solaris:~/ruby/build$ ../configure --prefix=$HOME/opt/ruby
	(略)
    vagrant@solaris:~/ruby/build$ make
	(略)
    generating parse.c
    sh: line 1: bison: not found
    *** Error code 127
    The following command caused the error:
    bison -d  -o y.tab.c parse.tmp.y
    make: Fatal error: Command failed for target `parse.c'

## bison インストール

リリースされたアーカイブだと不要なので configure ではチェックされない bison が必要だったのでインストールしました。

    vagrant@solaris:~/ruby/build$ sudo pkg install bison
    (略)
    vagrant@solaris:~/ruby/build$ make
    (略)
     compiling ../process.c
     ../process.c: In function ‘retry_fork_async_signal_safe’:
     ../process.c:3573:9: error: ‘fork’ is deprecated (declared at /usr/include/unistd.h:301) [-Werror=deprecated-declarations]
              pid = fork();
              ^
     ../process.c: In function ‘retry_fork_ruby’:
     ../process.c:3638:9: error: ‘fork’ is deprecated (declared at /usr/include/unistd.h:301) [-Werror=deprecated-declarations]
              pid = fork();
              ^
     ../process.c: At top level:
     cc1: warning: unrecognized command line option "-Wno-self-assign" [enabled by default]
     cc1: warning: unrecognized command line option "-Wno-constant-logical-operand" [enabled by default]
     cc1: warning: unrecognized command line option "-Wno-parentheses-equality" [enabled by default]
     cc1: warning: unrecognized command line option "-Wno-tautological-compare" [enabled by default]
     cc1: some warnings being treated as errors
     *** Error code 1
     The following command caused the error:
     gcc -O3 -fno-fast-math -ggdb3 -Wall -Wextra -Wno-unused-parameter -Wno-parentheses -Wno-long-long -Wno-missing-field-initializers -Wno-tautological-compare -Wno-parentheses-equality -Wno-constant-logical-operand -Wno-self-assign -Wunused-variable -Werror=implicit-int -Werror=pointer-arith -Werror=write-strings -Werror=declaration-after-statement -Werror=implicit-function-declaration -Werror=deprecated-declarations -Wno-packed-bitfield-compat -Wsuggest-attribute=noreturn -Wsuggest-attribute=format -std=gnu99  -D_FORTIFY_SOURCE=2 -fstack-protector -fno-strict-overflow -fvisibility=hidden -fexcess-precision=standard -DRUBY_EXPORT -fPIE   -I. -I.ext/include/i386-solaris2.11 -I../include -I.. -I../enc/unicode/9.0.0 -o process.o -c ../process.c
     make: Fatal error: Command failed for target `process.o'

`fork` が deprecated だということでエラーになりました。

## `-Werror=deprecated-declarations` 削除

とりあえず今回試したい件とは関係ないので `warnflags` から `-Werror=deprecated-declarations` を削除しました。

    vagrant@solaris:~/ruby/build$ sed -e 's/-Werror=deprecated-declarations//' Makefile > a
    vagrant@solaris:~/ruby/build$ mv a Makefile
	vagrant@solaris:~/ruby/build$ make
    (略)
	vagrant@solaris:~/ruby/build$ make install
    (略)

## sample/cbreak.rb の動作確認

動かしてみたら `STDIN.ioctl(TIOCGETP, tty)` で `Errno::EINVAL` になりました。

    vagrant@solaris:~/ruby/build$ export PATH=$HOME/opt/ruby/bin:$PATH
    vagrant@solaris:~/ruby/build$ ruby -v
    ruby 2.5.0dev (2017-05-02 trunk 58541) [i386-solaris2.11]
    vagrant@solaris:~/ruby/build$ ruby ../sample/cbreak.rb
            from ../sample/cbreak.rb:30:in `<main>'
            from ../sample/cbreak.rb:9:in `cbreak'
            from ../sample/cbreak.rb:18:in `set_cbreak'
    ../sample/cbreak.rb:18:in `ioctl': Invalid argument @ rb_ioctl - <STDIN> (Errno::EINVAL)

## TIOCGETP の確認

    vagrant@solaris:~/ruby/build$ find /usr/include -name '*.h' | xargs grep TIOCGETP
    /usr/include/sgtty.h:#define    TIOCGETP        (('t'<<8)|8)
    /usr/include/sys/mtio.h:#define MTIOCGETPOS             (MTIOC|17)      /* Get drive position */
    /usr/include/sys/termios.h:#define      TIOCGETP        (tIOC|8)
    /usr/include/sys/ttold.h: * Structure for TIOCGETP and TIOCSETP ioctls.
    /usr/include/sys/ttold.h:#define        TIOCGETP        (tIOC|8)
    vagrant@solaris:~/ruby/build$ vi a.c
    vagrant@solaris:~/ruby/build$ gcc a.c
    vagrant@solaris:~/ruby/build$ ./a.out
    7408
    vagrant@solaris:~/ruby/build$ cat a.c
    #include <sgtty.h>
    #include <stdio.h>
    
    int main() {
            printf("%x\n", TIOCGETP);
            return 0;
    }

include するのが `sys/termios.h` でも `sys/ttold.h` でも 7408 でした。

`sample/cbreak.rb` では `TIOCGETP = 0x40067408` なので、何か違うようです。

## macOS で確認

そういえば macOS だとどうだろうと思って試してみたら、動いてしまいました。

ただし `readline().print` のところで ``sample/cbreak.rb:33:in `<main>': private method `print' called for "hoge\n":String (NoMethodError)`` でこけたので、直す必要がありました。

## まとめ

ruby の sample はリリースに含まれていても、そのバージョンで動作確認されているとは限らないようです。

`readline().print` は `Kernel#readline` が `String` を返して、その `Kernel#print` を呼んでいるようなので、どのくらい古い ruby だと動くのか、それとも最初から動かなかったのか、よくわかりませんでした。
