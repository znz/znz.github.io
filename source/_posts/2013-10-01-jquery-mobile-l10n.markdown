---
layout: post
title: "jQuery Mobile 1.3.2 の日本語化"
date: 2013-10-01 15:00
comments: true
categories: jquery-mobile l10n rails javascript
---
jQuery Mobile の l10n についてちゃんと調べたのでまとめておきます。
今回は日本語化だけですが、
翻訳対象は網羅できていると思います。

ちゃんと他の言語にも対応させるためには
`mobileinit`
の中で設定する文字列を別途 i18n 用の仕組みを用意して
翻訳し分けることになると思います。

<!--more-->

## mobileinit

まず最初に注意すべき点として
[Configuring Defaults | jQuery Mobile API Documentation](http://api.jquerymobile.com/global-config/)
にも書いてありますが、
 `mobileinit`
のイベントハンドラは
`jquery.mobile.js`
の読み込み前に設定しておく必要があります。
(`$` や `on` を使うので `jquery.js` よりは後にします。)

これを知らないと設定が反映されなくて悩むことになります。

rails 3.1 以降との組み合わせなら、例えば
`app/assets/javascripts/mobile.js`
に以下のように書いておいて、
`app/assets/javascripts/mobile/jqm_ja.js`
などにローカライズ用の
`mobileinit`
を書くということになります。

```
//= require jquery
//= require jquery_ujs
//= require_tree ./mobile
//= require jquery.mobile
```

例えば以下のような感じになります。

```coffeescript app/assets/javascripts/mobile/jqm_ja.js.coffee
$(document).on "mobileinit", ->
  #$.mobile.loadingMessageTextVisible = true
  #$.mobile.loadingMessage = "読み込み中..."
  $.mobile.loader.prototype.options.text = "読み込み中です..."
  $.mobile.loader.prototype.options.textVisible = false
  $.mobile.pageLoadErrorMessage = "読み込みに失敗しました。"
  $.mobile.page.prototype.options.backBtnText = "戻る"
  $.mobile.listview.prototype.options.filterPlaceholder = "検索..."
  $.mobile.table.prototype.options.columnBtnText = "列の増減..."
  $.mobile.dialog.prototype.options.closeBtnText =
    $.mobile.selectmenu.prototype.options.closeText = "閉じる"
  $.mobile.collapsible.prototype.options.expandCueText = "クリックで開く"
  $.mobile.collapsible.prototype.options.collapseCueText = "クリックで閉じる"
```

## Error Loading Page

読み込みエラーの時のメッセージは
`$.mobile.pageLoadErrorMessage`
で設定できます。

いろいろな理由で表示されることが多いので、
真っ先に変更したくなるメッセージだと思います。

見た目は
`$.mobile.pageLoadErrorMessageTheme`
で変更できます。

## loading

1.3.2 でしか確認していませんが、
jQuery Mobile 1.2 からは
[Loader Widget](http://api.jquerymobile.com/loader/)
に変わっていて、
`$.mobile.loader.prototype.options.text`
で設定するようになっています。
ドキュメントによると、
メッセージの変更やテーマの変更以外にも
独自の HTML を表示するなど
いろいろとカスタマイズ出来るようです。

`$.mobile.loader.prototype.options.textVisible`
で有効にした時だけ表示されるようなので、
不要なら無視しても良いかもしれません。

deprecated になっていますが、互換性のため、
`$.mobile.loadingMessage`
や
`$.mobile.loadingMessageTextVisible`
もまだ使えるようです。
(
`$.mobile.loader.prototype.options.text`
や
`$.mobile.loader.prototype.options.textVisible`
より優先されるようです。)

## pageloadfailed

ローカライズとはちょっと離れて、
読み込みエラーのときに別処理をする方法を書いておきます。

devise の timeoutable を使っていると
普通の画面遷移をしようとしたときに
401 Unauthorized になることがあります。
そのときに読み込みエラーメッセージだけ出されても困るので、
[pageloadfailed](http://api.jquerymobile.com/pageloadfailed/)
で認証画面に飛ぶためのリダイレクト処理を入れてみました。

`event.preventDefault()`
を呼んでいないので、
`data.deferred`
の
`resolve()`
か
`reject()`
も呼ばず、
リダイレクト前のページの処理はデフォルトのままにしています。

```coffeescript app/assets/javascripts/mobile/pageloadfailed.js.coffee
$(document).on "pageloadfailed", (event, data) ->
  if data.xhr.status == 401
    window.location.href = data.absUrl
```

## Back

`data-role="page"`
の要素に
`data-add-back-btn="true"`
を追加したときに
[Header Widget](http://api.jquerymobile.com/header/)
に出てくる戻るボタンのテキストです。

`$.mobile.page.prototype.options.backBtnText`
で設定できます。

## Filter items...

ここからはページ全体ではなく
Widget ごとの翻訳の話になります。

まず
[Listview Widget](http://api.jquerymobile.com/listview/)
で
`data-filter="true"`
の時に出てくる検索入力欄の placeholder のテキストです。

まとめてデフォルトを変更するには
今までのものと同様に
`mobileinit`
で
`$.mobile.listview.prototype.options.filterPlaceholder`
に設定します。

個別に変更するには
listview
に
`data-filter-placeholder`
属性を設定します。

この後のものも同様に全体のデフォルト設定を
`mobileinit`
で設定したり、
属性で個別に設定したりできます。

## Columns...

[Column-Toggle Table Widget](http://api.jquerymobile.com/table-columntoggle/)
の列を増減させるポップアップを表示させるボタンのテキストです。

`mobileinit`
で
`$.mobile.table.prototype.options.columnBtnText`
を設定するか、
`data-column-btn-text`
属性で設定します。

## Close

[Dialog Widget](http://api.jquerymobile.com/dialog/)
の閉じるボタンのテキストは
`mobileinit`
で
`$.mobile.dialog.prototype.options.closeBtnText`
を設定するか、
`data-close-btn-text`
属性で設定します。

ダイアログの閉じるボタンはデフォルトではアイコンのみでテキストは表示されないのですが、
スクリーンリーダーで読み上げられるので、
アクセシビリティ的には重要と jQuery Mobile のドキュメントには書いてありました。

ドキュメントには書いてありませんが、
jQuery Mobile 1.3.2 のソースをみると
[Selectmenu Widget](http://api.jquerymobile.com/selectmenu/)
の
Multiple selects
の閉じるボタンのテキストとして
`$.mobile.selectmenu.prototype.options.closeText`
も設定しておくと良さそうです。

## clear text

検索入力はデフォルトで、
その他のテキスト入力では
`data-clear-btn=true`
の時に、
何かテキストを入力すると右に出てくるクリアボタンのオプションとして、
`data-clear-btn-text` があります。

これもデフォルトでは表示されず、スクリーンリーダーなどのアクセシビリティ用です。

`$.mobile.textinput.prototype.options.clearBtnText`
でまとめて設定すれば良いと思います。

`$.mobile.textinput.prototype.options.clearSearchButtonText`
という設定も残っていますが、
`deprecating for 1.3...`
とコメントに書いてあるので、
1.3 以降なら
`clearBtnText`
だけ設定しておけば良いと思います。

## click to expand contents, click to collapse contents

最後に
[Collapsible Widget](http://api.jquerymobile.com/collapsible/)
の翻訳です。

これもデフォルトでは表示されず、スクリーンリーダーなどのアクセシビリティ用です。

デフォルトが
`" click to expand contents"`
の方が
`$.mobile.collapsible.prototype.options.expandCueText`
や
`data-expand-cue-text`
で変更できて、
デフォルトが
`" click to collapse contents"`
の方が
`$.mobile.collapsible.prototype.options.collapseCueText`
や
`data-collapse-cue-text`
で変更できます。
