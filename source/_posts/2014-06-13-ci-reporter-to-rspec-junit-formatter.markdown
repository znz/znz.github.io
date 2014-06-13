---
layout: post
title: "ci_reporter から rspec_junit_formatter に移行した"
date: 2014-06-13 22:33:12 +0900
comments: true
categories: ruby rails rspec jenkins
---
rspec 2 から rpsec 3 への移行で `ci_reporter` がうまく動かなくなって困っていたら、
`rspec_junit_formatter` というのを[教えてもらった](https://twitter.com/joker1007/status/477372843601055744)ので、
移行してみました。

<!--more-->

## 対象バージョン

- `ruby` 2.1.2
- `rails` 3.2.18
- `rspec-rails` 3.0.1
- `rspec_junit_formatter` 0.2.0

## Gemfile の変更

`group :test` の `gem 'ci_reporter', require: false` を `gem 'rspec_junit_formatter'` に変更しました。

## jenkins で実行しているシェルスクリプトの変更

`bundle exec rake -f $(bundle show ci_reporter)/stub.rake ci:setup:rspec default SPEC_OPTS=--no-drb COVERAGE=on`
のように実行していたのを

```
rm -rf spec/reports
cat >.rspec <<EOF
--format RspecJunitFormatter
--out spec/reports/rspec.xml
EOF
bundle exec rake COVERAGE=on
```

に変更しました。
`spec/reports` は自動で作成してくれるようで、事前に作成しておく必要はないようです。
`ci_reporter` はタスクの最初の方で `spec/reports` を削除していたので、同様に削除するようにしました。

`COVERAGE=on` は `simplecov`, `simplecov-rcov` を有効にするかどうかのフラグとして使っているので今回の移行とは無関係です。

[RSpec JUnit Formatter の README](https://github.com/sj26/rspec_junit_formatter) に書いてある
`rspec --format RspecJunitFormatter --out rspec.xml`
という実行方法だとテスト用データベースの初期化関連で問題がおきたので、
`rspec` コマンド直接ではなく `rake` コマンドのデフォルトタスクで実行するために、
`.rspec` ファイルを書き換えるようにしました。
`SPEC_OPTS` で指定する方法でも良かったかもしれません。

## Jenkins の設定

`ci_reporter` の設定のときに
`ビルド後の処理` で `JUnitテスト結果の集計` を追加していて、
`テスト結果XML` は `spec/reports/**/*.xml` という設定にしていて、
`RspecJunitFormatter` の出力先の方をそちらにあわせたので、
Jenkins 側の設定は変更していません。

## テスト結果

Jenkins のテスト結果の表示では `spec.controllers` や `spec.models` などがパッケージになっていました。

`spec/support/*.rb` で `shared_examples` を定義していたら、
`spec.support` がパッケージになってしまっているのが気になりましたが、
`ci_reporter` のときはほぼすべてが `(root)` というパッケージになっていたのに比べると
便利になったように思います。

## `RSpec::Core::DeprecationError`

`RSpec.configure do |config|` で `config.raise_errors_for_deprecations!` を実行していると

    .../gems/rspec-core-3.0.1/lib/rspec/core/formatters/deprecation_formatter.rb:186:in `puts': Treating `metadata[:execution_result]` as a hash is deprecated. Use the attributes methods to access the data instead. Called from .../rspec_junit_formatter-0.2.0/lib/rspec_junit_formatter/rspec3.rb:43:in `result_of'. (RSpec::Core::DeprecationError)`

というエラーになったので、
`config.raise_errors_for_deprecations!`
を外しました。

[github の master では直っている](https://github.com/sj26/rspec_junit_formatter/commit/96f0115f7dabbea623f04b60dc23519683f39cfa)ので、次のバージョンがリリースされたら解決しそうです。
