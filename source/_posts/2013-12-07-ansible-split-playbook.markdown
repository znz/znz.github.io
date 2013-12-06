---
layout: post
title: "ansibleで1ファイルのplaybookからroleに書き換える方法"
date: 2013-12-07 01:30
comments: true
categories: ansible
---
`ansible` を使い始める時は1個の YAML ファイルから始めるのが手軽で良いのですが、
それを
[Best Practices](http://www.ansibleworks.com/docs/playbooks_best_practices.html)
にあわせて `roles` にしようとしたら、
対応がよくわからなくて困ったことがあったので、
書き換え方をまとめてみました。

<!--more-->

全体的に動作確認せずに以前使ったファイルを参考にしているので、
間違っている部分があるかもしれないので、その時はコメントなどで
教えてください。

## 1個の YAML ファイルの例

仮にこんな感じの `playbook.yml` があって、
これを `roles` にしたいとします。

```yaml playbook.yml
---
- hosts: foo-servers
  sudo: yes
  vars:
    adminuser: vagrant
    admingroup: vagrant
    homedir: /home/vagrant
  tasks:
  - apt: pkg=zsh
  - template: src=zshrc dest={{ homedir }}/.zshrc owner={{ adminuser }} group={{ admingroup }} mode=0644
```

この例だとテンプレートファイルとして `zshrc` ファイルが同じディレクトリにあります。

## role 作成

ここでは role の名前を `zsh` にしてみます。

元の `playbook.yml` と `zshrc` から

- `roles/zsh/tasks/main.yml`
- `roles/zsh/templates/zshrc.j2`
- `roles/zsh/vars/main.yml`

というファイルを作ることになります。

### roles/zsh/tasks/main.yml

処理のメイン部分です。
元の `playbook.yml` の `tasks` の値に並んでいた配列を書きます。

```yaml roles/zsh/tasks/main.yml
---
- apt: pkg=zsh
- template: src=zshrc dest={{ homedir }}/.zshrc owner={{ adminuser }} group={{ admingroup }} mode=0644
```

### roles/zsh/templates/zshrc.j2

元の `zshrc` にテンプレート言語がわかりやすいように拡張子を付けただけです。
例として `zshrc` にしてしまっただけなので、
本来は `vars` の変数を展開したいファイルを使うことになると思います。

### roles/zsh/vars/main.yml

変数の定義部分です。
元の `playbook.yml` の `vars` の値に並んでいたハッシュを書きます。

```yaml roles/zsh/vars/main.yml
---
adminuser: vagrant
admingroup: vagrant
homedir: /home/vagrant
```

### その他

`roles/zsh/handleres/main.yml` のような他のディレクトリとか、
`main.yml` 以外の `*.yml` ファイルとかも使えますが、
一番最初の段階では知らなくても大丈夫だと思います。

## 読み込み側 YAML

`vars` とか `tasks` を並べていた代わりに `roles` を使って読み込みます。
今回は `role: zsh` で読み込みましたが、他の例をみてみると `- zsh` だけでも良さそうです。

```yaml site.yml
---
- hosts: foo-servers
  sudo: yes
  roles:
    - role: zsh
```

## まとめ

最初は 1ファイルの YAML と roles 用の複数の YAML ファイルの対応関係がよくわからなくて、
roles 用の YAML ファイルに余計な記述をしてここでは使えないと言われたりして悩んだので、
簡単に対応関係をまとめてみました。

まだあまり使っていなくて、間違っているところもあるかもしれないので、
何かあればコメントなどで指摘してもらえるとありがたいです。
