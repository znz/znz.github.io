---
layout: post
title: "octopress で ansible の記事を書く時のエスケープ"
date: 2014-05-20 23:25:18 +0900
comments: true
categories: octopress ansible
---
octopress で ansible の記事を書く時に連続する `{` のエスケープに困ったので、
対処方法を調べました。

<!--more-->

## コードブロックのエスケープ

[How to escape {{ "{{" }} in markdown on Octopress? - Stack Overflow](http://stackoverflow.com/questions/15786144/how-to-escape-in-markdown-on-octopress)
にあるように

    ```種類 ファイル名
    コードブロック
	```

の外側を
`{{ "{%" }} raw %}` と `{{ "{%" }} endraw %}`
でくくればうまくいきました。

## 文中の {{ "{{" }} のエスケープ

先ほどのリンクのアンカーやこの節の見出しのような文の途中は

{% raw %}
    {{ "{{" }}
{% endraw %}

のように書けばエスケープできました。

幅なしスペース (`&#x200B;`) のような幅のない文字を挟んでごまかすという手もあるかもしれません。

## 文中の `{{ "{%" }}` のエスケープ

`{{ "{%" }} raw %}` と `{{ "{%" }} endraw %}` 自体をエスケープするのに

{% raw %}
    {{ "{%" }}
{% endraw %}

を使って

{% raw %}
    `{{ "{%" }} raw %}` と `{{ "{%" }} endraw %}`
{% endraw %}

のように書きました。

`%}` まで `""` の中に入れようとして

{% raw %}
    `{{ "{% raw %}" }}`
{% endraw %}

と書くと

{% raw %}
    Liquid Exception: Variable '{{ "{% raw %}' was not properly terminated with regexp: /\}\}/
{% endraw %}

というエラーになってうまくいきませんでした。
