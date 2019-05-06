### REPL Summary

It might look like much, but these are the important changes:

- augment the `src/visitors/process.js` and `src/visitors/process.js` to actually remove the dead branch when replacing `process.(browser|env)` (might become core with my suggestion in #2650)
- with the previous change, put `require`s for Watcher, (HMR)Server, WorkerFarm, ... in a conditional to not bundle them (aren't used anyway with the options used by the REPL)
- alias `fs` to a wrapper around `memory-fs` (makes most packages out of the box, though many try to be very clever if the browser is targeted and therefore break: mainly LESS, browserslist)
- don't bundle "native" Assets (Rust, Kotlin, ...) - compilers don't run in the browser
- some workarounds/monkeypatching for LESS, SASS, node-libs-browser
- in the JS packagers use `fs.readFileSync` directly because Parcel can't follow the code flow through `getExisting` to bundle the prelude snippets
- a browser implementation for Logger (only call console.\*)
- parse sourcemapppingURL only at end of file: #2427
- polyfill Buffer
- (merged scope hoisting destructuring: #2742 - but scope hoisting doesn't really work)
- REPL itself powered by Preact X

Building with scope hoisting or source maps results in an Out of Memory crash

### Scope Hoisting doesn't work / requiring node modules like `path` doesn't work

```
parcel/packages/core/parcel-bundler/src/builtins/index.js:
    node-libs-browser:
        require.resolve()
```

`require.resolve` should return module id that can be required?

### Peculiarities

Connecting with Chrome Devtools to Chrome on Android silently prevents the execution of Web Workers

### Other TODOs

Are these really optimized away: ?

```js
if(process.env.NODE_ENV === "development"){
  require("preact/debug");
}
if(module.hot) ...
if(process.browser) ...
```

Maybe:

```
parcel-bundler: (worker)
{
  "browser": {
    "./node.js": "./browser.js",
    "path": false
  }
}
```

### REPL Issues

- builtins/bundle loaders: fs.readFileSync(require.resolve)
- Safari SW: `clients` doesn't exist? disable html preview
- .babelrc isn't used:

  - `localRequire.resolve`: https://github.com/parcel-bundler/parcel/blob/599381399aaa00f02d6bd93c55ad22ca7d3fa0e6/packages/core/parcel-bundler/src/transforms/babel/babelrc.js#L272-L280

- need a better way to distinguish iframe/app requests in SW:
  - links to other html bundles in preview aren't recognized correctly

#### Features

- add test (that every preset runs without an error)
- better caching strategy: leverage hashing but still clear sw cache

#### Using Parcel's Cache

- mtime
  - "polyfill"
  - use `memfs` for mtime (but: `require(...).emitWarning is not a function`)
- (when enabled atm, Markdown isn't updating)
- with cache, watcher-like functionality (on state change with debouncin)

##### Future/Longterm

- Add a "expand" pull tab to options box
- Feedback that bundling was started/finished
- with Parcel 2: display graph
- Lazy load large `Asset` types
- {install pkg from NPM (via custom autoinstall)}
- use Parcel's devserver in SW
- "Production" ? NODE_ENV ? As cli flag?