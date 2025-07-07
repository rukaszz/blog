---
layout: post
title:  "Rust + WASM環境構築メモ：「Hello world!」を表示するまで"
date:   2025-07-04T18:25:52-05:00
author: rukaszz
categories: [rust]
tags: [environment, rust]
permalink: /posts/rust
---

# Rust + WebAssembly本

## 概要

参考書籍：RustとWebAssemblyによるゲーム開発(第1版)

### 目的

上記の参考書籍に従って環境構築をしていたら詰まったので，解消方法を備忘録的なメモとして記述する．

## 設定についての注意点

### 書籍の設定

プロジェクトの初期化：

```Bash
mkdir myProject
cd myProject
npm init rust-webpack
```

実行：

```Bash
npm run start
```

書籍に載っているエラーが生じたので，書籍の対処法を実施：

```Bash
curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh
```

エラーの対処は次節で解説する．

Rustエディションの更新については，次のようにCargo.tomlを編集した．

```toml
# You must change these to your own details.
[package]
name = "rust-webpack-template"
description = "My super awesome Rust, WebAssembly, and Webpack project!"
version = "0.1.0"
authors = ["You <you@example.com>"]
categories = ["wasm"]
readme = "README.md"
edition = "2021"

[lib]
crate-type = ["cdylib"]

[profile.release]
# This makes the compiled code faster and smaller, but it makes compiling slower,
# so it's only enabled in release mode.
lto = true

[features]
# If you uncomment this line, it will enable `wee_alloc`:
#default = ["wee_alloc"]

[dependencies]
# The `wasm-bindgen` crate provides the bare minimum functionality needed
# to interact with JavaScript.
wasm-bindgen = "0.2.78"

# `wee_alloc` is a tiny allocator for wasm that is only ~1K in code size
# compared to the default allocator's ~10K. However, it is slower than the default
# allocator, so it's not enabled by default.
wee_alloc = { version = "0.4.2", optional = true }

# The `web-sys` crate allows you to interact with the various browser APIs,
# like the DOM.
[dependencies.web-sys]
version = "0.3.55"
features = ["console"]

# The `console_error_panic_hook` crate provides better debugging of panics by
# logging them with `console.error`. This is great for development, but requires
# all the `std::fmt` and `std::panicking` infrastructure, so it's only enabled
# in debug mode.
[target."cfg(debug_assertions)".dependencies]
console_error_panic_hook = "0.1.7"

# These crates are used for running unit tests.
[dev-dependencies]
wasm-bindgen-test = "0.3.28"
futures = "0.3.18"
js-sys = "0.3.55"
wasm-bindgen-futures = "0.4.28"
```

### エラーへの対処：npm環境整理

```Bash
npm run start
```

書籍に従って上記コマンドを実行してもコンパイルに失敗する(2025/07/05現在)．
原因としては，まず私の環境の問題であった．npmだけでは不完全で，nvmによるバージョン管理をしてnpmなどnode.jsを最新化して修復した．

```Bash
~/myProject$ curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.4/install.sh | bash
=> Downloading nvm from git to '/home/.nvm'
=> Cloning into '/home/.nvm'...
remote: Enumerating objects: 382, done.
remote: Counting objects: 100% (382/382), done.
remote: Compressing objects: 100% (325/325), done.
remote: Total 382 (delta 43), reused 179 (delta 29), pack-reused 0 (from 0)
Receiving objects: 100% (382/382), 385.06 KiB | 2.26 MiB/s, done.
Resolving deltas: 100% (43/43), done.
* (HEAD detached at FETCH_HEAD)
  master
=> Compressing and cleaning up git repository

=> Appending nvm source string to /home/.bashrc
=> Appending bash_completion source string to /home/.bashrc
internal/modules/cjs/loader.js:638
    throw err;
    ^

Error: Cannot find module 'node:path'
    at Function.Module._resolveFilename (internal/modules/cjs/loader.js:636:15)
    at Function.Module._load (internal/modules/cjs/loader.js:562:25)
    at Module.require (internal/modules/cjs/loader.js:692:17)
    at require (internal/modules/cjs/helpers.js:25:18)
    at Object.<anonymous> (/usr/local/lib/node_modules/npm/lib/cli.js:10:18)
    at Module._compile (internal/modules/cjs/loader.js:778:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:789:10)
    at Module.load (internal/modules/cjs/loader.js:653:32)
    at tryModuleLoad (internal/modules/cjs/loader.js:593:12)
    at Function.Module._load (internal/modules/cjs/loader.js:585:3)
=> Close and reopen your terminal to start using nvm or run the following to use it now:

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
~/myProject$ source ~/.bashrc
~/myProject$ nvm use node
Now using node v24.0.2 (npm v11.3.0)
~/myProject$ npm -v
11.3.0
```

これでnvmのインストールが完了し，npmのバージョンも上がった．改めて「npm install webpack@latest」を実行．

脆弱性に関するエラーが生じていた：

```Bash
47 packages are looking for funding
  run `npm fund` for details

16 vulnerabilities (4 moderate, 12 high)
```

これは「npm audit fix --force」を行って解消した．(sudoは不要のはず)

### エラーへの対処：package.jsonの整備

次に「npm run start」を行うとエラーが変化した：

```Bash
~/myProject$ npm run start

> rust-webpack-template@1.0.0 start
> rimraf dist pkg && webpack-dev-server --open -d

[webpack-cli] Error: Option '-d, --devtool <value>' argument missing
[webpack-cli] Run 'webpack --help' to see available commands and options
```

これはまず，run startで何が起きているかを見ないといけない：

**package.json**

```json
{
  "author": "You <you@example.com>",
  "name": "rust-webpack-template",
  "version": "1.0.0",
  "scripts": {
    "build": "rimraf dist pkg && webpack",
    "start": "rimraf dist pkg && webpack-dev-server --open -d source-map",
    "test": "cargo test && wasm-pack test --headless"
  },
  "devDependencies": {
    "@wasm-tool/wasm-pack-plugin": "^1.1.0",
    "copy-webpack-plugin": "^5.0.3",
    "rimraf": "^3.0.0",
    "webpack": "^5.99.8",
    "webpack-cli": "^6.0.1",
    "webpack-dev-server": "^5.2.1"
  },
  "description": "```sh npm install ```",
  "main": "webpack.config.js",
  "directories": {
    "test": "tests"
  },
  "license": "ISC"
}
```

このstartの部分で，npm run startのときに実行するコマンドの詳細が書かれている．
大本では，「"start": "rimraf dist pkg && webpack-dev-server --open -d"」となっており，-dの引数の後にsource-mapが無いためにエラーになっているとエラーメッセージから推測できる．なので追加した．
また，webpack.config.jsにもsource-mapの記述を追加している．

### エラーへの対処：webpack.config.jsの整備

次に生じたエラーはこれである：

```Bash
~/myProject$ npm run start

> rust-webpack-template@1.0.0 start
> rimraf dist pkg && webpack-dev-server --open -d source-map

[webpack-cli] Invalid options object. Dev Server has been initialized using an options object that does not match the API schema.
 - options has an unknown property 'contentBase'. These properties are valid:
   object { allowedHosts?, bonjour?, client?, compress?, devMiddleware?, headers?, historyApiFallback?, host?, hot?, ipc?, liveReload?, onListening?, open?, port?, proxy?, server?, app?, setupExitSignals?, setupMiddlewares?, static?, watchFiles?, webSocketServer? }

```

このエラーの原因は，webpack-dev-serverのlatest-versionがv4以降の場合，contentBaseオプションが削除されているからである．
sontentBaseを使っているのが，参考書籍を基に進めていった時の初期状態であると推測して，contentBaseの代替表現を使用した．
参考：[contentBase](https://qiita.com/chocomint_t/items/4bc57945bce081922582)

**webpack.config.js**

```js
const path = require("path");
const CopyPlugin = require("copy-webpack-plugin");
const WasmPackPlugin = require("@wasm-tool/wasm-pack-plugin");
const { experiments } = require("webpack");

const dist = path.resolve(__dirname, "dist");

module.exports = {
  mode: "production",
  devtool: "source-map", // 追加部分
  entry: {
    index: "./js/index.js"
  },
  output: {
    path: dist,
    filename: "[name].js"
  },
  devServer: {
    static: {
      directory: path.join(__dirname, "dist"),  // 追加：contentBaseの代替表現
    }
  },
  plugins: [
    new CopyPlugin([
      path.resolve(__dirname, "static")
    ]),

    new WasmPackPlugin({
      crateDirectory: __dirname,
    }),
  ]
};

```

次にこんなエラーが生じた：

```Bash
ERROR in ./pkg/index_bg.wasm 1:0
Module parse failed: Unexpected character '' (1:0)
The module seem to be a WebAssembly module, but module is not flagged as WebAssembly module for webpack.
BREAKING CHANGE: Since webpack 5 WebAssembly is not enabled by default and flagged as experimental feature.
You need to enable one of the WebAssembly experiments via 'experiments.asyncWebAssembly: true' (based on async modules) or 'experiments.syncWebAssembly: true' (like webpack 4, deprecated).
For files that transpile to WebAssembly, make sure to set the module type in the 'module.rules' section of the config (e. g. 'type: "webassembly/async"').
(Source code omitted for this binary file)
 @ ./pkg/index.js 1:0-40 4:15-19 5:0-21
 @ ./js/index.js
```

上記のエラーメッセージを基にwebpack.config.jsに記述を追加した(webpack 5 以降、WebAssembly はデフォルトでは有効になっておらず、実験的な機能扱いのフラグが立っている)：

```js

module.exports = {
  experiments: {
    asyncWebAssembly: true // 実際はここだけ -> 実験的WebAssemblyの
  }, 
  mode: "production",
…

```

以上を設定すると，コンパイルが通った．

### rust + wasm の初期画面(canvas)

以上の設定を行って，npmをrunさせるとようやくコンパイルが通りました．．

```Bash
~/myProject$ npm run start

> rust-webpack-template@1.0.0 start
> rimraf dist pkg && webpack-dev-server --open -d source-map

🧐  Checking for wasm-pack...

✅  wasm-pack is installed at /home/.cargo/bin/wasm-pack. 

ℹ️  Compiling your crate in release mode...

<i> [webpack-dev-server] Project is running at:
<i> [webpack-dev-server] Loopback: http://localhost:8080/, http://[::1]:8080/
<i> [webpack-dev-server] On Your Network (IPv4): http://XX.XX.XX.XX:8080/
<i> [webpack-dev-server] Content not from webpack is served from '/home/myProject/dist' directory
[INFO]: 🎯  Checking for the Wasm target...
[INFO]: 🌀  Compiling to Wasm...
warning: Found `debug_assertions` in `target.'cfg(...)'.dependencies`. This value is not supported for selecting dependencies and will not work as expected. To learn more visit https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#platform-specific-dependencies
    Finished `release` profile [optimized] target(s) in 0.08s
<i> [webpack-dev-middleware] wait until bundle finished: /
[INFO]: ⬇️  Installing wasm-bindgen...
[INFO]: Optimizing wasm binaries with `wasm-opt`...
[INFO]: Optional fields missing from Cargo.toml: 'repository', 'license'. These are not necessary, but recommended
[INFO]: ✨   Done in 0.81s
[INFO]: 📦   Your wasm pkg is ready to publish at /home/myProject/pkg.
✅  Your crate has been correctly compiled

(node:202752) [DEP_WEBPACK_COMPILATION_ASSETS] DeprecationWarning: Compilation.assets will be frozen in future, all modifications are deprecated.
BREAKING CHANGE: No more changes should happen to Compilation.assets after sealing the Compilation.
	Do changes to assets earlier, e. g. in Compilation.hooks.processAssets.
	Make sure to select an appropriate stage from Compilation.PROCESS_ASSETS_STAGE_*.
(Use `node --trace-deprecation ...` to show where the warning was created)
assets by path *.js 61 KiB
  asset index.js 58.4 KiB [emitted] [minimized] (name: index) 2 related assets
  asset 605.js 2.58 KiB [emitted] [minimized] 1 related asset
asset dc4e3d4c10ad1359dbaa.module.wasm 20.7 KiB [emitted] [immutable]
asset index.html 179 bytes [emitted]
runtime modules 33.5 KiB 16 modules
orphan modules 33.4 KiB [orphan] 4 modules
cacheable modules 113 KiB (javascript) 20.7 KiB (webassembly)
  modules by path ./node_modules/ 109 KiB
    modules by path ./node_modules/webpack-dev-server/client/ 84.8 KiB 4 modules
    modules by path ./node_modules/webpack/hot/*.js 5.17 KiB
      ./node_modules/webpack/hot/dev-server.js 1.94 KiB [built] [code generated]
      + 3 modules
    ./node_modules/events/events.js 14.5 KiB [built] [code generated]
    ./node_modules/ansi-html-community/index.js 4.16 KiB [built] [code generated]
  modules by path ./pkg/ 4.25 KiB (javascript) 20.7 KiB (webassembly)
    ./pkg/index.js 167 bytes [built] [code generated]
    ./pkg/index_bg.wasm 120 bytes (javascript) 20.7 KiB (webassembly) [built] [code generated]
    ./pkg/index_bg.js 3.97 KiB [built] [code generated]
  ./js/index.js 48 bytes [built] [code generated]
webpack 5.99.8 compiled successfully in 3481 ms
<i> [webpack-dev-middleware] wait until bundle finished: /index.js
assets by status 20.7 KiB [cached] 1 asset
asset index.js 58.4 KiB [emitted] [minimized] (name: index) 2 related assets
asset 605.js 2.58 KiB [emitted] [minimized] 1 related asset
runtime modules 33.5 KiB 16 modules
orphan modules 33.4 KiB [orphan] 4 modules
cacheable modules 113 KiB (javascript) 20.7 KiB (webassembly)
  modules by path ./node_modules/ 109 KiB
    modules by path ./node_modules/webpack-dev-server/client/ 84.8 KiB 4 modules
    modules by path ./node_modules/webpack/hot/*.js 5.17 KiB
      ./node_modules/webpack/hot/dev-server.js 1.94 KiB [built] [code generated]
      + 3 modules
    ./node_modules/events/events.js 14.5 KiB [built] [code generated]
    ./node_modules/ansi-html-community/index.js 4.16 KiB [built] [code generated]
  modules by path ./pkg/ 4.25 KiB (javascript) 20.7 KiB (webassembly)
    ./pkg/index.js 167 bytes [built] [code generated]
    ./pkg/index_bg.wasm 120 bytes (javascript) 20.7 KiB (webassembly) [built] [code generated]
    ./pkg/index_bg.js 3.97 KiB [built] [code generated]
  ./js/index.js 48 bytes [built] [code generated]
webpack 5.99.8 compiled successfully in 1900 ms

```

![compiled successfully]({{ site.baseurl }}/assets/image/rust/done_compile_20250705.png)

rustはコンパイルを通すのが大変(まあ今回はほとんどrust関係ないですが……)．

今回は以上です．
