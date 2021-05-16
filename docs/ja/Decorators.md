<h1 align="center">Fastify</h1>

## Decorators

デコレータ API は、サーバーインスタンス自体や、HTTP リクエストサイクル内で使用されるリクエストオブジェクトや応答オブジェクトなど、
Fastify のコアオブジェクトのカスタマイズを可能にします。デコレータ API は、あらゆるタイプのプロパティをコアオブジェクトにアタッチすることができます。例えば、関数、プレーンオブジェクトまたはネイティブタイプなど

このAPIは*同期*です。デコレータを非同期に定義しようとすると、デコレータの初期化が完了する前に Fastify インスタンスが起動してしまう可能性があります。この問題を回避し、非同期のデコレータを登録するには、`register` API と `fastify-plugin` の組み合わせを代わりに使用する必要があります。詳しくは、[Plugins](Plugins.md) のドキュメントを参照してください。

この API でコアオブジェクトを装飾することで、基盤となる JavaScript エンジンが、サーバー、リクエスト、および応答オブジェクトの処理を最適化することができます。これはインスタンス化される前に、これらのオブジェクトインスタンスを定義することで実現されます。

以下のようにオブジェクトのライフサイクル中にオブジェクトを変更するような使い方は推奨されません。

```js
// ダメな例！

// リクエストハンドラが呼び出される前に、受信したリクエストに user プロパティを追加
fastify.addHook('preHandler', function (req, reply, done) {
  req.user = 'Bob Dylan'
  done()
})

// 追加された user プロパティをリクエストハンドラで使用
fastify.get('/', function (req, reply) {
  reply.send(`Hello, ${req.user}`)
})
```

上記の例では、リクエストオブジェクトがすでにインスタンス化された後に変更を行うため、JavaScript エンジンは、リクエストオブジェクトへのアクセスを最適化しなければなりません。デコレータ API を使用することで、この最適化の必要がなくなります。

```js
// まず、リクエストに 'user' プロパティをつける
fastify.decorateRequest('user', '')

// プロパティを更新する
fastify.addHook('preHandler', (req, reply, done) => {
  req.user = 'Bob Dylan'
  done()
})
// 最後にアクセスする
fastify.get('/', (req, reply) => {
  reply.send(`Hello, ${req.user}!`)
})
```

装飾されたフィールドの初期形状は、将来的に動的に設定される値に可能な限り近づけることが重要です。デコレータの初期化は、値が文字列の場合は `''` として、オブジェクトや関数の場合は `null` として行います。

参照型はすべてのリクエストで共有されるので、この例は値型でのみ動作する点に注意してください。
[decorateRequest](#decorate-request) 参照

このトピックに関するより詳細な情報は、
[JavaScript engine fundamentals: Shapes and Inline Caches](https://mathiasbynens.be/notes/shapes-ics)
を参照してください。

### 利用方法
<a name="usage"></a>

#### `decorate(name, value, [dependencies])`
<a name="decorate"></a>

このメソッドは、[Fastify server](Server.md) インスタンスをカスタマイズするために使用されます。

例えば、新しいメソッドをサーバーインスタンスにアタッチする場合などです:

```js
fastify.decorate('utility', function () {
  // いい感じの処理
})
```

前述の通り、非機能的な値を付けることができます:

```js
fastify.decorate('conf', {
  db: 'some.db',
  port: 3000
})
```

装飾されたプロパティにアクセスするには、以下のようにデコレータ API に提供された name を利用します:

```js
fastify.utility()

console.log(fastify.conf.db)
```

装飾された [Fastify server](Server.md) は、ルートハンドラ - [route](Routes.md) の中で、`this` にバインドされます:

```js
fastify.decorate('db', new DbConnection())

fastify.get('/', async function (request, reply) {
  reply({hello: await this.db.query('world')})
})
```

`dependencies` パラメータは、定義されているデコレータが依存しているデコレータのリスト（オプション）です。このリストは、他のデコレータの文字列名の単純なリストです。次の例では、"utility "デコレータが "greet "と "log "デコレータに依存しています。

```js
fastify.decorate('utility', fn, ['greet', 'log'])
```

依存関係が満たされていない場合は、`decorate` メソッドは例外を投げます。
依存関係のチェックは、サーバーインスタンスが起動する前に行われ、ランタイム中には行われません。

#### `decorateReply(name, value, [dependencies])`
<a name="decorate-reply"></a>

その名の通り、この API は新しいメソッドやプロパティをコアな `Reply` オブジェクトに追加するために使用します:

```js
fastify.decorateReply('utility', function () {
  // いい感じの処理
})
```

注意：アロー関数を使用すると、Fastify の `Reply` インスタンスに対する `this` のバインディングが壊れます。

注意: `decorateReply` を参照型で使用すると、警告が表示されます:

```js
// やめて。
fastify.decorateReply('foo', { bar: 'fizz'})
```
この例では、オブジェクトの参照はすべてのリクエストで共有されています。**何か変更を加えれば、すべてのリクエストに影響し、セキュリティの脆弱性やメモリリークを引き起こす可能性があります。**

To achieve proper encapsulation across requests configure a new value for each incoming request
in the [`'onRequest'` hook](Hooks.md#onrequest). Example:

リクエスト間で適切なカプセル化を行うためには、[`'onRequest'` hook](Hooks.md#onrequest)で、リクエストごとに新しい値を設定してください。例:

```js
const fp = require('fastify-plugin')

async function myPlugin (app) {
  app.decorateRequest('foo', null)
  app.addHook('onRequest', async (req, reply) => {
    req.foo = { bar: 42 }
  }) 
}

module.exports = fp(myPlugin)
```

`dependencies` パラメータについては、[`decorate`](#decorate) を参照してください。

#### `decorateRequest(name, value, [dependencies])`
<a name="decorate-request"></a>

上記の [`decorateReply`](#decorate-reply) のように、この API はコアの `Request` オブジェクトに新しいメソッドやプロパティを追加するために使用されます。

```js
fastify.decorateRequest('utility', function () {
  // いい感じの処理
})
```

注意：アロー関数を使用すると、Fastify の `Reply` インスタンスに対する `this` のバインディングが壊れます。

注意: `decorateRequest` を参照型で使用すると、警告が表示されます:

```js
// やめて。
fastify.decorateRequest('foo', { bar: 'fizz'})
```
この例では、オブジェクトの参照はすべてのリクエストで共有されています。**あらゆる変更はすべてのリクエストに影響を与え、セキュリティの脆弱性やメモリリークを引き起こす可能性があります。**

リクエスト間で適切なカプセル化を行うためには、[`'onRequest'` hook](Hooks.md#onrequest)で、リクエストごとに新しい値を設定してください。例:

```js
const fp = require('fastify-plugin')

async function myPlugin (app) {
  app.decorateRequest('foo', null)
  app.addHook('onRequest', async (req, reply) => {
    req.foo = { bar: 42 }
  }) 
}

module.exports = fp(myPlugin)
```

`dependencies` パラメータについては、[`decorate`](#decorate) を参照してください。

#### `hasDecorator(name)`
<a name="has-decorator"></a>

サーバーインスタンスのデコレータが存在しているかを確認するために使用されます。

```js
fastify.hasDecorator('utility')
```

#### hasRequestDecorator
<a name="has-request-decorator"></a>

Request デコレータが存在しているか確認するために使用されます。

```js
fastify.hasRequestDecorator('utility')
```

#### hasReplyDecorator
<a name="has-reply-decorator"></a>

Reply デコレータが存在しているか確認するために使用されます。


```js
fastify.hasReplyDecorator('utility')
```

### Decorators and Encapsulation
<a name="decorators-encapsulation"></a>

同一の**カプセル化された**コンテキストで、同じ名前のデコレータ（`decorate`、`decorateRequest`、`decorateReply`を使用）を複数回定義すると、例外が発生します。

例として、次のようなものは例外が投げられます。

```js
const server = require('fastify')()

server.decorateReply('view', function (template, args) {
  // いい感じのレンダリングエンジン
})

server.get('/', (req, reply) => {
  reply.view('/index.html', { hello: 'world' })
})

// コード内の別の場所で、違うデコレータを定義すると、例外が投げられる
server.decorateReply('view', function (template, args) {
  // 違うレンダリングエンジン
})

server.listen(3000)
```


しかし、以下の例では例外は投げられません。

```js
const server = require('fastify')()

server.decorateReply('view', function (template, args) {
  // いい感じのレンダリングエンジン
})

server.register(async function (server, opts) {
  // 現在のカプセル化されたプラグインに View デコレータを追加すると、
  // このカプセル化されたプラグインの外部では View は古いものであり、
  // 内部では新しいものであるため、例外は投げられません。
  server.decorateReply('view', function (template, args) {
    // 違うレンダリングエンジン
  })

  server.get('/', (req, reply) => {
    reply.view('/index.page', { hello: 'world' })
  })
}, { prefix: '/bar' })

server.listen(3000)
```

### Getters and Setters
<a name="getters-setters"></a>

デコレータは特別な "getter/setter" オブジェクトを受け入れます。これらのオブジェクトは `getter` と `setter` という名前の関数を持っています (ただし、`setter` 関数はオプションです)。これにより、デコレータ経由でプロパティを定義することができます。

例えば、

```js
fastify.decorate('foo', {
  getter () {
    return 'a getter'
  }
})
```

とすると、`foo` プロパティが Fastify インスタンスに定義されます。

```js
console.log(fastify.foo) // 'a getter'
```
