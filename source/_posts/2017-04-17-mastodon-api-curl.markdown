---
layout: post
title: "mastodonのAPIをcurlで試してみた"
date: 2017-04-17 01:24:42 +0900
comments: true
categories: mastodon
---
curl で [mastodon の開発環境](/blog/2017-04-16-contribute-mastodon.html)を使って API を試してみたメモです。

[mastodonのAPIをcurlで試してみた](http://qiita.com/znz/items/c924cd4e9d5e50e50058)と同じ内容です。

## 確認環境

[mastodon の開発環境](/blog/2017-04-16-contribute-mastodon.html)の通りですが、簡単に説明しておくと

- `git clone https://github.com/tootsuite/mastodon`
- `cd mastodon`
- `vagrant up`
- `vagrant ssh`

した後、初期アカウントのメールアドレスを修正した環境を想定しています。

## Create App

クライアント名を決めてスコープを決めて作成します。
クライアント名は同じ名前でも作成できるようです。(Twitter は名前がかぶると作成できなかったはず。)
複数のスコープを指定する時はスペース区切りです。
リダイレクト先がなければ `urn:ietf:wg:oauth:2.0:oob` にしておけば良いようです。
ウェブサイトがあればその URL も設定しておきます。

```bash
vagrant@mastodon:~$ CLIENT_NAME="Test Client"
vagrant@mastodon:~$ curl -X POST -d "client_name=$CLIENT_NAME&redirect_uris=${REDIRECT_URIS:-urn:ietf:wg:oauth:2.0:oob}&scopes=${SCOPES:-read write follow}&website=$WEBSITE" http://localhost:3000/api/v1/apps; echo $?
{"id":16,"redirect_uri":"urn:ietf:wg:oauth:2.0:oob","client_id":"ce06ac1d988d7f26f067dc7ea639e41523ec8135e48cceabb7cd0fd8c1413c65","client_secret":"be04a4f2ca2cd5d9a1cc1314d03a15f811be7f64005483b003447cc3e35e7749"}0
```

`echo` をつけているのは終了ステータスの確認と応答に改行がつかないことの対処です。

この応答のうち、`client_id` と `client_secret` が必要なので、保存しておきます。

## アカウントと紐付け

保存しておいた `client_id` と `client_secret` と、対象のサーバーに登録しているアカウントのメールアドレスとパスワードが必要です。

スコープは Create App の時に `read` のみ以外を指定した場合には、ここにも指定しないと `read` 権限のみになってしまうようです。
Create App の時に指定していなかったスコープは指定できないようです。(`{"error":"invalid_scope","error_description":"The requested scope is invalid, unknown, or malformed."}` になる。)
`/api/v1/apps` は `scopes` (複数形) ですが、`/oauth/token` では `scope` (単数形) なので注意が必要そうです。

```bash
vagrant@mastodon:~$ CLIENT_ID=ce06ac1d988d7f26f067dc7ea639e41523ec8135e48cceabb7cd0fd8c1413c65
vagrant@mastodon:~$ CLIENT_SECRET=be04a4f2ca2cd5d9a1cc1314d03a15f811be7f64005483b003447cc3e35e7749
vagrant@mastodon:~$ YOUR_EMAIL=admin@mastodon.dev
vagrant@mastodon:~$ YOUR_PASSWORD=mastodonadmin
vagrant@mastodon:~$ curl -X POST -d "client_id=$CLIENT_ID&client_secret=$CLIENT_SECRET&grant_type=password&username=$YOUR_EMAIL&password=$YOUR_PASSWORD&scope=${SCOPES:-read write follow}" -sS http://localhost:3000/oauth/token; echo $?
{"access_token":"96efc1d5e78e6bd9497915cbdc9a464d584e934414d3db1e1d68b2370f8b698c","token_type":"bearer","scope":"read write follow","created_at":1492358391}0
```

## API 呼び出し

ほとんどの API 呼び出しにはアクセストークンが必要です。
Create App や public タイムラインの取得などのように必要ないものもあります。

```bash
vagrant@mastodon:~$ ACCESS_TOKEN=96efc1d5e78e6bd9497915cbdc9a464d584e934414d3db1e1d68b2370f8b698c
```

## アカウント情報の取得

`/api/v1/accounts/verify_credentials` でアクセストークンに紐づいているアカウントの情報を取得できます。

```bash
vagrant@mastodon:~$ curl --header "Authorization: Bearer $ACCESS_TOKEN" -sS http://localhost:3000/api/v1/accounts/verify_credentials; echo $?
{"id":1,"username":"admin","acct":"admin","display_name":"","locked":false,"created_at":"2017-04-14T04:04:40.299Z","followers_count":0,"following_count":1,"statuses_count":26,"note":"","url":"http://mastodon.dev/@admin","avatar":"http://localhost:3000/avatars/original/missing.png","avatar_static":"http://localhost:3000/avatars/original/missing.png","header":"http://localhost:3000/headers/original/missing.png","header_static":"http://localhost:3000/headers/original/missing.png"}0
```

## 情報更新

`/oauth/token` で scope を指定していなかった場合、 `{"error":"This action is outside the authorized scopes"}` になって更新できませんでした。

```bash
vagrant@mastodon:~$ curl -X PATCH -d "display_name=Admin" --header "Authorization: Bearer $ACCESS_TOKEN" -sS http://localhost:3000/api/v1/accounts/update_credentials; echo $?
{"id":1,"username":"admin","acct":"admin","display_name":"Admin","locked":false,"created_at":"2017-04-14T04:04:40.299Z","followers_count":0,"following_count":1,"statuses_count":27,"note":"","url":"http://mastodon.dev/@admin","avatar":"http://localhost:3000/avatars/original/missing.png","avatar_static":"http://localhost:3000/avatars/original/missing.png","header":"http://localhost:3000/headers/original/missing.png","header_static":"http://localhost:3000/headers/original/missing.png"}0
```

## トゥート!



```bash
vagrant@mastodon:~$ curl -X POST -d "status=test message" --header "Authorization: Bearer $ACCESS_TOKEN" -sS http://localhost:3000/api/v1/statuses; echo $?
{"id":30,"created_at":"2017-04-16T16:00:55.054Z","in_reply_to_id":null,"in_reply_to_account_id":null,"sensitive":null,"spoiler_text":"","visibility":"public","application":{"name":"Test Client","website":""},"account":{"id":1,"username":"admin","acct":"admin","display_name":"","locked":false,"created_at":"2017-04-14T04:04:40.299Z","followers_count":0,"following_count":1,"statuses_count":27,"note":"","url":"http://mastodon.dev/@admin","avatar":"http://localhost:3000/avatars/original/missing.png","avatar_static":"http://localhost:3000/avatars/original/missing.png","header":"http://localhost:3000/headers/original/missing.png","header_static":"http://localhost:3000/headers/original/missing.png"},"media_attachments":[],"mentions":[],"tags":[],"uri":"tag:mastodon.dev,2017-04-16:objectId=30:objectType=Status","content":"<p>test message</p>","url":"http://mastodon.dev/@admin/30","reblogs_count":0,"favourites_count":0,"reblog":null,"favourited":false,"reblogged":false}0
```

## タイムライン取得

`curl -v --header "Authorization: Bearer $ACCESS_TOKEN" -sS http://localhost:3000/api/v1/timelines/home` などでタイムラインを取得できます。(`timelines` は複数形)

Link ヘッダーに `Link: <http://localhost:3000/api/v1/timelines/home?max_id=7>; rel="next", <http://localhost:3000/api/v1/timelines/home?since_id=30>; rel="prev"` のように書いてあって、 `next` で過去に辿っていけて、 `prev` で新しいトゥートを取得できるようです。
ただし `prev` は時間をあけて試すと間が抜けていたので、全部取得する用途には向いていないようです。
`next` の方は確認していませんが、過去に遡るという挙動を考えると、間が抜けることはないのではないかと思っています。

## エラーの例

パラメーターなどが間違っている時は JSON でエラーが返ってくるのですが、URL を間違えた時は空白のレスポンスが返ってくるので、知らないと悩みそうです。

```bash
vagrant@mastodon:~$ curl -X POST -d "client_name=$CLIENT_NAME&redirect_uris=${REDIRECT_URIS:-urn:ietf:wg:oauth:2.0:oob}&scopes=${SCOPES:-read write follow}&website=$WEBSITE" http://localhost:3000/api/v1/apps; echo $?
{"error":"Validation failed: Name can't be blank"}0
vagrant@mastodon:~$ curl --header "Authorization: Bearer ACCESS_TOKEN_HERE" -sS http://localhost:3000/api/v1/accounts/verify_credentials; echo $?
{"error":"The access token is invalid"}0
vagrant@mastodon:~$ curl --header "Authorization: Bearer $ACCESS_TOKEN" -sS http://localhost:3000/api/v1/timeline/home; echo $?
0
```

## 参考

- [参照時点の API のドキュメント](https://github.com/tootsuite/documentation/blob/eb32fc9738de44878697754431565260be89c17f/Using-the-API/API.md) ([最新](https://github.com/tootsuite/documentation/blob/master/Using-the-API/API.md))
- [参照時点の Testing the API with cURL](https://github.com/tootsuite/documentation/blob/eb32fc9738de44878697754431565260be89c17f/Using-the-API/Testing-with-cURL.md) ([最新](https://github.com/tootsuite/documentation/blob/master/Using-the-API/Testing-with-cURL.md))
