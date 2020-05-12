# enhanced-resolve

Offers an async require.resolve function. It's highly configurable.

## Features

- plugin system
- provide a custom filesystem
- sync and async node.js filesystems included

## Getting Started

### Install

```sh
# npm
npm install enhanced-resolve
# or Yarn
yarn add enhanced-resolve
```

### Resolve

There is a Node.js API which allows to resolve requests according to the Node.js resolving rules.
Sync and async APIs are offered. A `create` method allows to create a custom resolve function.

```js
const resolve = require("enhanced-resolve");

resolve("/some/path/to/folder", "module/dir", (err, result) => {
	result; // === "/some/path/node_modules/module/dir/index.js"
});

resolve.sync("/some/path/to/folder", "../../dir");
// === "/some/path/dir/index.js"

const myResolve = resolve.create({
	// or resolve.create.sync
	extensions: [".ts", ".js"]
	// see more options below
});

myResolve("/some/path/to/folder", "ts-module", (err, result) => {
	result; // === "/some/node_modules/ts-module/index.ts"
});
```

### Creating a Resolver

The easiest way to create a resolver is to use the `createResolver` function on `ResolveFactory`, along with one of the supplied File System implementations.

```js
const fs = require("fs");
const { CachedInputFileSystem, ResolverFactory } = require("enhanced-resolve");

// create a resolver
const myResolver = ResolverFactory.createResolver({
	// Typical usage will consume the `fs` + `CachedInputFileSystem`, which wraps Node.js `fs` to add caching.
	fileSystem: new CachedInputFileSystem(fs, 4000),
	extensions: [".js", ".json"]
	/* any other resolver options here. Options/defaults can be seen below */
});

// resolve a file with the new resolver
const context = {};
const resolveContext = {};
const lookupStartPath = "/Users/webpack/some/root/dir";
const request = "./path/to-look-up.js";
myResolver.resolve({}, lookupStartPath, request, resolveContext, (
	err /*Error*/,
	filepath /*string*/
) => {
	// Do something with the path
});
```

#### Resolver Options

| Field            | Default                     | Description                                                                                                                                   |
| ---------------- | --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| alias            | []                          | A list of module alias configurations or an object which maps key to value                                                                    |
| aliasFields      | []                          | A list of alias fields in description files                                                                                                   |
| cacheWithContext | true                        | If unsafe cache is enabled, includes `request.context` in the cache key                                                                       |
| conditionNames   | []                          | A list of exports field condition names                                                                                                       |
| descriptionFiles | ["package.json"]            | A list of description files to read from                                                                                                      |
| enforceExtension | false                       | Enforce that a extension from extensions must be used                                                                                         |
| extensions       | [".js", ".json", ".node"]   | A list of extensions which should be tried for files                                                                                          |
| exportsFields    | ["exports"]                 | A list of exports fields in description files                                                                                                 |
| mainFields       | ["main"]                    | A list of main fields in description files                                                                                                    |
| mainFiles        | ["index"]                   | A list of main files in directories                                                                                                           |
| modules          | ["node_modules"]            | A list of directories to resolve modules from, can be absolute path or folder name                                                            |
| unsafeCache      | false                       | Use this cache object to unsafely cache the successful requests                                                                               |
| plugins          | []                          | A list of additional resolve plugins which should be applied                                                                                  |
| symlinks         | true                        | Whether to resolve symlinks to their symlinked location                                                                                       |
| cachePredicate   | function() { return true }; | A function which decides whether a request should be cached or not. An object is passed to the function with `path` and `request` properties. |
| resolveToContext | false                       | Resolve to a context instead of a file                                                                                                        |
| fileSystem       |                             | The file system which should be used                                                                                                          |
| resolver         | undefined                   | A prepared Resolver to which the plugins are attached                                                                                         |

## Plugins

Similar to `webpack`, the core of `enhanced-resolve` functionality is implemented as individual plugins that are executed using [`tapable`](https://github.com/webpack/tapable).
These plugins can extend the functionality of the library, adding other ways for files/contexts to be resolved.

A plugin should be a `class` (or its ES5 equivalent) with an `apply` method. The `apply` method will receive a `resolver` instance, that can be used to hook in to the event system.

### Plugin Boilerplate

```js
class MyResolverPlugin {
	constructor(source, target) {
		this.source = source;
		this.target = target;
	}

	apply(resolver) {
		const target = resolver.ensureHook(this.target);
		resolver
			.getHook(this.source)
			.tapAsync("MyResolverPlugin", (request, resolveContext, callback) => {
				// Any logic you need to create a new `request` can go here
				resolver.doResolve(target, request, null, resolveContext, callback);
			});
	}
}
```

Plugins are executed in a pipeline, and register which event they should be executed before/after. In the example above, `source` is the name of the event that starts the pipeline, and `target` is what event this plugin should fire, which is what continues the execution of the pipeline. For an example of how these different plugin events create a chain, see `lib/ResolverFactory.js`, in the `//// pipeline ////` section.

## Tests

```javascript
npm test
```

[![Build Status](https://secure.travis-ci.org/webpack/enhanced-resolve.png?branch=master)](http://travis-ci.org/webpack/enhanced-resolve)

## Passing options from webpack

If you are using `webpack`, and you want to pass custom options to `enhanced-resolve`, the options are passed from the `resolve` key of your webpack configuration e.g.:

```
resolve: {
  extensions: ['.js', '.jsx'],
  modules: [path.resolve(__dirname, 'src'), 'node_modules'],
  plugins: [new DirectoryNamedWebpackPlugin()]
  ...
},
```

## License

Copyright (c) 2012-2019 JS Foundation and other contributors

MIT (http://www.opensource.org/licenses/mit-license.php)
