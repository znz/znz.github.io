---
layout: post
title: "ansible使いのためのYAML入門"
date: 2014-06-21 20:39:38 +0900
comments: true
categories: ansible yaml
---
ansible の設定ファイルは YAML なのですが、
ansible の説明では YAML 由来のいろんな書き方についてはあまり説明がないので、
ansible の設定ファイルを書く時に知っておくと便利な YAML の知識についてまとめてみました。

<!--more-->

## YAML のバージョン

[The Official YAML Web Site](http://yaml.org/ "The Official YAML Web Site")
によると YAML の仕様としては
YAML 1.2 (3rd Edition)
まで存在するようですが、
ansible は PyYaml を使っていて、
libyaml binding なので
対応している YAML のバージョンは
[YAML 1.1 (2nd Edition)](http://yaml.org/spec/1.1/)
になります。

## コレクション

参考: [2.1. Collections](http://yaml.org/spec/1.1/#id857181 "2.1. Collections")

### 配列

`- ` で列挙すると配列になります。
YAML は空白に敏感なので、
`-foo` のように書くとエラーになります。

```yaml
    - foo
	- bar
```

### マッピング

プログラミング言語によってはハッシュとか連想配列とか言われているデータ構造です。
キーと値を `: ` (コロンとスペース) で区切ります。

読みやすくするために `:` の前後の空白を増やしても良いようです。

```yaml
    foo:  bar
    hoge: fuga
    bar : baz
```

### マッピングと配列

マッピングの値に配列を入れるとき

```yaml
    roles:
      - foo
      - bar
```

のようにインデントするのが普通のようですが、

```yaml
    roles:
    - foo
    - bar
```

のようにインデントせずに配列を書いても大丈夫です。

インデントを浅くできるので、個人的にはこの書き方を多用しています。

### フロースタイル

フロースタイルというコンパクトな書き方も出来ます。
ansible の設定ファイルでは見覚えがないのですが、
知っておくと見た時に悩まなくて済むと思います。

[2.1. Collections](http://yaml.org/spec/1.1/#id857181 "2.1. Collections")
の `Example 2.5. Sequence of Sequences` や `Example 2.6. Mapping of Mappings` を参考にしてみてください。

## 構造

参考: [2.2. Structures](http://yaml.org/spec/1.1/#id857577 "2.2. Structures")

### ファイル冒頭の `---`

YAML ファイルの頭に `---` が入っていることがありますが、
これは複数 YAML ドキュメントを YAML ストリームとしてまとめるときに
YAML ドキュメントの開始部分を示すのに使うようです。

ansible などの用途では YAML ストリームは使わないので、
あってもなくても良いと思います。

迷うなら付けておけば YAML ファイルということがちょっとわかりやすくなると思います。

```console
    % cat example.yml
    ---
    foo: bar
    ---
    hoge: fuga
    % ruby -r yaml -r pp -e 'pp YAML.load(ARGF)' example.yml
    {"foo"=>"bar"}
    % ruby -r yaml -r pp -e 'pp YAML.load_stream(ARGF)' example.yml
    [{"foo"=>"bar"}, {"hoge"=>"fuga"}]
```

### コメント

`#` から行末までがコメントです。
行頭以外からでもコメントになります。

```yaml
    foo: # ここはコメント
      - bar
    hoge:
      # ここもコメント
      - fuga
```

## スカラー値

参考: [2.3. Scalars](http://yaml.org/spec/1.1/#id858081 "2.3. Scalars")

### リテラル形式と折り返し形式

`|` を使うと literal style で改行をそのまま使った文字列を指定できます。
ansible の設定では shell と組み合わせに便利です。

https://github.com/znz/ansible-role-rbenv/blob/7fceb7b4f5155ba385d88cdcaf436ecd84238aca/tasks/rbenv.yml
では以下のように `creates=` によるガード条件を最初の行に書いて、
次の行からシェルスクリプトの内容を書いています。

```yaml
    - name: rbenv install {{ rbenv_ruby_version }}
      shell: |
       creates={{ rbenv_root }}/versions/{{ rbenv_ruby_version }}
       set -e
       . /etc/profile.d/rbenv.sh
       export CONFIGURE_OPTS="--disable-install-doc"
       rbenv install {{ rbenv_ruby_version }}
       rbenv global {{ rbenv_ruby_version }}
```

`>` を使うと folded style になります。
literal style と違って改行も普通の空白に置き換えられるので、
ansible の設定ファイルでは使ったことがありません。

上の例からわかるように YAML としては `creates=` も文字列の一部で、
その文字列の一部をガード条件として解釈しているのは ansible の方になります。

## 真偽値

ansible の playbook ファイルで
`sudo: yes` だったり
`sudo: True` だったり
いくつかの書き方がありますが、
すべて YAML の解釈の時点で真偽値になっています。

以下の例では大文字小文字の差が多いので、フロースタイルでまとめています。

```console
    % cat example.yml
    ---
    - [true, True, TRUE]
    - [false, False, FALSE]
    - [yes, Yes, YES]
    - [no, No, NO]
    - [on, On, ON]
    - [off, Off, OFF]
    % ruby -r yaml -r pp -e 'pp YAML.load(ARGF)' example.yml
    [[[true, true, true],
      [false, false, false],
      [true, true, true],
      [false, false, false],
      [true, true, true],
      [false, false, false]]]
```

## 文字列として解釈させる

以上のように元に文字列以外として解釈される可能性があると文字列として指定したい時に困るので、
そういうときは `""` などでくくっておくか、
`!!str` で明示的に文字列型ということを指定します。

```console
    % cat /tmp/example.yml
    ---
    - "true"
    - 'false'
    - !!str yes
    % ruby -r yaml -r pp -e 'pp YAML.load(ARGF)' /tmp/example.yml
    ["true", "false", "yes"]
```

## まとめ

ansible の設定ファイルを書く時に、自分が困ったり悩んだことを中心にまとめてみました。
同じようなことで悩んだり、どうすればいいのか困った時に参考にしてみてください。
