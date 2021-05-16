<h1 align="center">Fastify</h1>

## ベンチマーク
ベンチマークは、ある変更がアプリケーションのパフォーマンスにどのような影響を与えるかを測定したい場合に重要です。私たちは、ユーザーやコントリビューターの視点からアプリケーションをベンチマークする簡単な方法を提供します。このセットアップでは、異なるブランチや異なる Node.js のバージョンでベンチマークを自動化することができます。

使用するモジュール:
- [Autocannon](https://github.com/mcollina/autocannon): Node.js で書かれた HTTP/1.1 ベンチマークツール
- [Branch-comparer](https://github.com/StarpTech/branch-comparer): 複数のgitブランチをチェックアウトし、スクリプトを実行し、結果をログに記録
- [Concurrently](https://github.com/kimmobrunfeldt/concurrently): コマンドを並列して実行
- [Npx](https://github.com/npm/npx): 異なる Node.js バージョンに対してスクリプトを実行したり、 ローカルバイナリを実行するために使われる NPM パッケージランナー。npm@5.2.0 に同梱

## Simple

### 現在のブランチでテストを実行
```sh
npm run benchmark
```

### 異なるNode.jsのバージョンでテストを実行 ✨
```sh
npx -p node@10 -- npm run benchmark
```

## Advanced

### 異なるブランチでテストを実行
```sh
branchcmp --rounds 2 --script "npm run benchmark"
```

### 異なるブランチで、異なるNode.jsのバージョンに対してテストを実行 ✨
```sh
branchcmp --rounds 2 --script "npm run benchmark"
```

### 現在のブランチとメインブランチの比較（Gitflow）
```sh
branchcmp --rounds 2 --gitflow --script "npm run benchmark"
```
or
```sh
npm run bench
```

### 異なる example の実行

```sh
branchcmp --rounds 2 -s "node ./node_modules/concurrently -k -s first \"node ./examples/asyncawait.js\" \"node ./node_modules/autocannon -c 100 -d 5 -p 10 localhost:3000/\""
```
