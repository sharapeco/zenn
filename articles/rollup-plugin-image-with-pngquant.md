---
title: "Rollupのplugin-imageにpngquantを組み込む"
emoji: "🗜️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["rollup", "pngquant"]
published: true
---

# はじめに

Rollupのプラグイン [@rollup/plugin-image](https://www.npmjs.com/package/@rollup/plugin-image) を使うと、画像ファイルをデータURLとしてスクリプトに含めることができる。これは、小さな画像ファイルをいくつか使用したライブラリなどを作成する際、ネットワークリクエストを減らすために有用だ。

例えば次のようにして使うことができる。

`src/index.js`:
```js
import logo from './rollup.png';

console.log(logo);
// => "data:image/png;base64,..."
```

`rollup.config.js`:
```js
import image from '@rollup/plugin-image';

export default {
  input: 'src/index.js',
  output: {
    dir: 'output',
    format: 'cjs'
  },
  plugins: [image()]
};
```

しかしながらこのプラグインには画像を最適化する機能がなく、画像を最適化するには別のツールが必要で少々面倒だ。そこで、この記事では @rollup/plugin-image と同等の動作をさせつつ、pngquantによる画像最適化を組み込んだプラグインを作成する方法を紹介する。

# プラグインの作成

まず [@rollup/plugin-image のソース](https://github.com/rollup/plugins/blob/master/packages/image/src/index.js) をコピーして `rollup/image-with-pngquant.js` として保存する。

必要なパッケージをインストールする。@rollup/plugin-image をインストールしておけば依存関係の解決が楽。

```shell-session
$ npm install --save-dev @rollup/plugin-image pngquant
```

## pngquant の準備

Pngquantを読み込み、オプションを格納する定数を用意しておく。

```js
import PngQuant from "pngquant";

const pngOptions = {
	speed: 1,
	quality: "65-80",
};
```

## load フックを非同期にする

Pngquantを使うためにはストリームを使い、非同期処理にする必要がある。[Plugin Development | Rollup](https://rollupjs.org/plugin-development/#build-hooks) には次のように書かれており、Promiseを返してもよいことになっている。

> `async`: The hook may also return a Promise resolving to the same type of value; otherwise, the hook is marked as `sync`.

```js
export default function image(opts = {}) {
	const options = Object.assign({}, defaults, opts);
	const filter = createFilter(options.include, options.exclude);

	return {
		name: "image",

		load(id) {
			return new Promise((resolve, reject) => {
				if (!filter(id)) {
					return resolve(null);
				}

				const mime = mimeTypes[extname(id)];
				if (!mime) {
					// not an image
					return resolve(null);
				}

				...

				resolve(code.trim());
			});
		},
	};
}
```

## ストリームの出力をデータURLとして取り出す

ストリームをデータURLとして取り出すためには、次の手順で行う。

1. ストリームを `Buffer` として取り出すためのクラスを作る
2. `Buffer.toString("base64")` を使ってbase64にエンコードする

### ストリームを Buffer として取り出すためのクラス

ストリームの出力を取り出すためには `Stream.Writable` を継承したクラスを作り、`_write` メソッドを実装する。`_write` メソッドはストリームからデータを受け取るたびに呼ばれるため、毎回データを保存しておく。

```js
import { Stream } from "node:stream";

class BufferStream extends Stream.Writable {
	constructor() {
		super();
		this._buffers = [];
	}

	_write(chunk, encoding, callback) {
		this._buffers.push(chunk);
		callback();
	}

	get buffer() {
		return Buffer.concat(this._buffers);
	}
}
```

ストリームの出力が終わった際に `finish` イベントが発生するので、そこで `buffer` プロパティを参照してデータを取り出せる。

```js
const destination = new BufferStream();
destination.on("finish", () => {
	console.log(destination.buffer);
});
source.pipe(destination);
```

## ストリームを繋いで画像を最適化する

入力ファイルは `fs.readFileSync()` の代わりに `fs.createReadStream()` を使ってストリームとして読み込む。ストリームの出力を取り出すためのクラスを作ったので、あとはこれらを繋ぐだけだ。

PNG画像の場合は次のコードが実行されるようにする。

```js
const source = createReadStream(id);
const destination = new BufferStream();

destination.on("finish", () => {
	const dataUri = getDataUri({
		format: "base64"
		isSvg: false,
		mime,
		buffer: destination.buffer.toString("base64"),
	});
	const code = options.dom
		? domTemplate({ dataUri })
		: constTemplate({ dataUri });
	resolve(code.trim());
});

destination.on("error", (err) => {
	reject(err);
});

source
	.pipe(
		new PngQuant([
			"--speed",
			pngOptions.speed,
			"--quality",
			pngOptions.quality,
			"--nofs",
			"-",
		]),
	)
	.pipe(destination);
```

# プラグインを置き換える

最後に `rollup-plugin-image` をimportする部分を作成したプラグインに置き換える。

`rollup.config.js`:

```js
import image from "./rollup/image-with-pngquant.js";
```
