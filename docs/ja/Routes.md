<h1 align="center">Fastify</h1>

## Routes

routes メソッドは、アプリケーションのエンドポイントを設定します。
Fastify でルートを宣言するには、省略した宣言と詳細な宣言の2つの方法があります。

- [詳細な宣言](#full-declaration)
- [ルートオプション](#options)
- [簡略化した宣言](#shorthand-declaration)
- [URL パラメータ](#url-building)
- [`async`/`await` の使用](#async-await)
- [Promise の解決](#promise-resolution)
- [ルートプレフィックス](#route-prefixing)
- ログ
  - [カスタムログレベル](#custom-log-level)
  - [カスタムログシリアライザー](#custom-log-serializer)
- [ルートハンドラの設定](#routes-config)
- [ルート制約](#constraints)

<a name="full-declaration"></a>
### 詳細な宣言

```js
fastify.route(options)
```

<a name="options"></a>
### ルートオプション

* `method`: 現在 `'DELETE'`、`'GET'`、`'HEAD'`、`'PATCH'`、`'POST'`、`'PUT'`、`'OPTIONS'` をサポートしています。配列で指定することもできます。
* `url`: このルートにマッチする URL のパス（エイリアス: `path`）
* `schema`: リクエストとレスポンスのスキーマを含むオブジェクト。
  [JSON Schema](https://json-schema.org/) フォーマットである必要があります。
  くわしくは, [こちら](Validation-and-Serialization.md) を確認してください。
  * `body`: POST、PUT、PATCH メソッドの場合に、リクエストボディを検証します。
  * `querystring` or `query`: クエリストリングの検証。これは、プロパティ `type` が `object` で、`properties` オブジェクトがパラメータである完全な JSON スキーマオブジェクトである場合もあれば、単純に以下に示すようなプロパティオブジェクトに含まれる値の場合もあります。
  * `params`: パラメータの検証
  * `response`: レスポンスのスキーマをフィルタリングし、生成します。スキーマを設定することで、10〜20％のスループットを確保することができます。
* `exposeHeadRoute`: すべての `GET` ルートに対して、同列の `HEAD` ルートを作成します。デフォルトでは、[`exposeHeadRoutes`](Server.md#exposeHeadRoutes) インスタンスオプションの値が設定されます。このオプションを無効にせずに、カスタム `HEAD` ハンドラを使用したい場合は、必ず `GET` ルートの前で定義してください。
* `attachValidation`: スキーマ検証エラーが発生した場合、エラーハンドラーにエラーを送信するのではなく、リクエストに `validationError` をアタッチします。
* `onRequest(request, reply, done)`: リクエストを受信した際直ちに実行される [function](Hooks.md#onrequest) で、function の配列として指定することもできます。
* `preParsing(request, reply, done)`: リクエストをパースする前に実行される [function](Hooks.md#preparsing) で、function の配列として指定することもできます。
* `preValidation(request, reply, done)`: 共有された `preValidation` フックのあとに呼び出される [function](Hooks.md#prevalidation) で、例えば、ルートレベルでの認証を行う必要がある場合に便利です。function の配列で指定することもできます。
* `preHandler(request, reply, done)`: リクエストハンドラの直前に呼び出される [function](Hooks.md#prehandler) で、配列での指定も可能です。
* `preSerialization(request, reply, payload, done)`: シリアル化する直前に呼び出される [function](Hooks.md#preserialization) で、配列での指定も可能です。
* `onSend(request, reply, payload, done)`: レスポンスが送られる直前に呼び出される [function](Hooks.md#route-hooks) で、配列での指定も可能です。
* `onResponse(request, reply, done)`: レスポンスが送信されたときに呼び出される [function](Hooks.md#onresponse) で、これ以上のデータをクライアントに送信することはできません。配列での指定も可能です。
* `handler(request, reply)`: リクエストを処理する function です。[Fastify server](Server.md) は、`this` にバインドされます。注意: アロー関数を使うと、`this` のバインドが壊れます。
* `errorHandler(error, request, reply)`: リクエストのスコープのためのカスタムエラーハンドラです。ルートへのリクエストに対して、デフォルトのエラー・グローバル・ハンドラ、および [`setErrorHandler`](Server.md#setErrorHandler) で設定されたものをオーバーライドします。デフォルトのハンドラーにアクセスするには、`instance.errorHandler` にアクセスします。これは、プラグインがまだオーバーライドしていない場合に限り、Fastify のデフォルトの `errorHandler` を指すことに注意してください。
* `validatorCompiler({ schema, method, url, httpPart })`: リクエストバリデーションのスキーマを構築 (build) する function です。
[Validation and Serialization](Validation-and-Serialization.md#schema-validator) を確認してください。
* `serializerCompiler({ { schema, method, url, httpStatus } })`: レスポンスのシリアル化のスキーマを構築 (build) する function です。 [Validation and Serialization](Validation-and-Serialization.md#schema-serializer) を確認してください。
* `schemaErrorFormatter(errors, dataVar)`: validation Compiler からのエラーをフォーマットする function です。[Validation and Serialization](Validation-and-Serialization.md#error-handling) を確認してください. ルートのリクエストに対して、グローバルスキーマエラーフォーマッタハンドラと、`setSchemaErrorFormatter` で設定されたものをオーバーライドします。
* `bodyLimit`: デフォルトのJSONボディ・パーサーが、このバイト数よりも大きなリクエスト・ボディをパースするのを防ぎます。値は整数でなければなりません。このオプションは、`fastify(options)` で最初に Fastify インスタンスを作成する際に、グローバルに設定することもできます。デフォルト値は `1048576` (1 MiB)です。
* `logLevel`: ルートのログレベルを設定します。下記を確認してください。
* `logSerializers`: ルートのログにシリアライザを設定します。
* `config`: カスタム設定を保存するためのオブジェクトです。
* `version`: エンドポイントのバージョンを定義した [semver](https://semver.org/)  互換の文字列です。[例](Routes.md#version)
* `prefixTrailingSlash`: プレフィックス付きのルートとして `/` を渡す処理方法を決定するための文字列です。
  * `both` (デフォルト): `/prefix` と `/prefix/` の両方が登録されます。
  * `slash`: `/prefix/` だけが登録されます。
  * `no-slash`: `/prefix` だけが登録されます。

  `request` は [Request](Request.md) で定義されます。

  `reply` は [Reply](Reply.md) で定義されます。


例:
```js
fastify.route({
  method: 'GET',
  url: '/',
  schema: {
    querystring: {
      name: { type: 'string' },
      excitement: { type: 'integer' }
    },
    response: {
      200: {
        type: 'object',
        properties: {
          hello: { type: 'string' }
        }
      }
    }
  },
  handler: function (request, reply) {
    reply.send({ hello: 'world' })
  }
})
```

<a name="shorthand-declaration"></a>
### 簡略化した宣言

上記のルート宣言はより *Hapi* 的ですが、*Express/Restify* 的なアプローチもサポートしています。:<br>
`fastify.get(path, [options], handler)`<br>
`fastify.head(path, [options], handler)`<br>
`fastify.post(path, [options], handler)`<br>
`fastify.put(path, [options], handler)`<br>
`fastify.delete(path, [options], handler)`<br>
`fastify.options(path, [options], handler)`<br>
`fastify.patch(path, [options], handler)`

例:
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
fastify.get('/', opts, (request, reply) => {
  reply.send({ hello: 'world' })
})
```

`fastify.all(path, [options], handler)` サポートされているすべてのメソッドに同じハンドラを追加します。

ハンドラは `options` オブジェクトで指定することもできます。
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
  },
  handler: function (request, reply) {
    reply.send({ hello: 'world' })
  }
}
fastify.get('/', opts)
```

> 注意: ハンドラが `options` と省略した宣言方法の3番目のパラメータの両方で指定されている場合は、`handler` の重複エラーが発生します。

<a name="url-building"></a>
### Url building

Fastify はスタティックとダイナミックの両方の URL をサポートします。<br>
**パラメトリック**パスを登録するには、パラメータ名の前にコロンを使用します。**ワイルドカード**の場合は、*アスタリスク*を使用します。
*スタティックルートは常にパラメトリックとワイルドカードの前にチェックされることに注意してください。*

```js
// パラメトリック
fastify.get('/example/:userId', (request, reply) => {})
fastify.get('/example/:userId/:secretToken', (request, reply) => {})

// ワイルドカード
fastify.get('/example/*', (request, reply) => {})
```

正規表現を用いたルートもサポートされていますが、正規表現はパフォーマンスの観点で、コストが非常に高くなるので、注意してください。
```js
// 正規表現を用いたパラメトリック
fastify.get('/example/:file(^\\d+).png', (request, reply) => {})
```

"/" から次の "/" の間に複数のパラメータを指定することが可能です。
```js
fastify.get('/example/near/:lat-:lng/radius/:r', (request, reply) => {})
```
*この場合、パラメータのセパレータとして、ダッシュ（"-"）を使用することを忘れないでください。*

最後に、正規表現で複数のパラメータを指定することも可能です。
```js
fastify.get('/example/at/:hour(^\\d{2})h:minute(^\\d{2})m', (request, reply) => {})
```
この場合、パラメータのセパレータとして、正規表現でマッチしない任意の文字を使用することができます。

複数のパラメータを持つルートはパフォーマンスに悪影響を及ぼす可能性があるので、可能な限りシングルパラメータのアプローチを取るようにしましょう。特にアプリケーションのホットパス上にあるルートではそうしましょう。ルーティングの処理方法に興味がある方は、[find-my-way](https://github.com/delvedor/find-my-way) をご覧ください。

パラメータを宣言せずにコロンを含むパスを使いたい場合は、ダブルコロンを使います。
```js
fastify.post('/name::verb') // /name:verb と解釈されます。
```

<a name="async-await"></a>
### `async/await` の使用

`async/await` 使いますよね？ありますよ。
```js
fastify.get('/', options, async function (request, reply) {
  var data = await getData()
  var processed = await processData(data)
  return processed
})
```

ご覧のとおり、データをユーザーに送り返すために `reply.send` を呼び出しているわけではありません。ボディを返せばそれでおしまいなのです。

必要であれば、`reply.send` を使ってユーザーにデータを送り返すこともできます。

```js
fastify.get('/', options, async function (request, reply) {
  var data = await getData()
  var processed = await processData(data)
  reply.send(processed)
})
```

ルートが、プロミスチェーンの外で `reply.send()` を呼び出すコールバックベースの API をラッピングしている場合は、`await reply` が可能になります。

```js
fastify.get('/', options, async function (request, reply) {
  setImmediate(() => {
    reply.send({ hello: 'world' })
  })
  await reply
})
```

reply を返しても動きます。

```js
fastify.get('/', options, async function (request, reply) {
  setImmediate(() => {
    reply.send({ hello: 'world' })
  })
  return reply
})
```

**要注意**
* `return value` と `reply.send(value)` を同時に使用した場合、先に発生した方が優先され、2つ目の値は破棄されます。また、レスポンスを2回送ろうとしたため、警告(*warn*)ログが出力されます。
* `undefined` を返すことはできません。詳しくは [promise-resolution](#promise-resolution) を確認してください。

<a name="promise-resolution"></a>
### Promise の解決

ハンドラが `async` 関数であったり、プロミスを返す場合には、コールバックと promise コントロールフローをサポートするのに必要な特別な動作に注意する必要があります。ハンドラの promise が `undefined` で解決された場合、その promise は虫され、リクエストがハングアップして *error* ログが出力されます。

1. `async/await` や promise を使いつつ、`reply.send` で値を返したい場合は、
    - `return` で値を返さないでください
    - `reply.send` を呼び出すことを忘れないでください
2. `async/await` や promise を使いたい場合は、
    - `reply.send` を使わないでください
    - `undefined` を返さないでください

こうすることで、`callback-style` と `async-await` の両方を、最小限のトレードオフでサポートすることができます。エラー処理はアプリケーション内で一貫した方法で処理されるべきなので、自由度が高いとしても、1つのスタイルにすることを強くお勧めします。

**注意**: 全ての acync 関数は、それ自体が promise を返します。

<a name="route-prefixing"></a>
### ルートプレフィックス

古典的なアプローチは、すべてのルートに API バージョン番号のプレフィックスを付け、例えば `/v1/user` とします。
Fastify は、すべてのルート名を手動で変更することなく、同じ API の異なるバージョンを作成するための高速でスマートな方法を提供します。それがどのように機能するか見てみましょう。

```js
// server.js
const fastify = require('fastify')()

fastify.register(require('./routes/v1/users'), { prefix: '/v1' })
fastify.register(require('./routes/v2/users'), { prefix: '/v2' })

fastify.listen(3000)
```

```js
// routes/v1/users.js
module.exports = function (fastify, opts, done) {
  fastify.get('/user', handler_v1)
  done()
}
```

```js
// routes/v2/users.js
module.exports = function (fastify, opts, done) {
  fastify.get('/user', handler_v2)
  done()
}
```
Fastify はコンパイル時にプレフィックスを自動的に処理するので、2つの異なるルートに同じ名前を使っていても平気です。 *(これはパフォーマンスに全く影響しないことも意味します！)*

あなたのクライアントは以下のルートにアクセスできるようになります。
- `/v1/user`
- `/v2/user`

これは何度でも行うことができ、ネストされた `register` でも動作し、 routes パラメータもサポートされています。
なお、[fastify-plugin`](https://github.com/fastify/fastify-plugin) を使用している場合は、このオプションは動作しませんのでご注意ください。

#### Handling of / route inside prefixed plugins

`/` ルートは、プレフィックスが `/` で終わっているかどうかによって、異なる動作をします。例えば、プレフィックスが `/something/` だとすると、`/` ルートを追加すると `/something/` にしかマッチしません。
プレフィックス `/something` を考慮した場合、`/` ルートを追加すると `/something` と `/something/` の両方にマッチします。

この動作を変更するには、上記の `prefixTrailingSlash` ルートオプションを参照してください。

<a name="custom-log-level"></a>
### カスタムログレベル

ルートで異なるログレベルが必要になることもあるでしょう。Fastify は非常に簡単な方法でこれを実現します。
オプション `logLevel` を、必要な [value](https://github.com/pinojs/pino/blob/master/docs/api.md#level-string) とともに、プラグインオプションまたはルートオプションに渡すだけです。

プラグインレベルで `logLevel` を設定すると、[`setNotFoundHandler`](Server.md#setnotfoundhandler) と [`setErrorHandler`](Server.md#seterrorhandler) も影響を受けることに注意してください。

```js
// server.js
const fastify = require('fastify')({ logger: true })

fastify.register(require('./routes/user'), { logLevel: 'warn' })
fastify.register(require('./routes/events'), { logLevel: 'debug' })

fastify.listen(3000)
```

または、ルートに直接渡します。

```js
fastify.get('/', { logLevel: 'warn' }, (request, reply) => {
  reply.send({ hello: 'world' })
})
```

*カスタムログレベルはルートにのみ適用され、`fastify.log` でアクセスできるグローバルな Fastify Logger には適用されないことを覚えておいてください。*

<a name="custom-log-serializer"></a>
### カスタムログシリアライザー

あるコンテキストでは、大きなオブジェクトをロギングする必要があるかもしれませんが、ルートによってはリソースの無駄遣いになるかもしれません。このような場合には、いくつかの [`serializer`](https://github.com/pinojs/pino/blob/master/docs/api.md#bindingsserializers-object) を定義して、適切なコンテキストにアタッチすることができます。

```js
const fastify = require('fastify')({ logger: true })

fastify.register(require('./routes/user'), {
  logSerializers: {
    user: (value) => `My serializer one - ${value.name}`
  }
})
fastify.register(require('./routes/events'), {
  logSerializers: {
    user: (value) => `My serializer two - ${value.name} ${value.surname}`
  }
})

fastify.listen(3000)
```

コンテキストごとにシリアライザを継承することができます。

```js
const fastify = Fastify({
  logger: {
    level: 'info',
    serializers: {
      user (req) {
        return {
          method: req.method,
          url: req.url,
          headers: req.headers,
          hostname: req.hostname,
          remoteAddress: req.ip,
          remotePort: req.socket.remotePort
        }
      }
    }
  }
})

fastify.register(context1, {
  logSerializers: {
    user: value => `My serializer father - ${value}`
  }
})

async function context1 (fastify, opts) {
  fastify.get('/', (req, reply) => {
    req.log.info({ user: 'call father serializer', key: 'another key' })
    // shows: { user: 'My serializer father - call father  serializer', key: 'another key' }
    reply.send({})
  })
}

fastify.listen(3000)
```

<a name="routes-config"></a>
### ルートハンドラの設定
新しいハンドラを登録すると、config オブジェクトをハンドラに渡し、それをハンドラで取得することができます。

```js
// server.js
const fastify = require('fastify')()

function handler (req, reply) {
  reply.send(reply.context.config.output)
}

fastify.get('/en', { config: { output: 'hello world!' } }, handler)
fastify.get('/it', { config: { output: 'ciao mondo!' } }, handler)

fastify.listen(3000)
```

<a name="constraints"></a>
### ルート制約

Fastify は、[`find-my-way`](https://github.com/delvedor/find-my-way) 制約や、`Host`  ヘッダーのような、リクエストのプロパティに基づいて、特定のリクエストにのみマッチするようにルートを制約することをサポートしています。制約は、ルートオプションの `constraints` プロパティで指定します。Fastify には、すぐに使える2つの組み込み制約があります: `version` 制約と `host` 制約です。また、リクエストに対してルートが実行されるべきかどうかを決定するために、リクエストの他の部分をを捜査する独自のカスタム制約を追加することができます。

#### Version Constraints

ルートの `constraints` オプションで `version` キーを指定することができます。バージョン管理されたルートでは、同じ HTTP ルートのパスに対して複数のハンドラを宣言することができ、それぞれのリクエストの `Accept-Version` ヘッダーに応じてマッチングされます。`Accept-Version` ヘッダーの値は [semver](http://semver.org/) の仕様に従うべきで、ルートはマッチングのために正確な semver のバージョンで宣言されるべきです。<br/>
Fastify は、ルートにバージョンが設定されている場合、リクエストの `Accept-Version` ヘッダーが設定されていることを要求し、同じパスに対してバージョンが設定されているルートを、バージョンが設定されていないルートよりも優先します。現在、高度なバージョン範囲やプレリリースはサポートされていません。<br/>
*この機能を使用すると、ルーターの全体的なパフォーマンスが低下することに注意してください。*


```js
fastify.route({
  method: 'GET',
  url: '/',
  { constraints: { version: '1.2.0'} },
  handler: function (request, reply) {
    reply.send({ hello: 'world' })
  }
})

fastify.inject({
  method: 'GET',
  url: '/',
  headers: {
    'Accept-Version': '1.x' // '1.2.0' や '1.2.x' も可能
  }
}, (err, res) => {
  // { hello: 'world' }
})
```

> ## ⚠  Security Notice
> キャッシュポイズニング攻撃を防ぐために、レスポンスの [`Vary`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Vary) ヘッダーに、バージョニングの定義に使用している値（例： `'Accept-Version'`）を設定することを忘れないでください。この設定は、Proxy/CDN の一部として行うこともできます。
>
> ```js
> const append = require('vary').append
> fastify.addHook('onSend', async (req, reply) => {
>   if (req.headers['accept-version']) { // または、使用しているカスタムヘッダー
>     let value = reply.getHeader('Vary') || ''
>     const header = Array.isArray(value) ? value.join(', ') : String(value)
>     if ((value = append(header, 'Accept-Version'))) { // または、使用しているカスタムヘッダー
>       reply.header('Vary', value)
>     }
>   }
> })
> ```

同じメジャーやマイナーを持つ複数のバージョンを宣言した場合、Fastify は常に `Accept-Version` ヘッダー値と互換性のある最も高いバージョンを選択します。<br/>
リクエストに`Accept-Version`ヘッダーがない場合は、404エラーが返されます。

カスタムバージョンマッチングロジックを定義することが可能です。これは、Fastify サーバーのインスタンスを作成する際に、[`constraints`](Server.md#constraints) の設定で行うことができます。

#### Host Constraints

`constraints` ルートオプションで `host` キーを指定すると、リクエストの `Host` ヘッダーの特定の値に対してのみマッチするようにルートを制限することができます。`host` 制約の値は、完全にマッチする場合は文字列として、任意のホストにマッチする場合は正規表現として指定できます。

```js
fastify.route({
  method: 'GET',
  url: '/',
  { constraints: { host: 'auth.fastify.io' } },
  handler: function (request, reply) {
    reply.send('hello world from auth.fastify.io')
  }
})

fastify.inject({
  method: 'GET',
  url: '/',
  headers: {
    'Host': 'example.com'
  }
}, (err, res) => {
  // host が制約にマッチしないので、404
})

fastify.inject({
  method: 'GET',
  url: '/',
  headers: {
    'Host': 'auth.fastify.io'
  }
}, (err, res) => {
  // => 'hello world from auth.fastify.io'
})
```

正規表現で `host` 制約を指定して、ワイルドカードのサブドメイン（またはその他のパターン）に一致するホストに制約をかけることもできます。

```js
fastify.route({
  method: 'GET',
  url: '/',
  { constraints: { host: /.*\.fastify\.io/ } }, // fastify.io の全てのサブドメインにマッチする
  handler: function (request, reply) {
    reply.send('hello world from ' + request.headers.host)
  }
})
```
