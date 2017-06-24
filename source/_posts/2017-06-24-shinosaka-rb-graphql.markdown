---
layout: post
title: "Shinosaka.rb #27 (GraphQL) に参加した"
date: 2017-06-24 13:00:00 +0900
comments: true
categories: event
---
[Shinosaka.rb #27](https://shinosakarb.doorkeeper.jp/events/61090) に参加しました。
Shinosaka.rb 自体は初参加でした。

今回は GraphQL の解説と node と rails でのハンズオンでした。

<!--more-->

以下、メモです。

## メモ

- [TDDBC大阪4.0 2017年7月1日（大阪府） - こくちーずプロ（告知'sプロ）](http://www.kokuchpro.com/event/tddbcosaka4/)
- [［認定証発行］アジャイル・リーダーシップとチェンジ・マネジメント・ワークショップ（Management 3.0） - Management 3.0（アジャイル・リーダーシップ、チェンジ･マネジメント、イノベーション・マネジメント） | Doorkeeper](https://management30.doorkeeper.jp/events/61787)
- 自己紹介
- QraphQL とは?
- 単一のエンドポイント
- ライブラリーもいろんな言語や環境に対応している
- REST API がツライから
- GraphQL の微妙な点: pagination, 認証も考慮されていない
- React との相性が良い: データ駆動
- GitHub が GraphQL を採用: https://developer.github.com/v4/explorer/
- http://graphql.org/

## 動作確認環境

- macOS Sierra 10.12.5
- node v8.1.2
- npm 5.0.3
- graphql などのバージョン: https://github.com/znz/getting_started_graphql を参照
- ruby ruby 2.3.3p222 (2016-11-21 revision 56859) [x86_64-darwin15]
- Bundler version 1.15.1
- Rails 5.1.1
- graphql 1.6.4, graphiql-rails 1.4.2
- その他: https://github.com/znz/getting_started_graphql_ruby を参照

## ソースコード

- node 版は https://github.com/znz/getting_started_graphql を参照
- Rails 版は https://github.com/znz/getting_started_graphql_ruby を参照

途中での graphiql での確認方法は下の作業メモの方に書いてあるので、あわせて参照してください。

## 感想

作業メモが長く続くので、先に感想を書いておきます。

node の方はエラーも json で帰ってきてブラウザーで見えて開発環境として使いやすそうな感じでしたが、
graphiql-rails の方はエラーの時に `SyntaxError: Unexpected token < in JSON at position 0` とだけ出て、
詳細はサーバー側のログをみないといけないので、node に比べるとちょっと使いづらいかもしれない、と思いました。

GraphQL 自体は色々と利点も多そうだと思いましたが、サーバー側は結局 REST とは別に作り込まないといけなさそうで、
アクセス権限などを考えると、既存のアプリケーションで簡単に置き換えられるものでもなさそうかな、と思いました。

## 実際に使ってみる

- http://graphql.org/code/
- npm init -y
- npm install graphql

## step 1

index.js として以下の内容を作成:

```
'use strict'

const { graphql, buildSchema } = require('graphql')

const schema = buildSchema(`
type Query {
  foo: String
}

type Schema {
  query: Query
}
`)

const resolvers = {
  foo: () => 'bar',
}

const query = `
query myQuery {
  foo
}
`

graphql(schema, query, resolvers)
  .then(result => console.log(result))
  .catch(err => console.log(err))
```

実行結果:

```
% node.index.js
{ data: { foo: 'bar' } }
```

## step 2

```
const schema = buildSchema(`
type Query {
  id: ID,
  title: String,
  watched: Boolean,
}

type Schema {
  query: Query
}
`)
```

```
const resolvers = {
  id: () => 1,
  title: () => 'bar',
  watched: () => true,
}
```

```
const query = `
query myQuery {
  id,
  title,
  watched,
}
`
```

```
% node index.js
{ data: { id: '1', title: 'bar', watched: true } }
```

query から watched を削ると `{ data: { id: '1', title: 'bar' } }` になる。

## step 3

```
const schema = buildSchema(`
type Video {
  id: ID,
  title: String,
  watched: Boolean,
}

type Query {
  video: Video
}

type Schema {
  query: Query
}
`)
```

```
const resolvers = {
  video: () =>({
    id: 1,
    title: 'bar',
    watched: true
  }),
}
```

```
const query = `
query myQuery {
  video {
    id,
    title,
    watched,
  }
}
`
```

## step 4

videos 対応

```
const schema = buildSchema(`
type Video {
  id: ID,
  title: String,
  watched: Boolean,
}

type Query {
  video: Video,
  videos: [Video],
}

type Schema {
  query: Query
}
`)
```

```
const videoA = {
  id: 1,
  title: 'title1',
  watched: true
}
const videoB = {
  id: 2,
  title: 'title2',
  watched: false
}
const videos = [videoA, videoB]
```

```
const resolvers = {
  video: () => ({
    id: 1,
    title: 'bar',
    watched: true,
  }),
  videos: () => videos,
}
```

```
const query = `
query myQuery {
  videos {
    id,
    title,
    watched,
  }
}
`
```

```
% node index.js
{ data: { videos: [ [Object], [Object] ] } }
```

## step 5

`yarn add express express-graphql` or `npm install express express-graphql`

`require('graphql')` の行の上に追加:

```
const express = require('express')
const graphqlHTTP = require('express-graphql')
```

追加:

```
const PORT = process.env.PORT || 3000
const server = express()
```

末尾の `graphql` の呼び出しを置き換え:

```
server.use('/graphql', graphqlHTTP({
  schema,
  graphiql: true,
  rootValue: resolvers,
}))

server.listen(PORT, () => {
  console.log(`Listening on http://localhost:${PORT}`)
})
```

`http://localhost:3000/graphql` を開いて

```
{
  videos {
    id,
    title,
    watched,
  }
}
```

などを試す。

右上の Docs でスキーマも見える。

## step 6

graphql の require のところを書き換え:

```
const {
  GraphQLSchema,
  GraphQLObjectType,
  GraphQLID,
  GraphQLString,
  GraphQLBoolean,
} = require('graphql')
```

buildSchema を書き換え:

```
const videoType = new GraphQLObjectType({
  name: 'Video',
  description: 'video',
  fields: {
    id: {
      type: GraphQLID,
      description: 'id of video',
    },
    title: {
      type: GraphQLString,
      description: 'title of video'
    },
    watched: {
      type: GraphQLBoolean,
      description: 'has watched'
    }
  }
})
```

```
const queryType = new GraphQLObjectType({
  name: 'QueryType',
  description: 'root query',
  fields: {
    video: {
      type: videoType,
      resolve: () => new Promise(resolve => {
        resolve({
          id: 1,
          title: 'title1',
          watched: true,
        })
      })
    }
  }
})
```

```
const schema = new GraphQLSchema({
  query: queryType,
})
```

`node index.js` を再起動して `http://localhost:3000/graphql` で

```
{
  video {
    id
    title
    watched
  }
}
```

などを試す。

## 休憩

## id: 1 だけ欲しいときなど

videos を移動して data.js を作成:

```
'use strict'

const videoA = {
  id: 1,
  title: 'title1',
  watched: true
}
const videoB = {
  id: 2,
  title: 'title2',
  watched: false
}
const videos = [videoA, videoB]

const getVideoById = (id) => new Promise(resolve => {
  const [video] = videos.filter(v => (v.id + '') === id)
  resolve(video)
})

exports.getVideoById = getVideoById
```

```
const { getVideoById } = require('./data')
```

```
const queryType = new GraphQLObjectType({
  name: 'QueryType',
  description: 'root query',
  fields: {
    video: {
      type: videoType,
      args: {
        id: {
          type: GraphQLID,
          description: 'id of video',
        },
      },
      resolve: (_, args) => getVideoById(args.id)
    }
  }
})
```

`node index.js` を再起動して `http://localhost:3000/graphql` で

```
{
  video(id: 2) {
    id
    title
    watched
  }
}
```

などを試す。

## id を必須にしたい

`require('graphql')` のところに `GraphQLNonNull,` を追加。

`type: new GraphQLNonNull(GraphQLID),` にする。

```
{
  "errors": [
    {
      "message": "Unknown operation named \"null\"."
    }
  ]
}
```

になってしまったが、 getVideos の追加の後、もう一度試したら動いたので謎。
謎のエラーが発生した時は Prettify を押すとエラーが起きなくなるみたい。

```
{
  video {
    id
    title
    watched
  }
}
```

などを試すと以下のように意図通りのエラーになる。

```
{
  "errors": [
    {
      "message": "Field \"video\" argument \"id\" of type \"ID!\" is required but not provided.",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ]
    }
  ]
}
```

## 配列

`GraphQLList` を追加

data.js に追加:

```
const getVideos = () => new Promise(resolve => resolve(videos))
```

```
exports.getVideos = getVideos
```

index.js:

```
const { getVideoById, getVideos } = require('./data')
```

```
    videos: {
      type: new GraphQLList(videoType),
      resolve: getVideos,
    },
```

`node index.js` を再起動して `http://localhost:3000/graphql` で

```
{
  videos {
    id
    title
    watched
  }
}
```

などを試す。

## mutation

schema に mutation を追加:

```
const schema = new GraphQLSchema({
  query: queryType,
  mutation: mutationType,
})
```

schema の上に追加:

```
const mutationType = new GraphQLObjectType({
  name: 'Mutation',
  description: 'Mutation type',
  fields: {
    createVideo: {
      type: videoType,
      args: {
        title: {
          type: new GraphQLNonNull(GraphQLString),
          description: 'title of video',
        },
      },
      resolve: (_, args) => {
        return createVideo(args)
      }
    },
  },
})
```

data.js:

```
const createVideo = ({ title }) => {
  const maxId = Math.max.apply(null, videos.map(v => v.id))
  const watched = false
  const video = {
    id: maxId + 1,
    title,
    watched,
  }
  return video
}
```

(videos への push が抜けていた。)

```
exports.createVideo = createVideo
```

index.js:

```
const { getVideoById, getVideos, createVideo } = require('./data')
```

`node index.js` を再起動して `http://localhost:3000/graphql` で

```
mutation M {
  createVideo(title: "hoge") {
    id
    title
    watched
  }
}
```

を試す。

この時点の index.js:

```
'use strict'

const express = require('express')
const graphqlHTTP = require('express-graphql')
const {
  GraphQLSchema,
  GraphQLObjectType,
  GraphQLID,
  GraphQLString,
  GraphQLBoolean,
  GraphQLNonNull,
  GraphQLList,
} = require('graphql')
const { getVideoById, getVideos, createVideo } = require('./data')

const PORT = process.env.PORT || 3000
const server = express()

/*
video
  id
  title
  watched
*/

const videoType = new GraphQLObjectType({
  name: 'Video',
  description: 'video',
  fields: {
    id: {
      type: GraphQLID,
      description: 'id of video',
    },
    title: {
      type: GraphQLString,
      description: 'title of video'
    },
    watched: {
      type: GraphQLBoolean,
      description: 'has watched'
    }
  }
})

const queryType = new GraphQLObjectType({
  name: 'QueryType',
  description: 'root query',
  fields: {
    videos: {
      type: new GraphQLList(videoType),
      resolve: getVideos,
    },
    video: {
      type: videoType,
      args: {
        id: {
          type: new GraphQLNonNull(GraphQLID),
          description: 'id of video',
        }
      },
      resolve: (_, args) => getVideoById(args.id)
    }
  }
})

const mutationType = new GraphQLObjectType({
  name: 'Mutation',
  description: 'Mutation type',
  fields: {
    createVideo: {
      type: videoType,
      args: {
        title: {
          type: new GraphQLNonNull(GraphQLString),
          description: 'title of video',
        },
      },
      resolve: (_, args) => {
        return createVideo(args)
      }
    },
  },
})

const schema = new GraphQLSchema({
  query: queryType,
  mutation: mutationType,
})

server.use('/graphql', graphqlHTTP({
  schema,
  graphiql: true,
}))

server.listen(PORT, () => {
  console.log(`Listening on http://localhost:${PORT}`)
})
```

data.js:

```
'use strict'

const videoA = {
  id: 1,
  title: 'title1',
  watched: true
}
const videoB = {
  id: 2,
  title: 'title2',
  watched: false
}
const videos = [videoA, videoB]

const getVideos = () => new Promise(resolve => resolve(videos))

const createVideo = ({ title }) => {
  const maxId = Math.max.apply(null, videos.map(v => v.id))
  const watched = false
  const video = {
    id: maxId + 1,
    title,
    watched,
  }
  videos.push(video)
  return video
}

const getVideoById = (id) => new Promise(resolve => {
  const [video] = videos.filter(v => (v.id + '') === id)
  resolve(video)
})

exports.getVideoById = getVideoById
exports.getVideos = getVideos
exports.createVideo = createVideo
```

## createVideo の args を分離したい

require のところに `GraphQLInputObjectType,` を追加。

```
const videoInputType = new GraphQLInputObjectType({
  name: 'VideoInputType',
  description: 'video input type',
  fields: {
    title: {
      type: new GraphQLNonNull(GraphQLString),
      description: 'title of video',
    },
  }
})

const mutationType = new GraphQLObjectType({
  name: 'Mutation',
  description: 'Mutation type',
  fields: {
    createVideo: {
      type: videoType,
      args: {
        video: {
          type: new GraphQLNonNull(videoInputType)
        },
      },
      resolve: (_, args) => {
        return createVideo(args.video)
      }
    },
  },
})
```

`node index.js` を再起動して `http://localhost:3000/graphql` で

```
mutation M {
  createVideo(video: {title: "hoge"}) {
    id
    title
    watched
  }
}
```

を試す (`video:` で一段増えているので注意)

```
{
  videos {
    id
    title
  }
}
```

などを試す。

## 休憩

### createVideo も Promise にするとどうか

createVideo の末尾を `return Promise.resolve(video)` にしても問題なく動いた。

## ruby でどうか

- `rails new getting_started_graphql_ruby`
- http://graphql-ruby.org/getting_started
- Gemfile に `gem 'graphql'` を追加
- `bundle install`
- `rails g graphql:install`
- Gemfile に `graphiql-rails` が追加されているので `bundle install`

## video 追加

- `rails g graphql:object Video id:Int title:String watched:Boolean`
- id は Int ではなく ID が正しいので `rails d graphql:object Video id:Int title:String watched:Boolean` で消してやり直し
- `rails g graphql:object Video id:ID title:String watched:Boolean`
- `app/graphql/types/query_type.rb` を変更

```
  field :video do
    type Types::VideoType
    argument :id, !types.ID
    description 'Find video by ID'
    resolve ->(obj, args, ctx) { Video.find(args["id"]) }
  end
```

`rails s` を起動して `http://localhost:3000/graphiql` (express-graphql での例と違って `/graphql` ではなく `i` が入る) で

```
{
  video(id: 1) {
    id
    title
  }
}
```

を試すと server 側で `NameError (uninitialized constant Video):` になるのを確認。

- `rails g model video title watched:boolean`
- `rake db:migrate`
- `rails c` で `Video.create(title: "Hoge", watched: false)` などでレコードを作成しておく
- graphiql で試す

```
{
  video(id: 1) {
    id
    title
    watched
  }
}
```

## mutation

- `app/graphql/mutations/create_video.rb`

```
# 動かない
Mutations::CreateVideo = GraphQL::Relay::Mutation.define do
  name "CreateVideo"

  return_field :video, Types::VideoType

  input_field :title, !types.String

  resolve ->(obj, args, ctx) {
    return Video.create(title: args["title"])
  }
end
```

- `app/graphql/getting_started_graphql_ruby_schema.rb` に `mutation(Mutations::CreateVideo)` を追加
- `GraphQL::Schema::InvalidTypeError (CreateVideo has an invalid type: must be an instance of GraphQL::BaseType, not GraphQL::Relay::Mutation` になってうまくいかない
- `rails g graphql:mutation` は relay mutation 用で違うらしい

## クライアント

- http://dev.apollodata.com/react/ が redux っぽくてしっくりきたらしい。
- rack-cors の設定が必要?
- https://github.com/facebook/relay

## mutation の動くコード例

rito さんに動く例をみせてもらって修正。


`app/graphql/mutations/video.rb`:

```
Mutations::Video = GraphQL::ObjectType.define do
  name "mutation"

  field :video, Types::VideoType do
    description "Create a video"
    argument :title, !types.String

    resolve ->(obj, args, ctx) {
      Video.create(title: args["title"], watched: false)
    }
  end
end
```

(`name "Video"` にすると `Duplicate type definition found for name 'Video'` で動かなかった。)

`app/graphql/getting_started_graphql_ruby_schema.rb`:

```
GettingStartedGraphqlRubySchema = GraphQL::Schema.define do
  query(Types::QueryType)
  mutation(Mutations::Video)
end
```

`http://localhost:3000/graphiql` で以下を試す。

```
mutation M {
  video(title: "foo") {
    id
    title
    watched
  }
}
```

```
{
  video(id: 2) {
    id
    title
    watched
  }
}
```

追加されたのがみえたら OK

時間切れで試せなかったけど、 mutation を複数追加する場合はどうなるのかがわからなかった。
