---
layout: post
title: "frozen_string_literalをtrueにしていっている"
date: 2017-01-08 17:35:03 +0900
comments: true
categories: ruby
---
[Ruby の trunk](https://www.ruby-lang.org/ja/documentation/repository-guide/) で最近は `frozen_string_literal` を `true` に変更するというコミットをしていっています。
([Revision 53141](https://svn.ruby-lang.org/cgi-bin/viewvc.cgi?revision=53141&view=revision "Revision 53141") で `frozen_string_literal: false` として magic comment 自体は追加されています。)

その時にちょっと確認に手間取った変更があったので、そのメモです。

<!--more-->

## lib/fileutils.rb

確認に手間取ったのは [Revision 57275](https://svn.ruby-lang.org/cgi-bin/viewvc.cgi?revision=57275&view=revision "Revision 57275") の `lib/fileutils.rb` の変更です。

## freeze の削除

まず最初は `.freeze` を削除した変更です。

`frozen_string_literal: true` にすれば `"文字列リテラル".freeze` の `.freeze` は不要になるはずと思ったのですが、実行してみてちゃんと `frozen?` が `true` になるのかと `--dump=insns` で disasm の結果をみて確認しました。

該当部分の変更は以下の通りです。

```diff
@@ -1432,9 +1432,9 @@ def join(dir, base)
     end

     if File::ALT_SEPARATOR
-      DIRECTORY_TERM = "(?=[/#{Regexp.quote(File::ALT_SEPARATOR)}]|\\z)".freeze
+      DIRECTORY_TERM = "(?=[/#{Regexp.quote(File::ALT_SEPARATOR)}]|\\z)"
     else
-      DIRECTORY_TERM = "(?=/|\\z)".freeze
+      DIRECTORY_TERM = "(?=/|\\z)"
     end
     SYSCASE = File::FNM_SYSCASE.nonzero? ? "-i" : ""
```

文字列補間がある場合は `freeze` の呼び出しが減っていました。

```console
% cat /tmp/a.rb
# frozen_string_literal: true
ALT_SEPARATOR = '\\'
p "(?=[/#{Regexp.quote(ALT_SEPARATOR)}]|\\z)".freeze.frozen?
% cat /tmp/b.rb
# frozen_string_literal: true
ALT_SEPARATOR = '\\'
p "(?=[/#{Regexp.quote(ALT_SEPARATOR)}]|\\z)".frozen?
% ruby /tmp/a.rb
true
% ruby /tmp/b.rb
true
% diff -u <(ruby --dump=insns /tmp/a.rb) <(ruby --dump=insns /tmp/b.rb)
--- /proc/self/fd/11    2017-01-08 17:43:38.532932848 +0900
+++ /proc/self/fd/13    2017-01-08 17:43:38.532932848 +0900
@@ -1,4 +1,4 @@
-== disasm: #<ISeq:<main>@/tmp/a.rb>=====================================
+== disasm: #<ISeq:<main>@/tmp/b.rb>=====================================
 0000 trace            1                                               (   2)
 0002 putobject        "\\"
 0004 putspecialobject 3
@@ -17,7 +17,6 @@
 0031 putobject        "]|\\z)"
 0033 concatstrings    3
 0035 freezestring     nil
-0037 opt_send_without_block <callinfo!mid:freeze, argc:0, ARGS_SIMPLE>, <callcache>
-0040 opt_send_without_block <callinfo!mid:frozen?, argc:0, ARGS_SIMPLE>, <callcache>
-0043 opt_send_without_block <callinfo!mid:p, argc:1, FCALL|ARGS_SIMPLE>, <callcache>
-0046 leave
+0037 opt_send_without_block <callinfo!mid:frozen?, argc:0, ARGS_SIMPLE>, <callcache>
+0040 opt_send_without_block <callinfo!mid:p, argc:1, FCALL|ARGS_SIMPLE>, <callcache>
+0043 leave
```

文字列補間がない場合は `opt_str_freeze` (`freeze` が再定義されていたら呼ぶ) から `putobject` (単純にスタックにプッシュするだけ) に変わっていました。

```console
% cat /tmp/1.rb
# frozen_string_literal: false
p ''.freeze.frozen?
% cat /tmp/2.rb
# frozen_string_literal: true
p ''.freeze.frozen?
% cat /tmp/3.rb
# frozen_string_literal: true
p ''.frozen?
% ruby /tmp/1.rb
true
% ruby /tmp/2.rb
true
% ruby /tmp/3.rb
true
% diff -u <(ruby --dump=insns /tmp/1.rb) <(ruby --dump=insns /tmp/2.rb)
--- /proc/self/fd/11    2017-01-08 17:52:25.569282848 +0900
+++ /proc/self/fd/13    2017-01-08 17:52:25.569282848 +0900
@@ -1,4 +1,4 @@
-== disasm: #<ISeq:<main>@/tmp/1.rb>=====================================
+== disasm: #<ISeq:<main>@/tmp/2.rb>=====================================
 0000 trace            1                                               (   2)
 0002 putself
 0003 opt_str_freeze   ""
% diff -u <(ruby --dump=insns /tmp/1.rb) <(ruby --dump=insns /tmp/3.rb)
--- /proc/self/fd/11    2017-01-08 17:52:27.952090848 +0900
+++ /proc/self/fd/13    2017-01-08 17:52:27.952090848 +0900
@@ -1,7 +1,7 @@
-== disasm: #<ISeq:<main>@/tmp/1.rb>=====================================
+== disasm: #<ISeq:<main>@/tmp/3.rb>=====================================
 0000 trace            1                                               (   2)
 0002 putself
-0003 opt_str_freeze   ""
+0003 putobject        ""
 0005 opt_send_without_block <callinfo!mid:frozen?, argc:0, ARGS_SIMPLE>, <callcache>
 0008 opt_send_without_block <callinfo!mid:p, argc:1, FCALL|ARGS_SIMPLE>, <callcache>
 0011 leave
```

## String.new への変更

同じコミットの `compare_stream` の変更は一通り目視確認していた時には `<<` による破壊的変更ではないので見落としていて、 `make install` を実行したら引っかかったので気づいたのですが、 `File#read` にバッファとして文字列を渡していることによる破壊的変更でした。

該当部分の変更は以下の通りです。

```diff
@@ -735,8 +735,8 @@ def compare_file(a, b)
   #
   def compare_stream(a, b)
     bsize = fu_stream_blksize(a, b)
-    sa = ""
-    sb = ""
+    sa = String.new(capacity: bsize)
+    sb = String.new(capacity: bsize)
     begin
       a.read(bsize, sa)
       b.read(bsize, sb)
```

変更候補として

- `"".dup` にする (文字列のエンコーディングがソースエンコーディング (この場合は UTF-8) になる)
- `String.new` にする (文字列のエンコーディングが ASCII-8BIT になる)

がありましたが、エンコーディングはどちらでも良かったのと、
ここではバッファサイズとして `bsize` バイトが望ましいという情報が別途存在していたので、
ruby 2.4.0 からの新機能の `String.new(capacity: size)` を使うことにしました。

エンコーディングの違いは以下のように確認できます。

```console
% cat /tmp/a.rb
# frozen_string_literal: true
p "".dup.frozen?
p "".dup.encoding
p String.new.frozen?
p String.new.encoding
% ruby /tmp/a.rb
false
#<Encoding:UTF-8>
false
#<Encoding:ASCII-8BIT>
```
