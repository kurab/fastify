<h1 align="center">Fastify</h1>

<a name="factory"></a>
## Factory

Fastify モジュールは、新しい <code><b>Fastify サーバー</b></code>インスタンスを作成するために使用されるファクトリ関数をエクスポートします。このファクトリ関数は、結果のインスタンスをカスタマイズするために使用されるオプションオブジェクトを受け入れます。このドキュメントでは、そのオプションオブジェクトで利用可能なプロパティについて説明します。

<a name="factory-http2"></a>
### `http2`

`true` の場合、Node.js のコアの [HTTP/2](https://nodejs.org/dist/latest-v8.x/docs/api/http2.html) モジュールがソケットのバインドに使われます。

+ Default: `false`

<a name="factory-https"></a>
### `https`

TLS用のサーバのリスニングソケットを設定するためのオブジェクトです。オプションは、Node.js コアの [`createServer` method](https://nodejs.org/dist/latest-v8.x/docs/api/https.html#https_https_createserver_options_requestlistener) と同様です。
このプロパティが `null` の場合、ソケットは TSL 用に設定されません。

このオプションは [http2](./Server.md#factory-http2) が設定されている場合にも適用されます。

+ Default: `null`

<a name="factory-connection-timeout"></a>
### `connectionTimeout`

サーバーのタイムアウトをミリ秒単位で定義します。このオプションの効果については、[`server.timeout` property](https://nodejs.org/api/http.html#http_server_timeout)のドキュメントを参照してください。serverFactory` オプションが指定されている場合、このオプションは無視されます。

+ Default: `0` (タイムアウトしない)

<a name="factory-keep-alive-timeout"></a>
### `keepAliveTimeout`

サーバーのタイムアウトをミリ秒単位で定義します。このオプションの効果を理解するために、[`server.keepAliveTimeout` property](https://nodejs.org/api/http.html#http_server_keepalivetimeout) を参照してください。このオプションは、HTTP/1
が使用されている場合にのみ適用されます。また、`serverFactory` オプションが指定されている場合、このオプションは無視されます。

+ Default: `5000` (5 秒)

<a name="factory-ignore-slash"></a>
### `ignoreTrailingSlash`

Fastify は [find-my-way](https://github.com/delvedor/find-my-way) を使ってルーティングを行います。このオプションを `true` にすると、トレイリング・スラッシュを無視することができます。このオプションは、生成されるサーバーインスタンスの*すべて*のルート登録に適用されます。

+ Default: `false`

```js
const fastify = require('fastify')({
  ignoreTrailingSlash: true
})

// "/foo" と "/foo/" が登録される
fastify.get('/foo/', function (req, reply) {
  reply.send('foo')
})

// "/bar" と "/bar/" が登録される
fastify.get('/bar', function (req, reply) {
  reply.send('bar')
})
```

<a name="factory-max-param-length"></a>
### `maxParamLength`
`maxParamLength` オプションを使うことで、パラメトリック（標準、正規表現、マルチの）ルートのパラメータに独自の長さを設定することができます、デフォルト値は100文字です。<br>
これは、特に正規表現ベースのルートを使用している場合に便利で、[DoS攻撃](https://www.owasp.org/index.php/Regular_expression_Denial_of_Service_-_ReDoS) から保護することができます。<br>
*制限に達した場合は、not found ルートが起動します。*

<a name="factory-body-limit"></a>
### `bodyLimit`

サーバーが受け入れることのできる最大のペイロードをバイト単位で定義します。

+ Default: `1048576` (1MiB)

<a name="factory-on-proto-poisoning"></a>
### `onProtoPoisoning`

JSON オブジェクトを `__proto__` で解析する際に、フレームワークがどのようなアクションを取らなければならないかを定義します。
この機能は [secure-json-parse](https://github.com/fastify/secure-json-parse) で提供されています。
プロパティ・ポイズニングのより詳しい情報は、 https://hueniverse.com/a-tale-of-prototype-poisoning-2610fa170061 を参照してください。

取り得る値は、`'error'`、`'remove'`、`'ignore'` です。

+ Default: `'error'`

<a name="factory-on-constructor-poisoning"></a>
### `onConstructorPoisoning`

JSONオブジェクトを `constructor` で解析する際に、フレームワークがどのようなアクションを取らなければならないかを定義します。
この機能は [secure-json-parse](https://github.com/fastify/secure-json-parse) で提供されています。
プロトタイプポイズニングについての詳細は https://hueniverse.com/a-tale-of-prototype-poisoning-2610fa170061 を参照してください。

取り得る値は、`'error'`、`'remove'`、`'ignore'` です。

+ Default: `'ignore'`

<a name="factory-logger"></a>
### `logger`

Fastify は、[Pino](https://getpino.io/) ロガーを介したビルトインロギングを含みます。
このプロパティは、内部ロガーインスタンスを構成するために使用されます。

このプロパティが取り得る値は以下の通りです。

+ Default: `false`. ロガーが無効になります。すべてのロギングメソッドは
null logger [abstract-logging](https://npm.im/abstract-logging)インスタンスを指します。

+ `pinoInstance`: 事前にインスタンス化されたPinoのインスタンスです。内部ロガーはこのインスタンスを指します。

+ `object`: 標準の Pino [options object](https://github.com/pinojs/pino/blob/c77d8ec5ce/docs/API.md#constructor) です。
これはPinoのコンストラクタに直接渡されます。もし、オブジェクトに以下のプロパティが存在しない場合は、適宜追加されます。
    * `level`: 最小のログレベルを設定します。設定されていない場合は、`'info'`に設定されます。
    * `serializers`: シリアル化関数のハッシュです。デフォルトでは、`req`（送信するリクエストオブジェクト）、`res`（受信するレスポンスオブジェクト）、`err`（標準の`Error`オブジェクト）にシリアライザが追加されます。これらのプロパティのいずれかを持つオブジェクトをログメソッドが受け取ると、そのプロパティに対してそれぞれのシリアライザが使用されます。
        ```js
        fastify.get('/foo', function (req, res) {
          req.log.info({req}) // log the serialized request object
          res.send('foo')
        })
        ```
      ユーザーが提供したシリアライザは、対応するプロパティのデフォルトのシリアライザを上書きします。
+ `loggerInstance`: カスタムロガーインスタンスです。ロガーは Pino インターフェースに準拠していなければなりません: `info`、`error`、`debug`、`fatal`、`warn`、`trace`、`child`
  ```js
  const pino = require('pino')();

  const customLogger = {
    info: function (o, ...n) {},
    warn: function (o, ...n) {},
    error: function (o, ...n) {},
    fatal: function (o, ...n) {},
    trace: function (o, ...n) {},
    debug: function (o, ...n) {},
    child: function() {
      const child = Object.create(this);
      child.pino = pino.child(...arguments);
      return child;
    },
  };

  const fastify = require('fastify')({logger: customLogger});
  ```

<a name="factory-disable-request-logging"></a>
### `disableRequestLogging`

デフォルトで、ロギングが有効になっている場合、Fastify は、リクエストを受信したときと、そのリクエストに対するレスポンスを受信したときに、`info` レベルのログメッセージを発行します。このオプションを `true` に設定することで、これらのログメッセージは無効になります。カスタムの `onRequest` と `onResponse` フックをアタッチすることで、より柔軟なリクエストの開始と終了のロギングが可能になります。

+ Default: `false`

```js
// 無効化された機能を再現するためのフックの例
fastify.addHook('onRequest', (req, reply, done) => {
  req.log.info({ url: req.raw.url, id: req.id }, 'received request')
  done()
})

fastify.addHook('onResponse', (req, reply, done) => {
  req.log.info({ url: req.raw.originalUrl, statusCode: reply.raw.statusCode }, 'request completed')
  done()
})
```

この設定を行うと、デフォルトの `onResponse` フックが返信コールバックのエラー時に書き込むエラーログも無効になるのでご注意ください。

<a name="custom-http-server"></a>
### `serverFactory`
カスタム HTTP サーバを Fastify に渡すには、`serverFactory` オプションを使用します。<br/>
`serverFactory` は、`request` と `response` オブジェクトをパラメータとして持つ `handler` パラメータと、Fastify に渡したのと同じオプションオブジェクトを取る関数です。

```js
const serverFactory = (handler, opts) => {
  const server = http.createServer((req, res) => {
    handler(req, res)
  })

  return server
}

const fastify = Fastify({ serverFactory })

fastify.get('/', (req, reply) => {
  reply.send({ hello: 'world' })
})

fastify.listen(3000)
```

内部的に、Fastify は Node コアの HTTP サーバの API を使用しているので、カスタムサーバを使用している場合は、同様の API が用意されている必要があります。もしそうでなければ、`serverFactory` 関数の中で、`return` 文の前に、サーバーインスタンスを拡張します。<br/>

<a name="factory-case-sensitive"></a>
### `caseSensitive`

デフォルトでは、`true` になっており、ルートは大文字小文字を区別して登録されます。つまり、`/foo` は `/Foo` と同等ではないということです。`false` に設定すると、`/foo` が `/FOO` と区別されない形でルートが登録されます。

`caseSensitive` を `false` に設定すると、全てのパスは小文字でマッチしますが、ルートのパラメータやワイルドカードは元の大文字小文字を維持します。

```js
fastify.get('/user/:username', (request, reply) => {
  // Given the URL: /USER/NodeJS
  console.log(request.params.username) // -> 'NodeJS'
})
```

この設定を `false` にすることは [RFC3986](https://tools.ietf.org/html/rfc3986#section-6.2.2.1) に反することに注意してください。

<a name="factory-request-id-header"></a>
### `requestIdHeader`

`request-id` を知るために使用されるヘッダー名です。[the request-id](Logging.md#logging-request-id)のセクションを参照してください。

+ Default: `'request-id'`

<a name="factory-request-id-log-label"></a>
### `requestIdLogLabel`

リクエストのログを取る際に、リクエストの識別子として使用するラベルを定義します。

+ Default: `'reqId'`

<a name="factory-gen-request-id"></a>
### `genReqId`

`request-id` を生成する関数です。受信したリクエストをパラメータとして受け取ります。

+ Default: `提供されている場合は'request-id'ヘッダの値、または単調に増加する整数値`

特に分散システムでは、以下のようにデフォルトの ID 生成をオーバーライドしたい場合があります。なお、`UUID` の生成については、[hyperid](https://github.com/mcollina/hyperid) が参考になります。

```js
let i = 0
const fastify = require('fastify')({
  genReqId: function (req) { return i++ }
})
```

*注意：genReqIdは、<code>[requestIdHeader](#requestidheader)</code> で設定されたヘッダが利用可能な場合は呼び出されません（デフォルトは'request-id'）*

<a name="factory-trust-proxy"></a>
### `trustProxy`

`trustProxy` オプションを有効にすると、Fastify はプロキシを利用しており、`X-Forwarded-*` ヘッダーフィールドが信頼できるものとします。そうでなければ簡単に偽装される可能性があります。

```js
const fastify = Fastify({ trustProxy: true })
```

+ Default: `false`
+ `true/false`: 全てのプロキシを信頼するか (`true`) 、全て信頼しないか (`false`)
+ `string`: 指定された IP/CIDR のみを信頼します (例: `'127.0.0.1'`)。コンマ区切りのリストにすることもできます (例: `'127.0.0.1,192.168.1.1/24'`)
+ `Array<string>`: 信頼する IP/CIDR リストを配列で指定することもできます (例: `['127.0.0.1']`)
+ `number`: Front-Facing プロキシの n番目のホップをクライアントとして信頼します
+ `Function`: `address` を第一引数にとるカスタム関数
    ```js
    function myTrustFn(address, hop) {
      return address === '1.2.3.4' || hop === 1
    }
    ```

詳細は [`@fastify/proxy-addr`](https://www.npmjs.com/package/@fastify/proxy-addr) パッケージを参照してください。

[`request`](Request.md)オブジェクトの `ip`, `ips`, `hostname`, `protocol` の値にアクセスすることができます。

```js
fastify.get('/', (request, reply) => {
  console.log(request.ip)
  console.log(request.ips)
  console.log(request.hostname)
  console.log(request.protocol)
})
```

*注意：リクエストに複数の <code>x-forwarded-host</code> または <code>x-forwarded-proto</code> ヘッダーが含まれている場合、<code>request.hostname</code> および <code>request.protocol</code> の抽出に使用されるのは、最後のものだけです。*

<a name="plugin-timeout"></a>
### `pluginTimeout`

プラグインをロードする最大時間 *ミリ秒* を指定します。指定しない場合、[`ready`](Server.md#ready) は `Error` コード `'ERR_AVVIO_PLUGIN_TIMEOUT'` で終了します。

+ Default: `10000`

<a name="factory-querystring-parser"></a>
### `querystringParser`

Fastify が使用するデフォルトの queryString パーサーは、Node.js のコア `querystring` モジュールです<br/>。
オプションの `querystringParser` を渡すことで、このデフォルト設定を変更し、[`qs`](https://www.npmjs.com/package/qs) のようなカスタムしたものを使用することができます。

```js
const qs = require('qs')
const fastify = require('fastify')({
  querystringParser: str => qs.parse(str)
})
```

<a name="exposeHeadRoutes"></a>
### `exposeHeadRoutes`

定義された各 `GET` ルートに対して、`HEAD` ルートを自動的に作成します。このオプションを無効にせずにカスタムの `HEAD` ハンドラを作成したい場合は、必ず `GET` ルートの前に定義してください。

+ Default: `false`

<a name="constraints"></a>
### `constraints`

Fastify に組み込まれたルート制約は、`find-my-way` によって提供され、`version` または `host` によってルートを制約することができます。新しい制約を追加したり、`constraints` オブジェクトに `find-my-way` 用の制約を与えることで、組み込みの制約をオーバーライドすることができます。制約条件の詳細については、[find-my-way](https://github.com/delvedor/find-my-way) のドキュメントを参照してください。

```js
const customVersionStrategy = {
  storage: function () {
    let versions = {}
    return {
      get: (version) => { return versions[version] || null },
      set: (version, store) => { versions[version] = store },
      del: (version) => { delete versions[version] },
      empty: () => { versions = {} }
    }
  },
  deriveVersion: (req, ctx) => {
    return req.headers['accept']
  }
}

const fastify = require('fastify')({
  constraints: {
    version: customVersionStrategy
  }
})
```

<a name="factory-return-503-on-closing"></a>
### `return503OnClosing`

`close` サーバーメソッドを呼び出した後に503を返します。
`false` の場合、サーバーは通常通りに受信したリクエストをルーティングします。

+ Default: `true`

<a name="factory-ajv"></a>
### `ajv`

Fastify で使用される Ajv インスタンスを設定します（カスタムインスタンスを提供する必要はありません）。

+ Default:

```js
{
  customOptions: {
    removeAdditional: true,
    useDefaults: true,
    coerceTypes: true,
    allErrors: false,
    nullable: true
  },
  plugins: []
}
```

```js
const fastify = require('fastify')({
  ajv: {
    customOptions: {
      nullable: false // [ajv options](https://ajv.js.org/#options) 参照
    },
    plugins: [
      require('ajv-merge-patch')
      [require('ajv-keywords'), 'instanceof'];
      // Usage: [plugin, pluginOptions] - オプション付きプラグイン
      // Usage: plugin - オプションなしプラグイン
    ]
  }
})
```

<a name="http2-session-timeout"></a>
### `http2SessionTimeout`

[timeout](https://nodejs.org/api/http2.html#http2_http2session_settimeout_msecs_callback) のデフォルト値を受信する全ての HTTP/2 セッションに設定します。タイムアウトになると、セッションは閉じられます。

+ Default: `5000` ms.

これは、HTTP/2 を利用する際に graceful "close" を提供するために必要です。Node コアのデフォルト値は `0` です。

<a name="framework-errors"></a>
### `frameworkErrors`

+ Default: `null`

Fastify は、最も一般的なユースケースのために、デフォルトのエラーハンドラを提供します。
このオプションを使うと、カスタムコードでこれらのハンドラの1つまたは複数をオーバーライドすることができます。

*注意：現時点では `FST_ERR_BAD_URL` のみ実装されています。*

```js
const fastify = require('fastify')({
  frameworkErrors: function (error, req, res) {
    if (error instanceof FST_ERR_BAD_URL) {
      res.code(400)
      return res.send("Provided url is not valid")
    } else {
      res.send(err)
    }
  }
})
```

<a name="client-error-handler"></a>
### `clientErrorHandler`

[clientErrorHandler](https://nodejs.org/api/http.html#http_event_clienterror) を設定し、クライアント接続から発せられる `error` イベントを取得してステータスコード `400` を返します。

このオプションを使うことで、`clientErrorHandler` をオーバーライドすることができます。

+ Default:
```js
function defaultClientErrorHandler (err, socket) {
  if (err.code === 'ECONNRESET') {
    return
  }

  const body = JSON.stringify({
    error: http.STATUS_CODES['400'],
    message: 'Client Error',
    statusCode: 400
  })
  this.log.trace({ err }, 'client error')

  if (socket.writable) {
    socket.end(`HTTP/1.1 400 Bad Request\r\nContent-Length: ${body.length}\r\nContent-Type: application/json\r\n\r\n${body}`)
  }
}
```

*注意: `clientErrorHandler` は生のソケットで動作します。ハンドラは、ステータス、HTTP ヘッダ、メッセージボディを含む、適切に形成された HTTP レスポンスを返すことが期待されます。ハンドラは、ソケットの書き込みを試みる前に、そのソケットが壊れている可能性があるため、書き込み可能かどうかを確認する必要があります。*

```js
const fastify = require('fastify')({
  clientErrorHandler: function (err, socket) {
    const body = JSON.stringify({
      error: {
        message: 'Client error',
        code: '400'
      }
    })

    // `this` は Fastify インスタンスにバインドされている
    this.log.trace({ err }, 'client error')

    // ハンドラは、有効な HTTP レスポンスを生成する責任がある
    socket.end(`HTTP/1.1 400 Bad Request\r\nContent-Length: ${body.length}\r\nContent-Type: application/json\r\n\r\n${body}`)
  }
})
```

<a name="rewrite-url"></a>
### `rewriteUrl`

リライト URL を受け付ける文字列を返す同期コールバック関数を設定します。

> URL をリライトすると、`req`オブジェクトの`url`プロパティが変更される

```js
function rewriteUrl (req) { // req は Node.js の HTTP リクエスト
  return req.url === '/hi' ? '/hello' : req.url;
}
```

`rewriteUrl` はルーティングの _前_ に呼び出されるため、カプセル化されておらず、インスタンス全体の設定であることに注意してください。

## Instance

### Server Methods

<a name="server"></a>
#### server
`fastify.server`: [**`Fastify factory function`**](Server.md) によって返された Node コアの [server](https://nodejs.org/api/http.html#http_class_http_server) モジュールです。

<a name="after"></a>
#### after
現在のプラグインと、その中に登録されているすべてのプラグインのロードが終了したときに呼び出されます。このメソッドは、常に `fastify.ready` の前に実行されます。

```js
fastify
  .register((instance, opts, done) => {
    console.log('Current plugin')
    done()
  })
  .after(err => {
    console.log('After current plugin')
  })
  .register((instance, opts, done) => {
    console.log('Next plugin')
    done()
  })
  .ready(err => {
    console.log('Everything has been loaded')
  })
```

関数を指定せずに `after()` が呼ばれた場合は、`Promise` を返します。

```js
fastify.register(async (instance, opts) => {
  console.log('Current plugin')
})

await fastify.after()
console.log('After current plugin')

fastify.register(async (instance, opts) => {
  console.log('Next plugin')
})

await fastify.ready()

console.log('Everything has been loaded')
```

<a name="ready"></a>
#### ready
すべてのプラグインがロードされたときに呼び出される関数です。何か問題が発生した場合は、エラーパラメータを受け取ります。

```js
fastify.ready(err => {
  if (err) throw err
})
```

引数なしで呼び出された場合は、`Promise` を返します。

```js
fastify.ready().then(() => {
  console.log('successfully booted!')
}, (err) => {
  console.log('an error happened', err)
})
```

<a name="listen"></a>
#### listen
全てのプラグインがロードされた後、与えられたポートでサーバを起動し、内部的には `.ready()` イベントを待ちます。コールバックは、Nodeコアと同じです。
デフォルトでは、特定のアドレスが提供されていない場合、サーバーは `localhost` によって解決されたアドレスをリッスンします (OSによっては `127.0.0.1` または `::1`). 
利用可能なすべてのインターフェースをリッスンしたい場合は、アドレスに `0.0.0.0` を指定すると、すべての IPv4 アドレスをリッスンします。
`::` を指定すると、すべての IPv6 アドレスをリッスンし、OSによっては、すべての IPv4 アドレスをリッスンします。
すべてのインターフェイスを聞く際には、固有の [security risks](https://web.archive.org/web/20170831174611/https://snyk.io/blog/mongodb-hack-and-secure-defaults/) が伴うため、注意が必要です。


```js
fastify.listen(3000, (err, address) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

アドレスの指定にも対応しています。

```js
fastify.listen(3000, '127.0.0.1', (err, address) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

また、バックログのキューサイズの指定も可能です。

```js
fastify.listen(3000, '127.0.0.1', 511, (err, address) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

オプションの指定もサポートされており、そのオブジェクトは、Node.js のサーバーリッスンの [options](https://nodejs.org/api/net.html#net_server_listen_options_callback) と同じです。

```js
fastify.listen({ port: 3000, host: '127.0.0.1', backlog: 511 }, (err) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

コールバックが指定されていない場合は、`Promise` が返されます。

```js
fastify.listen(3000)
  .then((address) => console.log(`server listening on ${address}`))
  .catch(err => {
    console.log('Error starting server:', err)
    process.exit(1)
  })
```

コールバックのないアドレスの指定もサポートされています。

```js
fastify.listen(3000, '127.0.0.1')
  .then((address) => console.log(`server listening on ${address}`))
  .catch(err => {
    console.log('Error starting server:', err)
    process.exit(1)
  })
```

コールバックのないオプションの指定もサポートされています。

```js
fastify.listen({ port: 3000, host: '127.0.0.1', backlog: 511 })
  .then((address) => console.log(`server listening on ${address}`))
  .catch(err => {
    console.log('Error starting server:', err)
    process.exit(1)
  })
```

Docker やその他のコンテナにデプロイする際には、`0.0.0.0` をリッスンすることをお勧めします。なぜなら、コンテナはデフォルトではマップされたポートを `localhost` に公開しないからです。

```js
fastify.listen(3000, '0.0.0.0', (err, address) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

リッスンのデフォルトのオプションは、以下の通りです。

```js
fastify.listen({
  port: 0,
  host: 'localhost',
  exclusive: false,
  readableAll: false,
  writableAll: false,
  ipv6Only: false
}, (err) => {})
```

<a name="getDefaultRoute"></a>
#### getDefaultRoute
サーバーの `defaultRoute` を取得するメソッドです。

```js
const defaultRoute = fastify.getDefaultRoute()
```

<a name="setDefaultRoute"></a>
#### setDefaultRoute
サーバーの `defaultRoute` を設定するメソッドです。

```js
const defaultRoute = function (req, res) {
  res.end('hello world')
}

fastify.setDefaultRoute(defaultRoute)
```

<a name="routing"></a>
#### routing
内部ルーターの `lookup` メソッドにアクセスして、リクエストを適切なハンドラにマッチさせるメソッドです。

```js
fastify.routing(req, res)
```

<a name="route"></a>
#### route
サーバーにルートを追加するためのメソッドで、省略可能な関数もあります。[こちら](Routes.md)を確認してください。

<a name="close"></a>
#### close
`fastify.close(callback)`: この関数を呼び出してサーバーインスタンスを閉じ、[`'onClose'`](Hooks.md#on-close) フックを実行します。<br>
また、`close` をコールすると、サーバーは新しく入ってくるリクエストに対して、`503` エラーで応答し、そのリクエストを破棄します。
この動作を変更するには [`return503OnClosing` flags](Server.md#factory-return-503-on-closing) を参照してください。

引数なしで呼び出された場合は、`Promise` を返します。

```js
fastify.close().then(() => {
  console.log('successfully closed!')
}, (err) => {
  console.log('an error happened', err)
})
```

<a name="decorate"></a>
#### decorate*
Fastify のインスタンス、Reply や Request をデコレーションする必要がある場合に便利な機能です。詳しくは、[こちら](Decorators.md) を確認してください。

<a name="register"></a>
#### register
Fastify では、ユーザーがプラグインで機能を拡張することができます。
プラグインは、ルートのセットやサーバーデコレーターなどができます。詳しくは[こちら](Plugins.md)を参照してください。

<a name="addHook"></a>
#### addHook
Fastify のライフサイクルに特定のフックを追加する機能です。[こちら](Hooks.md)を確認してください。

<a name="prefix"></a>
#### prefix
ルートの前に付けられるパスです。

例:

```js
fastify.register(function (instance, opts, done) {
  instance.get('/foo', function (request, reply) {
    // Will log "prefix: /v1"
    request.log.info('prefix: %s', instance.prefix)
    reply.send({ prefix: instance.prefix })
  })

  instance.register(function (instance, opts, done) {
    instance.get('/bar', function (request, reply) {
      // Will log "prefix: /v1/v2"
      request.log.info('prefix: %s', instance.prefix)
      reply.send({ prefix: instance.prefix })
    })

    done()
  }, { prefix: '/v2' })

  done()
}, { prefix: '/v1' })
```

<a name="pluginName"></a>
#### pluginName
現在のプラグインの名前。名前を定義するには順番に3つの方法があります。

1. [fastify-plugin](https://github.com/fastify/fastify-plugin) を使っている場合、メタデータの `name` が使われます。
2. `module.exports` プラグインの場合、ファイル名が使われます。
3. 通常の [function declaration](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions#Defining_functions) の場合、関数名が使われます。

*Fallback*: プラグインの最初の2行は、プラグイン名を表します。改行は ` -- ` に置き換えられます。これにより、多くのプラグインを扱っている場合、根本的な原因を特定するのに役立ちます。

重要: ネストしたプラグインを扱う必要がある場合、[fastify-plugin](https://github.com/fastify/fastify-plugin) を使った場合とは名前が異なります。新しいスコープが作成されないため、コンテキストデータを添付する場所がないからです。この場合、プラグイン名は、関係するすべてのプラグインの起動順を表し、`plugin-A -> plugin-B`という形式になります。

<a name="log"></a>
#### log
ロガーインスタンスです。詳しくは [こちら](Logging.md)。

<a name="version"></a>
#### version
インスタンスの Fastify バージョン。プラグインのサポートに使用されます。バージョンがプラグインでどのように使われるかについては、[Plugins](Plugins.md#handle-the-scope) を参照してください。

<a name="inject"></a>
#### inject
フェイクの HTTP インジェクション（テスト用）です。詳しくは、 [こちら](Testing.md#inject)。

<a name="add-schema"></a>
#### addSchema
`fastify.addSchema(schemaObj)` は、Fastify インスタンスにJSONスキーマを追加します。これにより、標準の `$ref` キーワードを使用するだけで、アプリケーションのあらゆる場所で再利用することができます。<br/>。
さらに詳しく知りたい方は、[Validation and Serialization](Validation-and-Serialization.md) ドキュメントをご覧ください。

<a name="get-schemas"></a>
#### getSchemas
`fastify.getSchemas()` は、`.addSchema` で追加されたすべてのスキーマのハッシュを返します。ハッシュのキーは、JSON スキーマが提供する `$id` です。

<a name="get-schema"></a>
#### getSchema
`fastify.getSchema(id)` では、`.addSchema` で追加した JSON スキーマと、それにマッチする`id` を返します。見つからない場合は `undefined` を返します。

<a name="set-reply-serializer"></a>
#### setReplySerializer
すべてのルートに返信シリアライザを設定します。[Reply.serializer(func)](Reply.md#serializerfunc) が設定されていない場合、デフォルトで使用されます。ハンドラは完全にカプセル化されているので、プラグインごとに異なるエラーハンドラを設定することができます。
注意：この関数パラメータは、ステータスが `2xx` の場合にのみ呼び出されます。エラーについては、[`setErrorHandler`](Server.md#seterrorhandler)を確認してください。

```js
fastify.setReplySerializer(function (payload, statusCode){
  // serialize the payload with a sync function
  return `my serialized ${statusCode} content: ${payload}`
})
```

<a name="set-validator-compiler"></a>
#### setValidatorCompiler

すべてのルートにスキーマバリデータのコンパイラを設定します。[#schema-validator](Validation-and-Serialization.md#schema-validator).

<a name="set-schema-error-formatter"></a>
#### setSchemaErrorFormatter

すべてのルートにスキーマエラーフォーマッタを設定します。[#error-handling](Validation-and-Serialization.md#schemaerrorformatter).

<a name="set-serializer-resolver"></a>
#### setSerializerCompiler

すべてのルートにスキーマ・シリアライザー・コンパイラを設定します。[#schema-serializer](Validation-and-Serialization.md#schema-serializer).
**注意:** [`setReplySerializer`](#set-reply-serializer) が設定されていると、優先されます！

<a name="validator-compiler"></a>
#### validatorCompiler

このプロパティはスキーマバリデータを取得するために使用します。設定されていない場合は、サーバーが起動するまでは `null` となり、その後は `function ({ schema, method, url, httpPart })` というシグネチャを持つ関数となり、データを検証するための関数にコンパイルされた入力 `schema` を返します。
このインプットスキーマは、[`.addSchema`](#add-schema) 関数で追加されたすべての共有スキーマにアクセスすることができます。

<a name="serializer-compiler"></a>
#### serializerCompiler

このプロパティを使ってスキーマ・シリアライザを取得することができます。設定されていない場合は、サーバーが起動するまでは `null` となり、その後は `function ({ schema, method, url, httpPart })` というシグネチャを持つ関数となり、入力された `schema` をデータを検証するための関数にコンパイルして返します。
この入力スキーマは、[`.addSchema`](#add-schema) 関数で追加されたすべての共有スキーマにアクセスすることができます。

<a name="schema-error-formatter"></a>
#### schemaErrorFormatter

このプロパティは、`validationCompiler`がスキーマの検証に失敗したときに発生するエラーをフォーマットする関数を設定するために使用できます。[#error-handling](Validation-and-Serialization.md#schemaerrorformatter) 参照。

<a name="schema-controller"></a>
#### schemaController

このプロパティは、アプリケーションのスキーマが保存される場所を完全に管理するために使用できます。
スキーマが Fastify に知られていない別のデータ構造に保存されている場合に役立ちます。
このプロパティが解決に役立つ例として、[issue #2446](https://github.com/fastify/fastify/issues/2446) を参照してください。

```js
const fastify = Fastify({
  schemaController: {
    /**
     * この factory は `fastify.register()` が呼ばれるときに必ず呼ばれる
     * スキーマが追加されている場合は、親コンテキストのスキーマを入力として受け取ることができます。
     * @param {object} parentSchemas これらのスキーマは、返された `bucket` の `getSchemas()` メソッド関数によって返されます。
     */
    bucket: function factory (parentSchemas) {
      return {
        addSchema (inputSchema) {
          // この関数は、ユーザが追加したスキーマを格納しなければなりません。
          // この関数は、`fastify.addSchema()`が呼ばれたときに呼び出されます。
        },
        getSchema (schema$id) {
          // この関数は、`schema$id` で要求された生のスキーマを返さなければなりません。
          // この関数は、`fastify.getSchema(id)`が呼ばれたときに呼び出されます。
          return aSchema
        },
        getSchemas () {
          // This function must return all the schemas referenced by the routes schemas' $ref
          // この関数は、ルートスキーマの $ref で参照されるすべてのスキーマを返さなければなりません。
          // プロパティがスキーマ `$id` で、値が生の JSON スキーマである JSON を返す必要があります。
          const allTheSchemaStored = {
            'schema$id1': schema1,
            'schema$id2': schema2
          }
          return allTheSchemaStored
        }
      }
    }
  }
});
```

<a name="set-not-found-handler"></a>
#### setNotFoundHandler

`fastify.setNotFoundHandler(handler(request, reply))`: 404ハンドラを設定します。この呼び出しは prefix によってカプセル化されているので、異なる[`prefix` option](Plugins.md#rout-prefixing-option) が `fastify.register()` に渡された場合、プラグインごとに異なる not found ハンドラを設定することができます。ハンドラは通常のルートハンドラとして扱われるので、リクエストは [Fastify lifecycle](Lifecycle.md#lifecycle) を全て通過します。

404 ハンドラとして [`preValidation`](https://www.fastify.io/docs/latest/Hooks/#route-hooks) や [`preHandler`](https://www.fastify.io/docs/latest/Hooks/#route-hooks) フックを設定できます。

注意: このメソッドを使って登録された`preValidation`フックは、Fastifyが認識していないルートに対して実行され、ルートハンドラが手動で[`reply.callNotFound`](Reply.md#call-not-found)を呼び出した場合には実行され **ません**。その場合は、preHandlerのみが実行されます。


```js
fastify.setNotFoundHandler({
  preValidation: (req, reply, done) => {
    // your code
    done()
  },
  preHandler: (req, reply, done) => {
    // your code
    done()
  }
}, function (request, reply) {
    // preValidationとpreHandlerフックを持つデフォルトのnot foundハンドラ
})

fastify.register(function (instance, options, done) {
  instance.setNotFoundHandler(function (request, reply) {
    // '/v1'で始まるURLにpreValidationとpreHandlerのフックを使わずに
    // not foundリクエストを処理する。
  })
  done()
}, { prefix: '/v1' })
```

<a name="set-error-handler"></a>
#### setErrorHandler

`fastify.setErrorHandler(handler(error, request, reply))`: エラーが発生したときに呼び出される関数を設定します。ハンドラは Fastify のインスタンスにバインドされ、完全にカプセル化されているので、ラグインごとに異なるエラーハンドラを設定することができます。<br> 
*async-await* もサポートされています。

*注意：`statusCode` が 400 未満の場合、Fastify はエラーハンドラを呼び出す前に自動的に 500 に設定します。*


```js
fastify.setErrorHandler(function (error, request, reply) {
  // Log error
  this.log.error(error)
  // Send error response
  reply.status(409).send({ ok: false })
})
```

Fastify には、エラーハンドラが設定されていない場合に呼び出されるデフォルト関数が用意されています。これは、`fastify.errorHandler` を使ってアクセスでき、`statusCode` に基づいてエラーをログに記録します。

```js
var statusCode = error.statusCode
if (statusCode >= 500) {
  log.error(error)
} else if (statusCode >= 400) {
  log.info(error)
} else {
  log.error(error)
}
```

<a name="print-routes"></a>
#### printRoutes

`fastify.printRoutes()`: ルータが使用する内部基数ツリーの表現を出力します。これはデバッグに役立ちます。別の方法として、`fastify.printRoutes({ commonPrefix: false })` を使うと、フラット化されたルートツリーを表示することができます。

*`ready`の中か後で呼び出してください。*


```js
fastify.get('/test', () => {})
fastify.get('/test/hello', () => {})
fastify.get('/hello/world', () => {})
fastify.get('/helicopter', () => {})

fastify.ready(() => {
  console.log(fastify.printRoutes())
  // └── /
  //     ├── test (GET)
  //     │   └── /hello (GET)
  //     └── hel
  //         ├── lo/world (GET)
  //         └── licopter (GET)

  console.log(fastify.printRoutes({ commonPrefix: false }))
  // └── / (-)
  //     ├── test (GET)
  //     │   └── /hello (GET)
  //     ├── hello/world (GET)
  //     └── helicopter (GET)
  
})
```

<a name="print-plugins"></a>
#### printPlugins

avvio が使用する内部プラグインツリーを出力します。順序が重要なも際のデバッグに便利です。<br/>
*`ready`の中か後で呼び出してください。*

```js
fastify.register(async function foo (instance) {
  instance.register(async function bar () {})
})
fastify.register(async function baz () {})

fastify.ready(() => {
  console.error(fastify.printPlugins())
  // will output the following to stderr:
  // └── root
  //   ├── foo
  //   │   └── bar
  //   └── baz
})
```

<a name="addContentTypeParser"></a>
#### addContentTypeParser

`fastify.addContentTypeParser(content-type, options, parser)` は、与えられたContent type にカスタムパーサを渡すために使われます。例えば、`text/json, application/vnd.oasis.opendocument.text` のような、カスタム Content type のパーサを追加するのに便利です。`content-type` には、文字列、文字列配列、または正規表現を指定します。

```js
// getDefaultJsonParserに渡される2つの引数は、それぞれProtoTypeポイゾニングとConstructorポイゾニングの設定です。ignoreはすべての検証をスキップし、JSON.parse()を直接呼び出すのと同様の動作をします。詳しくは、<a href="https://github.com/fastify/secure-json-parse#api">`secure-JSON-parse` documentation</a>をご覧ください。

fastify.addContentTypeParser('text/json', { asString: true }, fastify.getDefaultJsonParser('ignore', 'ignore'))
```

<a name="getDefaultJsonParser"></a>
#### getDefaultJsonParser

`fastify.getDefaultJsonParser(onProtoPoisoning, onConstructorPoisoning)` は2つの引数を取ります。最初の引数は ProtoType ポイズニングの設定で、2番目の引数はコンストラクタポイズニングの設定です。詳しくは、[`secure-JSON-parse` documentation](https://github.com/fastify/secure-json-parse#api) をご覧ください。

<a name="defaultTextParser"></a>
#### defaultTextParser

`fastify.defaultTextParser()`を使って、コンテンツをプレーンテキストとして解析することができます。

```js
fastify.addContentTypeParser('text/json', { asString: true }, fastify.defaultTextParser())
```

<a name="errorHandler"></a>
#### errorHandler

`fastify.errorHandler` は、Fastify のデフォルトのエラーハンドラを使ってエラーを処理するために使用できます。

```js
fastify.get('/', {
  errorHandler: (error, request, reply) => {
    if (error.code === 'SOMETHING_SPECIFIC') {
      reply.send({ custom: 'response' })
      return
    }

    fastify.errorHandler(error, request, response)
  }
}, handler)
```

<a name="initial-config"></a>
#### initialConfig

`fastify.initialConfig`: ユーザーが Fastify インスタンスに渡した初期オプションを登録する、凍結したリードオンリーオブジェクトを公開します。

現在、公開可能なプロパティは以下の通りです。
- connectionTimeout
- keepAliveTimeout
- bodyLimit
- caseSensitive
- http2
- https (明示的に渡された場合は、`false`/`true` または `{ allowHTTP1: true/false }` を返します)
- ignoreTrailingSlash
- disableRequestLogging
- maxParamLength
- onProtoPoisoning
- onConstructorPoisoning
- pluginTimeout
- requestIdHeader
- requestIdLogLabel
- http2SessionTimeout

```js
const { readFileSync } = require('fs')
const Fastify = require('fastify')

const fastify = Fastify({
  https: {
    allowHTTP1: true,
    key: readFileSync('./fastify.key'),
    cert: readFileSync('./fastify.cert')
  },
  logger: { level: 'trace'},
  ignoreTrailingSlash: true,
  maxParamLength: 200,
  caseSensitive: true,
  trustProxy: '127.0.0.1,192.168.1.1/24',
})

console.log(fastify.initialConfig)
/*
will log :
{
  caseSensitive: true,
  https: { allowHTTP1: true },
  ignoreTrailingSlash: true,
  maxParamLength: 200
}
*/

fastify.register(async (instance, opts) => {
  instance.get('/', async (request, reply) => {
    return instance.initialConfig
    /*
    will return :
    {
      caseSensitive: true,
      https: { allowHTTP1: true },
      ignoreTrailingSlash: true,
      maxParamLength: 200
    }
    */
  })

  instance.get('/error', async (request, reply) => {
    // initialConfig が読み取り専用であり、変更できないため、エラーが発生します。
    instance.initialConfig.https.allowHTTP1 = false

    return instance.initialConfig
  })
})

// Start listening.
fastify.listen(3000, (err) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```
