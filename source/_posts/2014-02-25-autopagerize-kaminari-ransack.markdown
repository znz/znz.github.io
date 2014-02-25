---
layout: post
title: "autopagerizeのような動作をkaminariとransackを使った環境で実装した"
date: 2014-02-25 18:58:39 +0900
comments: true
categories: rails ransack
---
autopagerize のような動作をするときに id 重複問題があるので、
[QA@IT で質問](http://qa.atmarkit.co.jp/q/3513)
してみたところ、
今のページに表示されている最後の id で対処するしかなさそうだとわかったので、
そういう方針で実装してみました。

<!--more-->

## スクリプト部分

コア部分は以前に実装したものをそのまま使いました。
i18n 対応などが出来ていませんが、冒頭に書いてある id や class が関連要素になります。

```coffeescript app/assets/javascripts/autonext.js.coffee
jQuery ->
  timerId = null
  loadingId = '#autopager_loading'
  nextId = '#next_for_pager'
  pageElement = '.page_element'
  lastContent = '.page_content:last'

  loadNext = ->
    nextLink = $(nextId)[0]
    if nextLink
      $(loadingId).show()
      $(nextId).remove()
      request = $.get nextLink.href, (data) ->
        $(loadingId).before($(data).find(pageElement)).hide()
        # 必要なら turbolinks の `page:change` のような DOM 変更通知をする
      request.error (xhr, status, error) ->
        if error == "Unauthorized"
          alert("認証の期限切れです。再読み込みしてください。")
        else
          alert("何かエラーです。少し待ってから、再読み込みしてください。")
        $(loadingId).html("<a href='javascript:location.reload()' class='btn btn-block btn-primary'>再読み込み</a>")
  autoNext = ->
    content = $(lastContent)
    if content[0] && content.offset().top < $(document).scrollTop() + $(window).height()
      if $(loadingId).css('display') is 'none'
        loadNext()
    timerId = null
    $(window).on 'scroll.autoNext', autoNextDefer

  autoNextDefer = ->
    $(window).off 'scroll.autoNext'
    timerId = setTimeout autoNext, 1000

  $(window).on 'scroll.autoNext', autoNextDefer
```

## views

view は次のような感じにしました。
`page_entries_info` を `.page_element` の中にするか、外にするかは悩ましいところです。

```haml app/views/posts/index.html.haml
    .page_element
      %small.page-entries-info= page_entries_info @posts
      = render @posts
      = link_to_auto_next_page_with_ransack @q, @posts, t('views.pagination.next'), class: "btn btn-large btn-block btn-success", id: "next_for_pager", params: params
    #autopager_loading(style="display:none")
      = fa_icon "spinner spin"
      = t("loading", default: "Loading...")
```

`kaminari` の `link_to_next_page` を使っている時に問題になったのですが、
`params` をちゃんと渡さないと `paginate @posts` で生成されるリンクと違って
パラメーター不足になってしまうようです。
そのため自作の `link_to_auto_next_page_with_ransack` でも渡すようにしています。

## helper

ここがリンク生成の肝になります。
ソートのキーのうち、今表示されている最後のものを使って、
`updated_at desc` でソートされているなら、
`updated_at_lt` で指定する、ということをしています。
仕組みの都合上、同じ値が複数入ると表示されずに抜け落ちてしまうので、
`id` やタイムスタンプなどのようなカラムに限定する必要がありそうです。

時刻は標準の `to_s` だと開発環境の一気に入力したデータで問題が起きたので、
`strftime` を使って `%N` (ナノ秒) まで入れるようにしました。

```ruby app/helpers/link_to_helper.rb
  def link_to_auto_next_page_with_ransack(search, scope, name, options = {}, &block)
    return if scope.last_page?
    params = options.delete(:params) || {}
    sorts = search.sorts
    if sorts.empty?
      raise "You must set sorts to ransack (e.g. `@q.sorts = 'updated_at desc' if @q.sorts.empty?`)"
    end
    order = sorts.first
    column = order.name
    dir = order.dir == "desc" ? "lt" : "gt"
    value = scope.last[column]
    if value.respond_to?(:strftime) && value.respond_to?(:hour)
      # expect Time, DateTime, ActiveSupport::TimeWithZone
      # and not Date
      value = value.strftime("%Y-%m-%d %H:%M:%S.%N %Z")
    end
    q = params[:q] || {}
    q = q.merge("#{column}_#{dir}" => value)
    params = params.merge(search.context.search_key => q)
    link_to name, params, options.reverse_merge(:rel => 'next'), &block
  end
```

## controller

コントローラーは `kaminari` や `ransack` を普通に使っているだけです。
`@posts = Post` の行は `cancan` の `load_and_authorize_resource` を使っている場合には不要なので、
別の行にしています。

```ruby app/controllers/posts_controller.rb
  def index
    @posts = Post
    @q = @posts.search(params[:q])
    @q.sorts = 'updated_at desc' if @q.sorts.empty?
    @posts = @q.result(distinct: true)
    @posts = @posts.page(params[:page]).per(5)
  end
```

## スタイルシート

読み込み中の部分のスタイルは以下のようにして、画面の最下部の横幅いっぱいに出るようにしています。
エラーの時はこの中身が再読み込みボタンに変わります。
開発環境だと `rails server` を停止した状態で自動読み込みさせれば、すぐに確認できます。

```css app/assets/stylesheets/application.css.scss
#autopager_loading {
  background-color: #000;
  bottom: 0px;
  color: #fff;
  font-size: 12px;
  height: 25px;
  left: 0px;
  opacity: 0.8;
  position: fixed;
  text-align: center;
  width: 100%;
  z-index: 1000;
}
```

## まとめ

autopagerize のような自動読み込みを `kaminari` と `ransack` を使った rails アプリに組み込んだ実装例を紹介しました。
部品だけで全体を示せていないので、もしかしたら書き忘れている部分もあるかもしれませんが、重要な部分は載せていると思いますので、参考にしてみてください。
