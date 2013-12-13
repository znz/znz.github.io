---
layout: post
title: "upstart-socket-bridgeとrubyを組み合わせる"
date: 2013-12-14 00:45
comments: true
categories: ubuntu upstart ruby
---
[Ubuntu Weekly Topics 2013年12月6日号](http://gihyo.jp/admin/clip/01/ubuntu-topics/201312/06)
の「その他のニュース」で紹介されていた
「upstart-socket-bridgeをxinetdライクなソケット待ち受け管理機構として扱う
[アプリケーションの作り方](http://cheesehead-techblog.blogspot.jp/2013/12/upstart-socket-bridge.html)
。」
が python3 で書かれていて、
同じことが ruby で実装できるのか気になったので、
IRC でちょっと助言を受けつつ移植してみました。

<!--more-->

## 試した環境

- amd64 の Ubuntu 13.10 (saucy)
- `/usr/bin/ruby` は `ruby 1.9.3p194 (2012-04-20 revision 35410) [x86_64-linux]`

## ruby で試す

必須なのはソケット周りだけですが、
デバッグ用のメッセージをファイルに残すようにして
動作確認していたので、
その部分も残しています。

```ruby /tmp/test-service.rb
 #!/usr/bin/ruby
 require 'socket'
 open("/tmp/test.log", "w") do |f|
   ENV.each do |key, value|
     f.puts "#{key}=#{value}"
   end
   begin
     serv_socket = Socket.for_fd(ENV["UPSTART_FDS"].to_i)
     client_socket, client_addrinfo = serv_socket.accept
     message = client_socket.recv(1024)
     f.puts message
     client_socket.send("I got your message: #{message}", 0)
     client_socket.close
   rescue Exception => e
     f.puts e.inspect
     f.puts e.backtrace
   end
   f.puts "finished"
 end
```

バグとしては
`for_fd` の引数の `to_i` を忘れていたり、
`send` の引数が足りなかったりしました。

```text /etc/init/socket-test.conf
 description "upstart-socket-bridge test"
 start on socket PROTO=inet PORT=34567 ADDR=127.0.0.1  # 34567 番ポートで待ち受け
 setuid exampleuser                                    # root ではなく exampleuser で動作
 exec /usr/bin/ruby /tmp/test-service.rb               # サービス起動
```

`nc` コマンドで接続して動作確認します。

```console
 $ nc localhost 34567
 Hello Ruby
 I got your message: Hello Ruby
```

最後にテストで作成したファイルを削除しておきます。

```console
 $ sudo rm /etc/init/socket-test.conf  # ブリッジとの接続解除
 $ rm /tmp/test-service.rb             # テストサービス削除
 $ rm /tmp/test.log                    # ログファイル削除
```

## まとめ

python3 の `socket.fromfd` に相当するのは
ruby だと `BasicSocket.for_fd` で、
`BasicSocket` クラスには `accept` がないので、
`BasicSocket` クラスを継承している `Socket` クラスの
`Socket#for_fd` を使いました。

`inetd` や `xinetd` だと標準入出力にソケットをつないでくれて、
サービスは簡単にかけるのに、
`upstart` だと `accept` して `accept` から返ってきたソケットを
`close` するまでがサービス側でやらないといけないようになっていて、
ちょっと面倒に感じました。

たまたま目についたプログラムを移植してみただけで、
深追いはしていないのですが、
どういう利点があるものなのか、
調べてみるのも良いのかもしれません。
