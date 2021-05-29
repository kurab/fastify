<h1 align="center">Fastify</h1>

## Logging

Logging はデフォルトでは無効になっていますが、 `{ logger: true }` または `{ logger: { level: 'info' } }` を渡すことで有効にできます。なお、Logger が無効になっている場合、実行時に有効にすることはできません。この目的のために [abstract-logging](https://www.npmjs.com/package/abstract-logging) を使用しています。

Fastify はパフォーマンスに重点を置いているため、ロガーとして [pino](https://github.com/pinojs/pino) を使用し、有効な場合のデフォルトのログレベルは `'info'` に設定されています。

Logger を有効にするのは超簡単です。

```js
const fastify = require('fastify')({
  logger: true
})

fastify.get('/', options, function (request, reply) {
  request.log.info('Some info about the current request')
  reply.send({ hello: 'world' })
})
```

Fastify インスタンスから Pino インスタンスを使用することで、ルートハンドラの外側で新しいログをトリガーすることができます。

```js
fastify.log.info('Something important happened!');
```

いくつかのオプションをロガーに渡したい場合は、Fastify に渡すだけで良いです。すべての利用可能なオプションは、 [Pino documentation](https://github.com/pinojs/pino/blob/master/docs/api.md#pinooptions-stream) をご確認ください。もし、ファイルの保存先を指定したい場合は、以下のようにします。

```js
const fastify = require('fastify')({
  logger: {
    level: 'info',
    file: '/path/to/file' // pino.destination() が使われる
  }
})

fastify.get('/', options, function (request, reply) {
  request.log.info('Some info about the current request')
  reply.send({ hello: 'world' })
})
```

Pino のインスタンスにカスタムストリームを渡したい場合は、logger オブジェクトに stream フィールドを追加するだけです。

```js
const split = require('split2')
const stream = split(JSON.parse)

const fastify = require('fastify')({
  logger: {
    level: 'info',
    stream: stream
  }
})
```

<a name="logging-request-id"></a>

デフォルトでは、Fastify はトラッキングを容易にするために、各リクエストに ID を追加します。もし、"request-id" ヘッダーが存在すれば、その値が使用され、そうでなければ、新しいインクリメンタル ID が生成されます。カスタマイズオプションについては、Fastify Factory の [`requestIdHeader`](Server.md#factory-request-id-header) と [`genReqId`](Server.md#genreqid) を参照してください。

デフォルトの Logger は、`req`、`res`、`err` プロパティでオブジェクトをシリアライズする、標準シリアライザのセットで構成されています。`req` で受け取るオブジェクトは Fastify [`Request`](Request.md) オブジェクトで、`res` で受け取るオブジェクトは Fastify [`Reply`](Reply.md) オブジェクトです。
この動作は、カスタムシリアライザを指定することでカスタマイズできます。


```js
const fastify = require('fastify')({
  logger: {
    serializers: {
      req (request) {
        return { url: request.url }
      }
    }
  }
})
```

例えば、以下のような方法でレスポンスのペイロードとヘッダーをログに記録することができます（*推奨されていませんが*）。

```js
const fastify = require('fastify')({
  logger: {
    prettyPrint: true,
    serializers: {
      res (reply) {
        // The default
        return {
          statusCode: reply.statusCode
        }
      },
      req (request) {
        return {
          method: request.method,
          url: request.url,
          path: request.path,
          parameters: request.parameters,
          // ログにヘッダーを含めることは、GDPRなどのプライバシー法に違反する可能性があります。
          // "redact" オプションを使用して、機密性の高いフィールドを削除する必要があります。
          // ログに認証データが漏れる可能性があります。
          headers: request.headers
        };
      }
    }
  }
});
```

**注意**： 子ロガーを作成する際にリクエストがシリアル化されるため、`req` メソッド内でボディをシリアル化することはできません。その時点では、ボディはまだ解析されていません。

`req.body` のログを取る方法を見てみましょう。

```js
app.addHook('preHandler', function (req, reply, done) {
  if (req.body) {
    req.log.info({ body: req.body }, 'parsed body')
  }
  done()
})
```


*Pino 以外のロガーは、このオプションを無視します。*

独自のロガーインスタンスを提供することもできます。設定オプションを渡すのではなく、インスタンスを渡します。提供するロガーはPinoインターフェースに準拠していなければなりません。つまり、以下のメソッドを持っていなければなりません。: `info`、`error`、`debug`、`fatal`、`warn`、`trace`、`child`

例:

```js
const log = require('pino')({ level: 'info' })
const fastify = require('fastify')({ logger: log })

log.info('リクエスト情報を持っていない')

fastify.get('/', function (request, reply) {
  request.log.info('リクエスト情報を持っているが、`log` と同じ Logger インスタンス')
  reply.send({ hello: 'world' })
})
```

*現在のリクエストのロガーインスタンスは、ライフサイクル [lifecycle](Lifecycle.md) のあらゆる部分で利用可能です。*

## Log Redaction

[Pino](https://getpino.io) は、記録されたログの特定のプロパティの値を不明瞭にする、オーバーヘッドの少ないログリダクションをサポートしています。
例えば、セキュリティの観点から、HTTP ヘッダから `Authorization` ヘッダを除いたすべてのヘッダを記録したい場合があります。

```js
const fastify = Fastify({
  logger: {
    stream: stream,
    redact: ['req.headers.authorization'],
    level: 'info',
    serializers: {
      req (request) {
        return {
          method: request.method,
          url: request.url,
          headers: request.headers,
          hostname: request.hostname,
          remoteAddress: request.ip,
          remotePort: request.socket.remotePort
        }
      }
    }
  }
})
```

詳細は https://getpino.io/#/docs/redaction を参照
