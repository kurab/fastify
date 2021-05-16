<h1 align="center">Fastify</h1>

## `Content-Type` パーサー
Fastify はデフォルトでは `'application/json'` と `'text/plain'` コンテンツタイプしかサポートしていません。デフォルトの文字セットは `utf-8` です。異なるコンテンツタイプをサポートする必要がある場合は、`addContentTypeParser` API を使用できます。*デフォルトの JSON パーサーやプレーンテキストパーサーは変更可能です。

他の API と同様に、`addContentTypeParser` は宣言されたスコープにカプセル化されます。つまり、root スコープで宣言した場合はどこでも利用できますが、プラグインの中で宣言した場合は、そのスコープとその子でのみ利用できるということです。

Fastify は自動的に、パースされたリクエスト・ペイロードを [Fastify request](Request.md) オブジェクトに追加します。このオブジェクトは `request.body` でアクセスできます。

### 利用方法
```js
fastify.addContentTypeParser('application/jsoff', function (request, payload, done) {
  jsoffParser(payload, function (err, body) {
    done(err, body)
  })
})

// 複数の Content Type を処理可能
fastify.addContentTypeParser(['text/xml', 'application/xml'], function (request, payload, done) {
  xmlParser(payload, function (err, body) {
    done(err, body)
  })
})

// Node.js ver 8.0.0 以降、Async をサポート
fastify.addContentTypeParser('application/jsoff', async function (request, payload) {
  var res = await jsoffParserAsync(payload)

  return res
})

// 正規表現も使える
fastify.addContentTypeParser(/^image\/.*/, function (request, payload, done) {
  imageParser(payload, function (err, body) {
    done(err, body)
  })
})

// デフォルトの JSON/Text パーサーを他の Content Type に使える
fastify.addContentTypeParser('text/json', { parseAs: 'string' }, fastify.getDefaultJsonParser('ignore', 'ignore'))
```

Fastify はマッチする `RegExp` で探そうとする前に、まず content-type パーサーを `string` 値とマッチさせようとします。
content-type が重複して提供されている場合、Fastify は渡されたのが新しい順にマッチする content-type を見つけようとします。
そのため、一般的な content-type をより正確に指定したい場合は、以下の例のように、まず一般的な Content-type を指定し、次により具体的な Content-type を指定します。

```js
// 2番目の Content-Type パーサーの値が1番目のものとマッチするため、2番めの Content-type パーサーしか呼び出されない
fastify.addContentTypeParser('application/vnd.custom+xml', (request, body, done) => {} )
fastify.addContentTypeParser('application/vnd.custom', (request, body, done) => {} )

// Fastify が最初に `application/vnd.custom+xml` Content-type パーサーにマッチしようとするため、期待した動作が得られる
fastify.addContentTypeParser('application/vnd.custom', (request, body, done) => {} )
fastify.addContentTypeParser('application/vnd.custom+xml', (request, body, done) => {} )
```

また、`hasContentTypeParser` API を使って、特定の Content-type パーサーが既に存在するかどうかを調べることもできます。

```js
if (!fastify.hasContentTypeParser('application/jsoff')){
  fastify.addContentTypeParser('application/jsoff', function (request, payload, done) {
    jsoffParser(payload, function (err, body) {
      done(err, body)
    })
  })
}
```

**注意**: パーサーの古い構文である `function(req, done)` や `async function(req)` はまだサポートされていますが、非推奨となっています。

#### Body パーサー
リクエストのボディを解析するには2つの方法があります。最初の方法は上に示した通りで、カスタム Content-type パーサーを追加して、リクエストを処理します。2つ目の方法では、`addContentTypeParser` API に `parseAs` オプションを渡して、どのようにボディを取得したいかを宣言します。`parseAs` オプションを使用すると、Fastify は内部的にリクエストを処理して、ボディの[最大サイズ](Server.md#factory-body-limit)やコンテンツの長さなど、いくつかのチェックを行います。制限を超えた場合、カスタムパーサーは呼び出されません。

```js
fastify.addContentTypeParser('application/json', { parseAs: 'string' }, function (req, body, done) {
  try {
    var json = JSON.parse(body)
    done(null, json)
  } catch (err) {
    err.statusCode = 400
    done(err, undefined)
  }
})
```

例として、[`example/parser.js`](../../examples/parser.js)をご覧ください。

##### カスタムパーサーオプション
+ `parseAs` (string): `'string'`または `'buffer'`のいずれかで、入力データをどのように収集するかを指定します。デフォルトは `'buffer'` です。
+ `bodyLimit` (number): カスタムパーサーが受け付けるペイロードの最大サイズをバイト単位で指定します。デフォルトでは、[`Fastify factory function`](Server.md#bodylimit) に渡されるグロールの制限値になります。

#### Catch-All
Content-type に関係なく、すべてのリクエストをキャッチしなければならないケースがあります。Fastify では、`'*'`コンテンツタイプを使用することができます。
```js
fastify.addContentTypeParser('*', function (request, payload, done) {
  var data = ''
  payload.on('data', chunk => { data += chunk })
  payload.on('end', () => {
    done(null, data)
  })
})
```

この方法では、対応する Content-type パーサーを持たないすべてのリクエストが、指定した関数によって処理されます。

これは、リクエストをパイプする際にも便利です。以下のように Content-type パーサーを定義することができます。

```js
fastify.addContentTypeParser('*', function (request, payload, done) {
  done()
})
```

そして、コアの HTTP リクエストにアクセスして、直接パイプすることができます:

```js
app.post('/hello', (request, reply) => {
  reply.send(request.raw)
})
```

受信した [json line](https://jsonlines.org/) オブジェクトをログに記録する例を示します。

```js
const split2 = require('split2')
const pump = require('pump')

fastify.addContentTypeParser('*', (request, payload, done) => {
  done(null, pump(payload, split2(JSON.parse)))
})

fastify.route({
  method: 'POST',
  url: '/api/log/jsons',
  handler: (req, res) => {
    req.body.on('data', d => console.log(d)) // log every incoming object
  }
})
 ```

ファイルのアップロードのパイプラインには、[このプラグイン](https://github.com/fastify/fastify-multipart)がお勧めです。
