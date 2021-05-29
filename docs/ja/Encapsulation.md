<h1 align="center">Fastify</h1>

<a id="encapsulation"></a>
## カプセル化

Fastify の基本的な機能である "カプセル化コンテキスト "です。カプセル化コンテキストは [routes](./Routes.md) で、どの [decorators](./Decorators.md) が使えるか、どんな [hooks](./Hooks.md)、および [plugins](./Plugins.md) が登録されているかを管理します。カプセル化コンテキストの例を図で表すと以下のようになります。

![Figure 1](../resources/encapsulation_context.svg)

上の図には、いくつかのエンティティがあります。

1. _親コンテキスト - Root Context_
2. 3つの _親プラグイン - Root Plugin_
3. それぞれ以下を持つ2つの _子コンテキスト - Child Context_
    * 2つの _子プラグイン - Child Plugin_
    * 3つの _子プラグイン_ を持つ1つの _孫コンテキスト - Grandchild Context_

すべての _子コンテキスト_ と _孫コンテキスト_ は _親プラグイン_ にアクセスできます。各 _子コンテキスト_ の中で _孫コンテキスト_ は、その _子コンテキスト_ に登録されている _子プラグイン_ にアクセスできますが、その _子コンテキスト_ は、その _孫コンテキスト_ に登録されている _子プラグイン_ にはアクセス**できません**。

Fastify では、_親コンテキスト_ を除くすべてが [plugin](./Plugins.md) であることを考えると、この例の全ての "コンテキスト"と"プラグイン"は、decorator、hook、plugin そして route で構成できるプラグインになります。
したがって、この例を具体的に説明するために、3つの route を持つ REST API サーバーの基本的なシナリオを考えてみましょう。
1番目の route（`/one`）は認証を必要とし、2番目の route（`/two`）は認証を必要とせず、3番目の route（/three）は2番目の route と同じコンテキストにアクセスします。
認証を行うために [fastify-bearer-auth][bearer] を使用し、この例のコードは次のようになります。

```js
'use strict'

const fastify = require('fastify')()

fastify.decorateRequest('answer', 42)

fastify.register(async function authenticatedContext (childServer) {
  childServer.register(require('fastify-bearer-auth'), { keys: ['abc123'] })

  childServer.route({
    path: '/one',
    method: 'GET',
    handler (request, response) {
      response.send({
        answer: request.answer,
        // request.fooはpublicContextでのみ定義されているため、未定義となります。
        foo: request.foo,
        // request.barはgrandchildContextでのみ定義されているため、未定義となります。
        bar: request.bar
      })
    }
  })
})

fastify.register(async function publicContext (childServer) {
  childServer.decorateRequest('foo', 'foo')

  childServer.route({
    path: '/two',
    method: 'GET',
    handler (request, response) {
      response.send({
        answer: request.answer,
        foo: request.foo,
        // request.barはgrandchildContextでのみ定義されているため、未定義となります。
        bar: request.bar
      })
    }
  })

  childServer.register(async function grandchildContext (grandchildServer) {
    grandchildServer.decorateRequest('bar', 'bar')

    grandchildServer.route({
      path: '/three',
      method: 'GET',
      handler (request, response) {
        response.send({
          answer: request.answer,
          foo: request.foo,
          bar: request.bar
        })
      }
    })
  })
})

fastify.listen(8000)
```

上記のサーバーの例では、元の図で説明したカプセル化の概念をすべて示しています。

1. 各 _子コンテキスト_ (`authenticatedContext`、`publicContext` と `grandchildContext`) は、_親コンテキスト_ で定義された `answer` リクエストデコレータにアクセスできます。
2. `authenticatedContext` のみが `fastify-bearer-auth` プラグインへアクセスできます。
3. `publicContext` と `grandchildContext` の両方が、`foo` リクエストデコレータにアクセスできます。
4. `grandchildContext` だけが、`bar` リクエストデコレータへアクセスできます。

これを確認するには、サーバーを起動してリクエストを発行します。

```sh
# curl -H 'authorization: Bearer abc123' http://127.0.0.1:8000/one
{"answer":42}
# curl http://127.0.0.1:8000/two
{"answer":42,"foo":"foo"}
# curl http://127.0.0.1:8000/three
{"answer":42,"foo":"foo","bar":"bar"}
```

[bearer]: https://github.com/fastify/fastify-bearer-auth

<a id="shared-context"></a>
## Sharing Between Contexts
## コンテキスト間の共有

前述の例の各コンテキストは、親コンテキストから _のみ_ 継承されることに注意してください。親コンテキストは、子・孫コンテキスト内のエンティティにアクセスできません。
このデフォルトの状態が望まれないこともあります。
そのような場合には、 [fastify-plugin](fastify-plugin) を使用してカプセル化コンテキストを解除し、子・孫コンテキストに登録されているものを親コンテキストで使用できるようにすることができます。

前述の例で `publicContext` が `grandchildContext` 内で定義された `bar` デコレータにアクセスする必要があると仮定すると、コードは次のように書き換えられます。

```js
'use strict'

const fastify = require('fastify')()
const fastifyPlugin = require('fastify-plugin')

fastify.decorateRequest('answer', 42)

// わかりやすくするために、`authenticatedContext` を省略しました。

fastify.register(async function publicContext (childServer) {
  childServer.decorateRequest('foo', 'foo')

  childServer.route({
    path: '/two',
    method: 'GET',
    handler (request, response) {
      response.send({
        answer: request.answer,
        foo: request.foo,
        bar: request.bar
      })
    }
  })

  childServer.register(fastifyPlugin(grandchildContext))

  async function grandchildContext (grandchildServer) {
    grandchildServer.decorateRequest('bar', 'bar')

    grandchildServer.route({
      path: '/three',
      method: 'GET',
      handler (request, response) {
        response.send({
          answer: request.answer,
          foo: request.foo,
          bar: request.bar
        })
      }
    })
  }
})

fastify.listen(8000)
```

サーバーを再起動し、`/two` と `/three` に再度リクエストを発行します。

```sh
# curl http://127.0.0.1:8000/two
{"answer":42,"foo":"foo","bar":"bar"}
# curl http://127.0.0.1:8000/three
{"answer":42,"foo":"foo","bar":"bar"}
```

fastify-plugin: https://github.com/fastify/fastify-plugin
