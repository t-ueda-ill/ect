# ECT

CoffeeScript 構文を埋め込める、パフォーマンス重視の JavaScript テンプレートエンジン。

## インストール

```
npm install ect
```

**外部依存パッケージはありません。** CoffeeScript コンパイラはライブラリ内にバンドル済みです。

**動作要件:** Node.js >= 18.3.0

## 特徴

  * 高速なレンダリング性能
  * テンプレートのキャッシュ
  * テンプレート変更時の自動リロード
  * テンプレート内で CoffeeScript コードを記述可能
  * 複数行の式に対応
  * タグのカスタマイズに対応
  * Node.js およびクライアントサイドで動作
  * 強力かつシンプルな構文
  * テンプレートの継承・パーシャル・ブロック
  * `express` と互換性あり
  * `RequireJS` と互換性あり
  * `eco` と後方互換性あり

## 使い方

```js
var ECT = require('ect');

var renderer = ECT({ root : __dirname + '/views', ext : '.ect' });
var html = renderer.render('page', { title: 'Hello, World!' });
```

コールバック形式も使用できます。

```js
var ECT = require('ect');

var renderer = ECT({ root : __dirname + '/views', ext : '.ect' });

renderer.render('page', { title: 'Hello, World!' }, function (error, html) {
	console.log(error);
	console.log(html);
});
```

JavaScript オブジェクトをテンプレートのルートとして使用することもできます。

```js
var ECT = require('ect');

var renderer = ECT({ root : {
				layout: '<html><head><title><%- @title %></title></head><body><% content %></body></html>',
				page: '<% extend "layout" %><p>ページコンテンツ</p>'
				}
			});

var html = renderer.render('page', { title: 'Hello, World!' });
```

### Express との連携

app.js
```js
var express = require('express');
var app = express();
var ECT = require('ect');
var ectRenderer = ECT({ watch: true, root: __dirname + '/views', ext : '.ect' });

app.set('view engine', 'ect');
app.engine('ect', ectRenderer.render);

app.get('/', function (req, res){
	res.render('index');
});

app.listen(3000);
console.log('Listening on port 3000');
```

views/index.ect
```html
<% extend 'layout' %>
<% include 'extra' %>
<div>Hello, World!</div>
```

views/extra.ect
```html
<div>Include me!</div>
```

views/layout.ect
```html
<html>
	<body>
		<% content %>
	</body>
</html>
```

## 構文

### エスケープなしの出力

```
<%- someVar %>
```

### エスケープ済み出力

```
<%= someVar %>
```

### CoffeeScript コード

```
<% for article in @articles : %>
	<% include 'article', article %>
<% end %>
```

条件分岐の例:

```
<% if @user?.authenticated : %>
	<% include 'partials/user' %>
<% else : %>
	<% include 'partials/auth' %>
<% end %>
```

### テンプレートの継承

子テンプレートで親テンプレートを指定します。

```
<% extend 'layout' %>
```

親テンプレートでは、子の挿入位置を次のように定義します。

```
<% content %>
```

### パーシャル

```
<% include 'partial' %>
```

パーシャルにデータコンテキストを渡すこともできます。

```
<% include 'partial', { customVar: 'Hello, World!' } %>
```

### ブロック

```
<% block 'blockName' : %>
	<p>ブロックの内容</p>
<% end %>
```

親テンプレートでは、ブロックの挿入位置を次のように定義します。

```
<% content 'blockName' %>
```

ブロックは複数レベルの継承に対応しており、再定義も可能です。

## オプション

### レンダラー

  - `root` — テンプレートのルートフォルダ、または JavaScript オブジェクト
  - `ext` — テンプレートの拡張子（デフォルト: `''`、オブジェクトルートの場合は不使用）
  - `cache` — コンパイル済み関数のキャッシュ（デフォルト: `true`）
  - `watch` — テンプレート変更時の自動リロード（デフォルト: `false`、デバッグ時に有用、クライアントサイドでは非対応）
  - `open` — 開始タグ（デフォルト: `<%`）
  - `close` — 閉じタグ（デフォルト: `%>`）

### コンパイラーミドルウェア

  - `root` — ベース URL（デフォルト: `/`、クライアント側の `root` オプションと同じ値にすること）
  - `gzip` — gzip によるテンプレート圧縮（デフォルト: `false`）

## CLI ツール

ECT にはテンプレートをプリコンパイルする CLI ツールが付属しています。

```
ect source [destination]
```

### オプション

  - `-o`, `--open` — 開始タグ（デフォルト: `<%`）
  - `-c`, `--close` — 閉じタグ（デフォルト: `%>`）
  - `-h`, `--help` — ヘルプを表示

### 使用例

```bash
# 標準出力にコンパイル結果を表示
ect template.ect

# ファイルに出力
ect template.ect compiled.js
```

## クライアントサイドでの使用

[coffeescript.js](https://github.com/jashkenas/coffeescript) と [ect.min.js](https://github.com/baryshev/ect/tree/master/ect.min.js) をダウンロードして読み込みます。

```html
<script src="/path/coffeescript.js"></script>
<script src="/path/ect.min.js"></script>
```

使用例:

```js
var renderer = ECT({ root : '/views' });
var data = { title : 'Hello, World!' };
var html = renderer.render('template.ect', data);
```

### サーバーサイドコンパイラーミドルウェアとの併用

[ect.min.js](https://github.com/baryshev/ect/tree/master/ect.min.js) のみダウンロードして読み込みます。サーバー側のコンパイラーミドルウェアがテンプレートをコンパイル済みで配信するため、CoffeeScript コンパイラは不要です。

```html
<script src="/path/ect.min.js"></script>
```

サーバー側の設定:

```js
var connect = require('connect');
var ECT = require('ect');

var renderer = ECT({ root : __dirname + '/views', ext : '.ect' });

var app = connect()
	.use(renderer.compiler({ root: '/views', gzip: true }))
	.use(function(err, req, res, next) {
		res.end(err.message);
	});

app.listen(3000);
```

クライアント側の使用例:

```js
var renderer = ECT({ root : '/views', ext : '.ect' });
var data = { title : 'Hello, World!' };
var html = renderer.render('template', data);
```

注意: クロスドメイン制約を避けるため、ルートフォルダは同一ドメイン上に配置してください。

## ライセンス

(The MIT License)

Copyright (c) 2012 Vadim M. Baryshev &lt;vadimbaryshev@gmail.com&gt;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
