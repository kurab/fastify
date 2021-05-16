<h1 align="center">Fastify</h1>

## はじめに
こんにちは！Fastifyをチェックしていただきありがとうございます！<br>
このドキュメントは、フレームワークとその機能を簡単に紹介することを目的としています。このドキュメントは、例やドキュメントの他の部分へのリンクを含む基本的な導入です。<br>
さあ、始めましょう！

<a name="install"></a>
### インストール
npm でインストール:
```
npm i fastify --save
```
yarn でインストール:
```
yarn add fastify
```

<a name="first-server"></a>
### 初めてのサーバー
初めてのサーバーを作ってみましょう:
```js
// Require the framework and instantiate it
const fastify = require('fastify')({
  logger: true
})

// Declare a route
fastify.get('/', function (request, reply) {
  reply.send({ hello: 'world' })
})

// Run the server!
fastify.listen(3000, function (err, address) {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
  fastify.log.info(`server listening on ${address}`)
})
```

`async/await` を使いたいですよね？ Fastify では特別な設定なしに使えます。<br>
*(ファイルディスクリプタやメモリリークを避けるために [make-promises-safe](https://github.com/mcollina/make-promises-safe) の使用もお勧めします。)*
```js
const fastify = require('fastify')({
  logger: true
})

fastify.get('/', async (request, reply) => {
  return { hello: 'world' }
})

const start = async () => {
  try {
    await fastify.listen(3000)
  } catch (err) {
    fastify.log.error(err)
    process.exit(1)
  }
}
start()
```

簡単ですね。<br>
残念ながら、複雑なアプリケーションを書くには、この例よりもかなり多くのコードが必要になります。新しいアプリケーションを作るときの典型的な問題は、複数のファイル、非同期ブートストラップ、コードのアーキテクチャをどう扱うかということです。<br>
Fastifyは、上記のすべての問題を解決するのに役立つ簡単なプラットフォームを提供しています！

> ## 注意
> 上記の例や、このドキュメントに掲載されている以降の例では、デフォルトでlocalhost `127.0.0.1` のインターフェイス *だけ* をリッスンしています。利用可能なすべてのIPv4インターフェースをリッスンするためには、この例を次のように修正して、`0.0.0.0` をリッスンする必要があります。
>
> ```js
> fastify.listen(3000, '0.0.0.0', function (err, address) {
>   if (err) {
>     fastify.log.error(err)
>     process.exit(1)
>   }
>   fastify.log.info(`server listening on ${address}`)
> })
> ```
>
> 同様に、IPv6によるローカル接続のみを受け付けたい場合は、`::1` を指定します。また、`::` を指定すると、すべての IPv6 アドレスでの接続を受け付け、また、OS がサポートしている場合には、すべての IPv4 アドレスでの接続も受け付けます。
>
> Docker（または他のタイプの）コンテナにデプロイする場合、 `0.0.0.0` または `::` を使用するのがアプリケーションを公開する最も簡単な方法です。

<a name="first-plugin"></a>
### 初めてのプラグイン
すべてがオブジェクトである JavaScript のように、Fastify ではすべてがプラグインです。<br>
詳しく見ていく前に、どのように機能するか見てみましょう。<br>
基本的なサーバを宣言しましょう。ただし、エントリーポイント内でルートを宣言するのではなく、外部ファイルで宣言します（[ルート宣言](Routes.md) を確認してください）。
```js
const fastify = require('fastify')({
  logger: true
})

fastify.register(require('./our-first-route'))

fastify.listen(3000, function (err, address) {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
  fastify.log.info(`server listening on ${address}`)
})
```

```js
// our-first-route.js

async function routes (fastify, options) {
  fastify.get('/', async (request, reply) => {
    return { hello: 'world' }
  })
}

module.exports = routes
```
この例では、Fastify フレームワークのコアである `register` API を使用しました。これは、ルートやプラグインなどを追加する唯一の方法です。

冒頭で、Fastify はアプリケーションの非同期ブートストラップをサポートすると述べました。なぜこれが重要なのでしょうか？
データストレージを処理するためにデータベース接続が必要なシナリオを考えてみましょう。サーバーがリクエストを受け付ける前に、データベース接続が可能である必要があります。どのように対処すればよいのでしょうか。<br>
典型的な解決策は、複雑なコールバック、または promise を使用することです。フレームワーク API と他のライブラリやアプリケーションコードを混在させるシステムです。<br>

Fastifyは、最小限の労力で、内部的にこれを処理します！

上記の例にデータベース接続を追加してみましょう。<br>

まず `fastify-plugin` と `fastify-mongodb` をインストールしてください:

```
npm i --save fastify-plugin fastify-mongodb
```

**server.js**
```js
const fastify = require('fastify')({
  logger: true
})

fastify.register(require('./our-db-connector'))
fastify.register(require('./our-first-route'))

fastify.listen(3000, function (err, address) {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
  fastify.log.info(`server listening on ${address}`)
})

```

**our-db-connector.js**
```js
const fastifyPlugin = require('fastify-plugin')

async function dbConnector (fastify, options) {
  fastify.register(require('fastify-mongodb'), {
    url: 'mongodb://localhost:27017/test_database'
  })
}

// Wrapping a plugin function with fastify-plugin exposes the decorators
// and hooks, declared inside the plugin to the parent scope.
module.exports = fastifyPlugin(dbConnector)

```

**our-first-route.js**
```js
async function routes (fastify, options) {
  const collection = fastify.mongo.db.collection('test_collection')

  fastify.get('/', async (request, reply) => {
    return { hello: 'world' }
  })

  fastify.get('/animals', async (request, reply) => {
    const result = await collection.find().toArray()
    if (result.length === 0) {
      throw new Error('No documents found')
    }
    return result
  })

  fastify.get('/animals/:animal', async (request, reply) => {
    const result = await collection.findOne({ animal: request.params.animal })
    if (result === null) {
      throw new Error('Invalid value')
    }
    return result
  })
}

module.exports = routes
```

いや、簡単ですね！<br>
新しい概念を導入したので、ここで行ったことを振り返ってみましょう。<br>
ご覧の通り、データベースコネクタとルートの登録の両方に `register` を使用しています。
これは Fastify の最も重要な機能の一つで、プラグインを宣言したのと同じ順番でロードし、現在のプラグインがロードされた後にのみ次のプラグインをロードします。このようにして、最初のプラグインでデータベースコネクタを登録し、2番目のプラグインでそれを使用することができます。*(プラグインのスコープをどのように扱うかを理解するには、[ここ](Plugins.md#handle-the-scope)を読んでください)*。
プラグインのロードは、`fastify.listen()`、`fastify.inject()`、`fastify.ready()`のいずれかを呼び出したときに始まります。

また、`decorate` API を使用して、カスタムオブジェクトを Fastify 名前空間に追加し、どこでも使用できるようにしました。コードの再利用を容易にし、コードやロジックの重複を減らすために、この API を使うことを推奨します。


Fastify プラグインがどのように動作するか、新しいプラグインをどのように開発するか、アプリケーションを非同期的にブートストラップする複雑さに対処するために Fastify API 全体をどのように使用するかについての詳細は、[the hitchhiker's guide to plugins](Plugins-Guide.md)をお読みください。

<a name="plugin-loading-order"></a>
### プラグインのロード順
アプリケーションが一貫した、予測可能な動作を保証するために、常に以下のようにコードをロードすることを強くお勧めします。
```
└── plugins (from the Fastify ecosystem)
└── your plugins (your custom plugins)
└── decorators
└── hooks
└── your services
```
このようにして、現在のスコープで宣言されたすべてのプロパティに常にアクセスできるようになります。<br/>
前述したように、Fastify は強固なカプセル化モデルを提供しており、アプリケーションを単一で独立したサービスとして構築するのに役立ちます。もし、ルートのサブセットのためだけにプラグインを登録したい場合は、以下のような構造を再現すればよいのです。
```
└── plugins (from the Fastify ecosystem)
└── your plugins (your custom plugins)
└── decorators
└── hooks
└── your services
    │
    └──  service A
    │     └── plugins (from the Fastify ecosystem)
    │     └── your plugins (your custom plugins)
    │     └── decorators
    │     └── hooks
    │     └── your services
    │
    └──  service B
          └── plugins (from the Fastify ecosystem)
          └── your plugins (your custom plugins)
          └── decorators
          └── hooks
          └── your services
```

<a name="validate-data"></a>
### データのバリデーション
データのバリデーションは非常に重要であり、フレームワークのコアコンセプトです。
受信したリクエストを検証するために、Fastify は [JSON Schema](https://json-schema.org/) を使用します。
ルートのバリデーションの例を見てみましょう:
```js
const opts = {
  schema: {
    body: {
      type: 'object',
      properties: {
        someKey: { type: 'string' },
        someOtherKey: { type: 'number' }
      }
    }
  }
}

fastify.post('/', opts, async (request, reply) => {
  return { hello: 'world' }
})
```
この例では、ルートにオプションオブジェクトを渡す方法を示しています。オプションオブジェクトは、ルート、`body`、`querystring`、`params`、`headers` のすべてのスキーマを含む `schema` キーを受け取ります。<br>
詳しくは[Validation and Serialization](Validation-and-Serialization.md)をお読みください。

<a name="serialize-data"></a>
### データのシリアライズ
Fastify は JSON を手厚くサポートしています。JSON の解析や JSON のシリアライズには、非常に最適化されています。<br>
JSON のシリアライズを高速化するために（遅いでしょ？！）、次の例に示すように、schema オプションの `response` キーを使用します。
```js
const opts = {
  schema: {
    response: {
      200: {
        type: 'object',
        properties: {
          hello: { type: 'string' }
        }
      }
    }
  }
}

fastify.get('/', opts, async (request, reply) => {
  return { hello: 'world' }
})
```
このようにスキーマを指定することで、シリアライズを2～3倍高速化することができます。また、Fastify はレスポンススキーマに存在するデータのみをシリアライズするので、潜在的な機密データの漏洩を防ぐことができます。
詳しくは[Validation and Serialization](Validation-and-Serialization.md)をお読みください。

<a name="extend-server"></a>
### サーバーを拡張する
Fastify は非常に拡張性が高く、ミニマルに作られています。私たちは、素朴なフレームワークこそが素晴らしいアプリケーションを実現するのに必要だと考えています。<br>
言い換えれば、Fastify は「バッテリーを内蔵した」フレームワークではなく、素晴らしい[エコシステム](Ecosystem.md)に依存しているのです!

<a name="test-server"></a>
### テストする
Fastify はテストフレームワークを提供していませんが、Fastify の機能とアーキテクチャを利用してテストを書く方法を推奨しています。<br>
詳しくは [testing](Testing.md) のドキュメントを読んでください。

<a name="cli"></a>
### CLI からサーバーを起動する
Fastify は、[fastify-cli](https://github.com/fastify/fastify-cli) のおかげで CLI も統合されています。

まず、`fastify-cli` をインストールしましょう:

```
npm i fastify-cli
```

グローバルオプション `-g` をつけても良いです。

次に、`package.json` に以下を追加します:
```json
{
  "scripts": {
    "start": "fastify start server.js"
  }
}
```

サーバーファイルを作成します:
```js
// server.js
'use strict'

module.exports = async function (fastify, opts) {
  fastify.get('/', async (request, reply) => {
    return { hello: 'world' }
  })
}
```

最後に、コマンドで起動します:
```bash
npm start
```

<a name="slides"></a>
### 参考資料とビデオ
- Slides
  - [Take your HTTP server to ludicrous speed](https://mcollina.github.io/take-your-http-server-to-ludicrous-speed) by [@mcollina](https://github.com/mcollina)
  - [What if I told you that HTTP can be fast](https://delvedor.github.io/What-if-I-told-you-that-HTTP-can-be-fast) by [@delvedor](https://github.com/delvedor)

- Videos
  - [Take your HTTP server to ludicrous speed](https://www.youtube.com/watch?v=5z46jJZNe8k) by [@mcollina](https://github.com/mcollina)
  - [What if I told you that HTTP can be fast](https://www.webexpo.net/prague2017/talk/what-if-i-told-you-that-http-can-be-fast/) by [@delvedor](https://github.com/delvedor)
