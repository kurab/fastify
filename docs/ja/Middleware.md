<h1 align="center">Fastify</h1>

## Middleware

Fastify v3.0.0 からは、Middleware がサポートされていないため、[`fastify-express`](https://github.com/fastify/fastify-express) や [`middie`](https://github.com/fastify/middie) などの外部プラグインが必要になります。

Expressミドルウェアを `use` するために [`fastify-express`](https://github.com/fastify/fastify-express) プラグインを登録した例です。

```js
await fastify.register(require('fastify-express'))
fastify.use(require('cors')())
fastify.use(require('dns-prefetch-control')())
fastify.use(require('frameguard')())
fastify.use(require('hsts')())
fastify.use(require('ienoopen')())
fastify.use(require('x-xss-protection')())
```

また、シンプルな Express スタイルの Middleware をサポートしつつ、パフォーマンスを向上させた [`middie`](https://github.com/fastify/middie) を使用することもできます。

```js
await fastify.register(require('middie'))
fastify.use(require('cors')())
```

Middleware はカプセル化できることを覚えておいてください。つまり、[plugins guide](Plugins-Guide.md) で説明されているように、`register` を使って Middleware の実行場所を決めることができるのです。

Fastify Middleware は、Fastify [Reply](Reply.md#reply) インスタンスに固有の `send` メソッドやその他のメソッドを公開しません。
これは、Fastify が [Request](Request.md#request) と [Reply](Request.md#reply) オブジェクトを使って受信する `req` と `res` の Node インスタンスを内部でラップしているからですが、これは Middleware の後に行われます。
Middleware を作成する必要がある場合は、Node `req` および `res` インスタンスを使用する必要があります。
もしくは、[Request](Request.md#request) と [Reply](Reply.md#reply) の Fastify インスタンスをすでに持っている `preHandler` hook を使うことができます。
詳細については、[Hooks](Hooks.md#hooks) を参照してください。

<a name="restrict-usage"></a>
### 特定のパスによる Middleware の実行制限
特定のパスの下でのみミドルウェアを実行する必要がある場合は、`use` する最初のパラメータとしてパスを渡すだけです！

*ただし、パラメータ付きのルート（例：`/user/:id/comments`）には対応しておらず、複数のパスでのワイルドカードにも対応していません。*

```js
const path = require('path')
const serveStatic = require('serve-static')

// Single path
fastify.use('/css', serveStatic(path.join(__dirname, '/assets')))

// Wildcard path
fastify.use('/css/(.*)', serveStatic(path.join(__dirname, '/assets')))

// Multiple paths
fastify.use(['/css', '/js'], serveStatic(path.join(__dirname, '/assets')))
```

### 代替

Fastify は、最も一般的に使用されている Middleware の代替品をいくつか提供しています。例えば、[`helmet`](https://github.com/helmetjs/helmet) の場合は [`fastify-helmet`](https://github.com/fastify/fastify-helmet)、[`cors`](https://github.com/expressjs/cors) の場合は [`fastify-cors`](https://github.com/fastify/fastify-cors)、そして [`serve-static`](https://github.com/expressjs/serve-static) の場合は [`fastify-static`](https://github.com/fastify/fastify-static) などです。
