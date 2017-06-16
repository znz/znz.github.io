---
layout: post
title: "ansibleでjsonファイルの設定を更新した"
date: 2017-06-16 21:26:00 +0900
comments: true
categories: ansible docker
---
ansible で json ファイル (今回は `/etc/docker/daemon.json`) を更新したかったのですが、 `lineinfile` や `replace` や `ini_file` のように単独のモジュールで簡単にできるものではなかったので、少し工夫をして実現しました。

<!--more-->

## 動作確認バージョン

- ansible 2.3.1.0

combine フィルター (後述) が New in version 2.0 なので、 1.x では動かないと思います。

## 実例

https://github.com/znz/ansible-role-docker や https://github.com/znz/ansible-playbook-gitlab-dokku/tree/master/provision/roles/docker-dns にあります。

## json ファイル読み込み

まずは `command` モジュールで読み込んで、 `set_fact` で変数に設定しておきます。

    - name: "Read daemon.json"
      command: cat /etc/docker/daemon.json
      register: result
      changed_when: no

記事を書いていて気づいたのですが、リモートからファイルを読み込むには `slurp` モジュールというのがあるようですが、作った時点では知らなかったので、使っていません。

base64 でエンコードされていて b64decode を通す必要があるようなので、何度も参照するなら、次の `set_fact` を組み合わせた方が良さそうなのは変わらなさそうです。

## json から dict に変換

json の文字列から [`from_json`フィルター](http://docs.ansible.com/ansible/playbooks_filters.html#filters-for-formatting-data)で変換します。

{% raw %}

    - set_fact: docker_daemon_json="{{ result.stdout | from_json }}"

{% endraw %}

## combine フィルターでマージ

[`combine`フィルター](http://docs.ansible.com/ansible/playbooks_filters.html#combining-hashes-dictionaries)で設定をマージして、 `to_json` フィルターで json 文字列に変換してファイルに書き出します。

今回の用途では、ネストしたデータは考慮する必要がなく、トップレベルのキーが一致するもので置き換えられればよかったので、 `recursive=True` は指定していません。

`when` でのチェックもしておかないと、設定内容としては変わっていないのに、 json 文字列になった時にキーの順番が変わっているのか、 changed になってしまうことがあったので、 `when` で設定内容に変更がある時だけ書き込むようにしています。

    - name: "Update daemon.json"
      template:
        src: daemon.json.j2
        dest: /etc/docker/daemon.json
        owner: root
        group: root
        mode: 0400
      when: "docker_daemon_json != docker_daemon_json|combine({'dns':dns})"
      notify:
      - restart docker

templates/daemon.json.j2:

{% raw %}

    {{ docker_daemon_json | combine({"dns": dns}) | to_json }}

{% endraw %}

## 動作確認

全体ができた後に、

- ファイルがない時の動作
- 設定が変わらない時の動作
- 設定が変わる時の動作

も確認しておきます。

## docker のデーモン設定ファイル

docker の[デーモン設定ファイル](http://docs.docker.jp/engine/reference/commandline/dockerd.html#daemon-configuration-file)は `insecure-registries` (`--insecure-registry=[]`) や `dns` や `bip` など dockerd のオプションで指定できるものはなんでも指定できます。

[docker デーモンのオプション変更](http://docs.docker.jp/engine/admin/systemd.html#custom-docker-daemon-options)は systemd の docker.service の設定を変更する方法もあるようですが、ローカルの変更をローカルの独立ファイルに隔離できて、バージョンアップの影響も少なそうなので、 daemon.json を使うことにしました。
