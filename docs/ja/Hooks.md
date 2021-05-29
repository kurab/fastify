<h1 align="center">Fastify</h1>

## Hooks

Hook は `fastify.addHook` メソッドで登録され、アプリケーションや Request/Response の lifecycle における特定のイベントを Listen することができます。イベントがトリガーされる前に hook を登録する必要があり、そうしないとイベントは失われます。

フックを使用することで、Fastify の lifecycle と直接対話することができます。Request/Reply hook とアプリケーション hook があります。

- [Request/Reply Hooks](#requestreply-hooks)
  - [onRequest](#onrequest)
  - [preParsing](#preparsing)
  - [preValidation](#prevalidation)
  - [preHandler](#prehandler)
  - [preSerialization](#preserialization)
  - [onError](#onerror)
  - [onSend](#onsend)
  - [onResponse](#onresponse)
  - [onTimeout](#ontimeout)
  - [Manage Errors from a hook](#manage-errors-from-a-hook)
  - [Respond to a request from a hook](#respond-to-a-request-from-a-hook)
- [Application Hooks](#application-hooks)
  - [onReady](#onready)
  - [onClose](#onclose)
  - [onRoute](#onroute)
  - [onRegister](#onregister)
- [Scope](#scope)
- [Route level hooks](#route-level-hooks)

**注意：** `async`/`await` を使用している場合や `Promise` を返している場合は、`done` コールバックは使用できません。このような状況で `done` コールバックを呼び出すと、ハンドラーの重複呼び出しなど、予期しない動作が発生する可能性があります。

## Request/Reply Hooks

[Request](Request.md) と [Reply](Reply.md) は、Fastify のコアオブジェクトです。<br>
`done` は、[lifecycle](Lifecycle.md) を継続するための機能です。

各 hook がどこで実行されるかは、[lifecycle page](Lifecycle.md) を見れば簡単に理解できます。<br>
Hook は Fastify のカプセル化の影響を受けるため、選択したルートに適用することができます。詳しくは、[Scopes](#scope) のセクションをご覧ください。

Request/Replyで使用できるフックは8種類あります *（実行順）*。

### onRequest
```js
fastify.addHook('onRequest', (request, reply, done) => {
  // Some code
  done()
})
```
Or `async/await`:
```js
fastify.addHook('onRequest', async (request, reply) => {
  // Some code
  await asyncMethod()
})
```

**Notice:** in the [onRequest](#onrequest) hook, `request.body` will always be `null`, because the body parsing happens before the [preValidation](#prevalidation) hook.
**注意:** [onRequest](#onrequest) hook では、`request.body` は常に `null` になります。なぜなら、bodyの解析は [preValidation](#prevalidation) hook の前に行われるからです。

### preParsing

`preParsing` hook 使用している場合、Request payload stream が解析される前に変換することができます。この hook は、他の hook と同様に request と reply オブジェクト、および現在の request payload を含む stream を受け取ります。

（`return` またはコールバック関数を介して）値を返す場合は、stream を返さなければなりません。

たとえば、リクエスト・ボディを解凍できます。

```js
fastify.addHook('preParsing', (request, reply, payload, done) => {
  // Some code
  done(null, newPayload)
})
```

または `async/await` で

```js
fastify.addHook('preParsing', async (request, reply, payload) => {
  // Some code
  await asyncMethod()
  return newPayload
})
```

**注意：** 
[preParsing](#preparsing) hook では、`request.body` は常に `null` になります。なぜなら、bodyの解析は [preValidation](#prevalidation) hook の前に行われるからです。

**注意：** また、返された stream に `receivedEncodedLength` プロパティを追加する必要があります。このプロパティは、Request の Payload と `Content-Length` ヘッダー値を正しく一致させるために使用されます。理想的には、このプロパティは受信した各 Chunk で更新されるべきです。

**注意:** パーサーの古い構文である `function(request, reply, done)` と `async function(request, reply)` はまだサポートされていますが、非推奨となっています。

### preValidation

`preValidation` hook を使用している場合は、検証される前に Payload を変更することができます。例えば、以下のようになります。

```js
fastify.addHook('preValidation', (request, reply, done) => {
  req.body = { ...req.body, importantKey: 'randomString' }
  done()
})
```

または `async/await` で

```js
fastify.addHook('preValidation', async (request, reply) => {
  const importantKey = await generateRandomString()
  req.body = { ...req.body, importantKey }
})
```

### preHandler
```js
fastify.addHook('preHandler', (request, reply, done) => {
  // some code
  done()
})
```

または `async/await` で

```js
fastify.addHook('preHandler', async (request, reply) => {
  // Some code
  await asyncMethod()
})
```
### preSerialization

`preSerialization` hook を使用している場合、シリアル化される前に Payload を変更（または置換）することができます。例えば、以下のようになります。

```js
fastify.addHook('preSerialization', (request, reply, payload, done) => {
  const err = null
  const newPayload = { wrapped: payload }
  done(err, newPayload)
})
```

または `async/await` で

```js
fastify.addHook('preSerialization', async (request, reply, payload) => {
  return { wrapped: payload }
})
```

**注意：** Payload が `string`、`Buffer`、`stream`、`null` の場合、hook は呼び出されません。

### onError
```js
fastify.addHook('onError', (request, reply, error, done) => {
  // Some code
  done()
})
```

または `async/await` で

```js
fastify.addHook('onError', async (request, reply, error) => {
  // カスタムエラーロギングに便利ですが、
  // エラーを更新するためにこの hook を使うべきではありません。
})
```

この hook は、カスタムのエラーロギングを行ったり、エラー時に特定のヘッダーを追加する必要がある場合に便利です。<br/>
この hook は、エラーを変更するためのものではなく、`reply.send` を呼び出すと例外が発生します。<br/>
この hook は、`customErrorHandler` が実行された後、`customErrorHandler` がエラーを返した場合にのみ実行されます。*（デフォルトの `customErrorHandler` は常にエラーを返しますのでご注意ください）*<br/>
**注意：** 他の hook と異なり、`done` 関数にエラーを渡すことはサポートされていません。

### onSend
`onSend` hook を使用している場合は、Payload を変更することができます。例えば

```js
fastify.addHook('onSend', (request, reply, payload, done) => {
  const err = null;
  const newPayload = payload.replace('some-text', 'some-new-text')
  done(err, newPayload)
})
```

または `async/await` で

```js
fastify.addHook('onSend', async (request, reply, payload) => {
  const newPayload = payload.replace('some-text', 'some-new-text')
  return newPayload
})
```

Payload をクリアして空のボディを持つ Response を送信するために、Payload を `null` に置き換えることもできます。

```js
fastify.addHook('onSend', (request, reply, payload, done) => {
  reply.code(304)
  const newPayload = null
  done(null, newPayload)
})
```

> また、Payload を空の文字列 `''` に置き換えることで、空のボディを送信することもできますが、この場合、`Content-Length` ヘッダーが `0` に設定され、Payload が `null` の場合は `Content-Length` ヘッダーが設定されないことに注意してください。

注意：Payload を変更する場合、`string`、`Buffer`、`stream`、`null` のいずれかにしか変更できません。


### onResponse
```js

fastify.addHook('onResponse', (request, reply, done) => {
  // Some code
  done()
})
```

または `async/await` で

```js
fastify.addHook('onResponse', async (request, reply) => {
  // Some code
  await asyncMethod()
})
```

`onResponse` hook は、Response が送信されたときに実行されるので、クライアントにそれ以上のデータを送信することはできません。しかし、統計データの収集など、外部サービスにデータを送信する際には便利です。

### onTimeout

```js

fastify.addHook('onTimeout', (request, reply, done) => {
  // Some code
  done()
})
```

または `async/await` で

```js
fastify.addHook('onTimeout', async (request, reply) => {
  // Some code
  await asyncMethod()
})
```
`onTimeout` は、（Fastify インスタンスに `connectionTimeout` プロパティが設定されている場合に）サービス内でタイムアウトした Request を監視する必要がある場合に便利です。`onTimeout` hook は、Request がタイムアウトし、HTTP ソケットがハングアップしたときに実行されます。したがって、クライアントにデータを送信することができません。

### Hook からのエラー管理
Hook の実行中にエラーが発生した場合、それを `done()` に渡すだけで、Fastify は自動的に Request をクローズし、適切なエラーコードをユーザーに送信します。

```js
fastify.addHook('onRequest', (request, reply, done) => {
  done(new Error('Some error'))
})
```

カスタムのエラーコードをユーザーに渡したい場合は、`reply.code()` を使用してください。

```js
fastify.addHook('preHandler', (request, reply, done) => {
  reply.code(400)
  done(new Error('Some error'))
})
```

*エラーは [`Reply`](Reply.md#errors) で処理されます。*

または、`async/await` を使用している場合は、エラーを発生させることができます。

```js
fastify.addHook('onResponse', async (request, reply) => {
  throw new Error('Some error')
})
```

### Hook からの Request への応答

必要に応じて、認証 Hook を実装する場合など、ルートハンドラに到達する前に Request に応答することができます。
Hook からの応答は、Hook チェーンが __停止__ し、残りの Hook やハンドラが実行されないことを意味します。
Hook がコールバックアプローチを使っている場合、つまり `async` 関数ではないか、`Promise` を返している場合は、 `reply.send()` を呼び出すだけで、コールバックの呼び出しを避けることができます。
Hook が `async` の場合、 `reply.send()` は関数が戻る __前__、または Promise が解決する前に呼ばれなければならず、そうでなければ Request は続行されます。
`reply.send()` が Promise チェーンの外で呼ばれた場合、 `return reply` が重要で、そうしないと Request が2回実行されることになります。

__コールバックと `async`/`Promise` を混在させないことが重要__ で、そうしないと Hook チェーンが2回実行されてしまいます。

`onRequest` または `preHandler` を使用している場合は、`reply.send` を使用してください。

```js
fastify.addHook('onRequest', (request, reply, done) => {
  reply.send('Early response')
})

// async 関数でも動作する
fastify.addHook('preHandler', async (request, reply) => {
  await something()
  reply.send({ hello: 'world' })
  return reply // このケースでは任意ですが、Good Practice です。
})
```

stream で応答したい場合は、hook に `async` 関数を使わないようにする必要があります。どうしても `async` 関数を使いたい場合は、[test/hooks-async.js](https://github.com/fastify/fastify/blob/94ea67ef2d8dce8a955d510cd9081aabd036fa85/test/hooks-async.js#L269-L275) のパターンに沿ったコードにする必要があります。

```js
fastify.addHook('onRequest', (request, reply, done) => {
  const stream = fs.createReadStream('some-file', 'utf8')
  reply.send(stream)
})
```

`await` なしに Response を送っている場合は、常に `return reply` しなければなりません。

```js
fastify.addHook('preHandler', async (request, reply) => {
  setImmediate(() => { reply.send('hello') })

  // これは、ハンドラがプロミスチェーンの外でレスポンスが送られるのを待つように
  // シグナルを送るために必要です。
  return reply
})

fastify.addHook('preHandler', async (request, reply) => {
  // fastify-static プラグインは非同期にファイルを送信するので、
  // 応答を返す必要があります。
  reply.sendFile('myfile')
  return reply
})
```

## Application Hooks

application-lifecycle にも Hook することができます。

- [onReady](#onready)
- [onClose](#onclose)
- [onRoute](#onroute)
- [onRegister](#onregister)

### onReady
サーバーが Request の Listen を開始する前に起動されます。Route の変更や新しい Hook の追加はできません。
登録された Hook 関数は連続して実行されます。
すべての `onReady` hook 関数が完了した後に、サーバは Request の受付を開始します。
Hook 関数は1つの引数をとります: Hook 関数が完了した後に呼び出されるコールバック `done` です。
Hook 関数は、関連付けられた Fastify インスタンスにバインドされた `this` で呼び出されます。

```js
// callback style
fastify.addHook('onReady', function (done) {
  // Some code
  const err = null;
  done(err)
})

// or async/await style
fastify.addHook('onReady', async function () {
  // Some async code
  await loadCacheFromDatabase()
})
```

<a name="on-close"></a>
### onClose

サーバーを停止するために `fastify.close()` が呼び出されたときにトリガーされます。[plugins](Plugins.md) が "shutdown" イベントを必要とするときに便利です。例えば、データベースン接続を閉じるときなどです。<br/>
最初の引数は Fastify インスタンスで、2番目の引数はコールバック `done` です。

```js
fastify.addHook('onClose', (instance, done) => {
  // Some code
  done()
})
```

<a name="on-route"></a>
### onRoute

新しい Route が登録されたときに起動されます。Listener には、唯一のパラメータとして `routeOptions` オブジェクトが渡されます。このインターフェイスは同期的であり、そのため Listener にはコールバックが渡されません。この Hook はカプセル化されています。

```js
fastify.addHook('onRoute', (routeOptions) => {
  //Some code
  routeOptions.method
  routeOptions.schema
  routeOptions.url // Route の完全な URL で、もしあれば prefix も含まれます。
  routeOptions.path // `url` alias
  routeOptions.routePath // prefix を除く Route の URL
  routeOptions.bodyLimit
  routeOptions.logLevel
  routeOptions.logSerializers
  routeOptions.prefix
})
```

もしプラグインを作成していて、オプションの変更や新しい Route Hook の追加など、アプリケーションの Route をカスタマイズする必要がある場合、ここでやるのが良いでしょう。

```js
fastify.addHook('onRoute', (routeOptions) => {
  function onPreSerialization(request, reply, payload, done) {
    // Your code
    done(null, payload)
  }
  // preSerializationには、array または undefined を指定できます。
  routeOptions.preSerialization = [...(routeOptions.preSerialization || []), onPreSerialization]
})
```

<a name="on-register"></a>
### onRegister

新しいプラグインが登録され、新しいカプセル化コンテキストが作成されたときに実行されます。この Hook は、登録されたコードの**前に**実行されます。<br/>
この Hook は、プラグインのコンテキストが形成されたことを知る必要があるプラグインを開発していて、その特定のコンテキストで操作したい場合に便利です。この Hook はカプセル化されます。<br/>

**注意:** この Hook は、[`fastify-plugin`](https://github.com/fastify/fastify-plugin) でラップされたプラグインでは呼び出されません。

```js
fastify.decorate('data', [])

fastify.register(async (instance, opts) => {
  instance.data.push('hello')
  console.log(instance.data) // ['hello']

  instance.register(async (instance, opts) => {
    instance.data.push('world')
    console.log(instance.data) // ['hello', 'world']
  }, { prefix: '/hola' })
}, { prefix: '/ciao' })

fastify.register(async (instance, opts) => {
  console.log(instance.data) // []
}, { prefix: '/hello' })

fastify.addHook('onRegister', (instance, opts) => {
  // 参照を保持せずに古い配列から新しい配列を作成するため、
  // ユーザは `data` プロパティのインスタンスを
  // カプセル化して持つことができます。
  instance.data = instance.data.slice()

  // 新しく登録されたインスタンスのオプション
  console.log(opts.prefix)
})
```

<a name="scope"></a>
## Scope

[onClose](#onclose) を除いて、すべての Hook はカプセル化されています。つまり、[plugins guide](Plugins-Guide.md) で説明されているように `register` を使用することで、どこで Hook を実行するかを決めることができます。関数を渡すと、その関数は正しい Fastify コンテキストにバインドされ、そこから Fastify API に完全にアクセスできるようになります。

```js
fastify.addHook('onRequest', function (request, reply, done) {
  const self = this // Fastify context
  done()
})
```

各 Hook における Fastify のコンテキストは、Route が登録されたプラグインと同じであることに注意してください。例えば、

```js
fastify.addHook('onRequest', async function (req, reply) {
  if (req.raw.url === '/nested') {
    assert.strictEqual(this.foo, 'bar')
  } else {
    assert.strictEqual(this.foo, undefined)
  }
})

fastify.get('/', async function (req, reply) {
  assert.strictEqual(this.foo, undefined)
  return { hello: 'world' }
})

fastify.register(async function plugin (fastify, opts) {
  fastify.decorate('foo', 'bar')

  fastify.get('/nested', async function (req, reply) {
    assert.strictEqual(this.foo, 'bar')
    return { hello: 'world' }
  })
})
```

警告：関数を [arrow function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) で宣言すると、`this` は Fastify ではなく、現在のスコープのものになります。

<a name="route-hooks"></a>

## Route レベルの Hook

Route に**固有**のカスタム Lifecycle Hook ([onRequest](#onrequest), [onResponse](#onresponse), [preParsing](#preparsing), [preValidation](#prevalidation), [preHandler](#prehandler), [preSerialization](#preserialization), [onSend](#onsend), [onTimeout](#ontimeout), and [onError](#onerror)) の1つまたは複数の Hook を宣言することができます。
これらの Hook は、そのカテゴリの最後の Hook として常に実行されます。<br/>
これは、認証を実装する必要がある場合に便利で、[preParsing](#preparsing) hook や [preValidation](#prevalidation) hook がまさにあなたが必要とするものです。
複数のルートレベルのフックを配列として指定することもできます。

```js
fastify.addHook('onRequest', (request, reply, done) => {
  // Your code
  done()
})

fastify.addHook('onResponse', (request, reply, done) => {
  // your code
  done()
})

fastify.addHook('preParsing', (request, reply, done) => {
  // Your code
  done()
})

fastify.addHook('preValidation', (request, reply, done) => {
  // Your code
  done()
})

fastify.addHook('preHandler', (request, reply, done) => {
  // Your code
  done()
})

fastify.addHook('preSerialization', (request, reply, payload, done) => {
  // Your code
  done(null, payload)
})

fastify.addHook('onSend', (request, reply, payload, done) => {
  // Your code
  done(null, payload)
})

fastify.addHook('onTimeout', (request, reply, done) => {
  // Your code
  done()
})

fastify.addHook('onError', (request, reply, error, done) => {
  // Your code
  done()
})

fastify.route({
  method: 'GET',
  url: '/',
  schema: { ... },
  onRequest: function (request, reply, done) {
    // この hook は、常に、共有された `onRequest` hook の後に実行されます。
    done()
  },
  onResponse: function (request, reply, done) {
    // この hook は、常に、共有された `onResponse` hook の後に実行されます。
    done()
  },
  preParsing: function (request, reply, done) {
    // この hook は、常に、共有された `preParsing` hook の後に実行されます。
    done()
  },
  preValidation: function (request, reply, done) {
    // この hook は、常に、共有された `preValidation` hook の後に実行されます。
    done()
  },
  preHandler: function (request, reply, done) {
    // この hook は、常に、共有された `preHandler` hook の後に実行されます。
    done()
  },
  // // 配列を使った例。すべての Hook がこの構文をサポートしています。
  //
  // preHandler: [function (request, reply, done) {
  //   // この hook は、常に、共有された `preHandler` hook の後に実行されます。
  //   done()
  // }],
  preSerialization: (request, reply, payload, done) => {
    // この hook は、常に、共有された `preSerialization` hook の後に実行されます。
    done(null, payload)
  },
  onSend: (request, reply, payload, done) => {
    // この hook は、常に、共有された `onSend` hook の後に実行されます。
    done(null, payload)
  },
  onTimeout: (request, reply, done) => {
    // この hook は、常に、共有された `onTimeout` hook の後に実行されます。
    done()
  },
  onError: (request, reply, error, done) => {
    // この hook は、常に、共有された `onError` hook の後に実行されます。
    done()
  },
  handler: function (request, reply) {
    reply.send({ hello: 'world' })
  }
})
```

**注意:** どちらのオプションも、関数の配列を受け取ることができます。
