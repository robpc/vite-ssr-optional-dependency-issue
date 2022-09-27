# vite-ssr-optional-dependency-issue

## Problem

See the issue by running

```bash
npm install
npm run dev
```

Loading the page results in the error

```bash
Error: ENOENT: no such file or directory, open '__vite-optional-peer-dep:canvas:jsdom'
    at Object.openSync (node:fs:585:3)
    at Object.readFileSync (node:fs:453:35)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1122:18)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
    at Module.require (node:internal/modules/cjs/loader:1005:19)
    at require (node:internal/modules/cjs/helpers:102:18)
    at Object.<anonymous> (./node_modules/jsdom/lib/jsdom/utils.js:158:18)
    at Module._compile (node:internal/modules/cjs/loader:1105:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1159:10)
```

The issue is not present when downgrading to `vite@3.0.4`.

## Investigation

Of note in the stacktrace is this line `./node_modules/jsdom/lib/jsdom/utils.js:158`.

```js
exports.Canvas = null;
let canvasInstalled = false;
try {
  require.resolve('canvas');
  canvasInstalled = true;
} catch (e) {
  // canvas is not installed
}
if (canvasInstalled) {
  ////// 👇 This is the line that causes the error //////
  const Canvas = require('canvas');
  if (typeof Canvas.createCanvas === 'function') {
    // In browserify, the require will succeed but return an empty object
    exports.Canvas = Canvas;
  }
}
```

Clearly the code expects `require.resolve("canvas");` to throw an exception if the canvas package is not
present, and prevent the offending line from running.

## Test 1

If you modify the jsdom code above to print what `require.resolve("canvas")` returns, like so

```js
const r = require.resolve('canvas');
console.log('I RESOLVED CANVAS', JSON.stringify(r, null, 2));
```

After restarting the dev server, you see

```md
I RESOLVED CANVAS "__vite-optional-peer-dep:canvas:jsdom"
```
