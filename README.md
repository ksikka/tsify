# tsify

[Browserify](http://browserify.org/) plugin for compiling [TypeScript](http://www.typescriptlang.org/)

[![NPM version](https://img.shields.io/npm/v/tsify.svg)](https://www.npmjs.com/package/tsify)
[![Downloads](http://img.shields.io/npm/dm/tsify.svg)](https://npmjs.org/package/tsify)
[![Build status](https://img.shields.io/travis/TypeStrong/tsify.svg)](http://travis-ci.org/TypeStrong/tsify)
[![Dependency status](https://img.shields.io/david/TypeStrong/tsify.svg)](https://david-dm.org/TypeStrong/tsify)
[![devDependency Status](https://img.shields.io/david/dev/TypeStrong/tsify.svg)](https://david-dm.org/TypeStrong/tsify#info=devDependencies)
[![peerDependency Status](https://img.shields.io/david/peer/TypeStrong/tsify.svg)](https://david-dm.org/TypeStrong/tsify#info=peerDependencies)

# Example Usage

### Browserify API:

``` js
var browserify = require('browserify');
var tsify = require('tsify');

browserify()
    .add('main.ts')
    .plugin(tsify, { noImplicitAny: true })
    .bundle()
    .on('error', function (error) { console.error(error.toString()); })
    .pipe(process.stdout);
```

### Command line:

``` sh
$ browserify main.ts -p [ tsify --noImplicitAny ] > bundle.js
```

Note that when using the Browserify CLI, compilation will always halt on the first error encountered, unlike the regular TypeScript CLI.  This behavior can be overridden in the API, as shown in the API example.

Also note that the square brackets `[ ]` in the example above are *required* if you want to pass parameters to tsify; they don't denote an optional part of the command.

# Installation

Just plain ol' [npm](https://npmjs.org/) installation:

### 1. Install browserify
```sh
npm install browserify
```

### 2. Install tsify
``` sh
npm install tsify
```

For use on the command line, use the flag `npm install -g`.

# Options

* **tsify** will generate sourcemaps if the `--debug` option is set on Browserify.
* **tsify** supports almost all options from the TypeScript compiler.  Notable exceptions:
	* `-d, --declaration` - See [tsify#15](https://github.com/TypeStrong/tsify/issues/15)
	* `--out, --outDir` - Use Browserify's file output options instead.  These options are overridden because **tsify** writes to an internal memory store before bundling, instead of to the filesystem.
* **tsify** supports the following extra options:
	* `--typescript` - This allows you to pass in a different TypeScript compiler, such as [NTypeScript](https://github.com/TypeStrong/ntypescript).  Note that when using the API, you can pass either the name of the alternative compiler or a reference to it:
		* `{ typescript: 'ntypescript' }`
		* `{ typescript: require('typescript') }`, useful for when you want to use a different version of the official TypeScript compiler than the one packaged with tsify.

# Does this work with...

### tsconfig.json?

tsify will automatically read options from `tsconfig.json`.  However, some options from this file will be ignored:

* `compilerOptions.declaration` - See [tsify#15](https://github.com/TypeStrong/tsify/issues/15)
* `compilerOptions.out` and `compilerOptions.outDir` - Use Browserify's file output options instead.  These options are overridden because **tsify** writes its intermediate JavaScript output to an internal memory store instead of to the filesystem.
* `files` - Use Browserify's file input options instead.  This is necessary because Browserify needs to know which file(s) are the entry points to your program.

### Watchify?

Yes!  **tsify** can do incremental compilation using [watchify](//github.com/substack/watchify), resulting in much faster incremental build times.  Just follow the Watchify documentation, and add **tsify** as a plugin as indicated in the documentation above.

### Gulp?

No problem.  See the Gulp recipes on using [browserify](https://github.com/gulpjs/gulp/blob/master/docs/recipes/browserify-uglify-sourcemap.md) and [watchify](https://github.com/gulpjs/gulp/blob/master/docs/recipes/fast-browserify-builds-with-watchify.md), and add **tsify** as a plugin as indicated in the documentation above.

### Grunt?

Use [grunt-browserify](https://github.com/jmreidy/grunt-browserify) and you should be good!  Just add **tsify** as a plugin in your Grunt configuration.

### IE 11?

The inlined sourcemaps that Browserify generates [may not be readable by IE 11](//github.com/TypeStrong/tsify/issues/19) for debugging purposes.  This is easy to fix by adding [exorcist](//github.com/thlorenz/exorcist) to your build workflow after Browserify.

### ES2015? *(formerly known as ES6)*

TypeScript's ES2015 output mode should work without too much additional setup.  Browserify does not support ES2015 modules, so if you want to use ES2015 you still need some transpilation step.  Make sure to add [babelify](//github.com/babel/babelify) to your list of transforms.  Note that if you are using the API, you need to set up **tsify** before babelify:

``` js
browserify()
    .plugin(tsify, { target: 'es6' })
    .transform(babelify, { extensions: [ '.tsx', '.ts' ] })
```

# FAQ / Common issues

### SyntaxError: 'import' and 'export' may appear only with 'sourceType: module'

This error occurs when a TypeScript file is not compiled to JavaScript before being run through the Browserify bundler.  There are a couple known reasons you might run into this.

* If you are trying to output in ES6 mode, then you have to use an additional transpilation step such as [babelify](//github.com/babel/babelify) because Browserify does not support bundling ES6 modules.
* Make sure that if you're using the API, your setup `.plugin('tsify')` is done *before* any transforms such as `.transform('babelify')`.  **tsify** needs to run first!
* There is a known issue in Browserify regarding including files with `expose` set to the name of the included file.  More details and a workaround are available in [#60](//github.com/TypeStrong/tsify/issues/60).

# Why a plugin?

There are several TypeScript compilation transforms available on npm, all with various issues.  The TypeScript compiler automatically performs dependency resolution on module imports, much like Browserify itself.  Browserify transforms are not flexible enough to deal with multiple file outputs given a single file input, which means that any working TypeScript compilation transform either skips the resolution step (which is necessary for complete type checking) or performs multiple compilations of source files further down the dependency graph.

**tsify** avoids this problem by using the power of plugins to perform a single compilation of the TypeScript source up-front, using Browserify to glue together the resulting files.

# License

MIT

# Changelog

* 0.15.2 - Added support for the `files` property of `tsconfig.json`.
* 0.15.1 - Added support for `--project` flag to use a custom location for `tsconfig.json`.
* 0.15.0 - Removed `debuglog` dependency.
* 0.14.8 - Reverted removal of `debuglog` dependency for compatibility with old versions of Node 0.12.
* 0.14.7 - Only generate sourcemap information in the compiler when `--debug` is set, for potential speed improvements when not using sourcemaps.
* 0.14.6 - Fixed output when `--jsx=preserve` is set.
* 0.14.5 - Removed `lodash` and `debuglog` dependencies.
* 0.14.4 - Fixed sourcemap paths when using Browserify's `basedir` option.
* 0.14.3 - Fixed `allowJs` option to enable transpiling ES6+ JS to ES5 or lower.
* 0.14.2 - Fixed `findConfigFile` for TypeScript 1.9 dev.
* 0.14.1 - Removed module mode override for ES6 mode (because CommonJS mode is now supported by TS 1.8).
* 0.14.0 - Updated to TypeScript 1.8 (thanks @joelday!)
* 0.13.2 - Fixed `findConfigFile` for use with the TypeScript 1.8 dev version.
* 0.13.1 - Fixed bug where `*.tsx` was not included in Browserify's list of extensions if the `jsx` option was set via `tsconfig.json`.
* 0.13.0 - Updated to TypeScript 1.7.
* 0.12.2 - Fixed resolution of entries outside of `process.cwd()` (thanks @pnlybubbles!)
* 0.12.1 - Updated `typescript` dependency to lock it down to version 1.6.x
* 0.12.0 - Updated to TypeScript 1.6.
* 0.11.16 - Updated `typescript` dependency to lock it down to version 1.5.x
* 0.11.15 - Added `*.tsx` to Browserify's list of extensions if `--jsx` is set (with priority *.ts > *.tsx > *.js).
* 0.11.14 - Override sourcemap settings with `--inlineSourceMap` and `--inlineSources` (because that's what Browserify expects).
* 0.11.13 - Fixed bug introduced in last change where non-entry point files were erroneously being excluded from the build.
* 0.11.12 - Fixed compilation when the current working directory is a symlink.
* 0.11.11 - Updated compiler host to support current TypeScript nightly.
* 0.11.10 - Updated resolution of `lib.d.ts` to support TypeScript 1.6 and to work with the `--typescript` option.
* 0.11.9 - Fixed dumb error.
* 0.11.8 - Handled JSX output from the TypeScript compiler to support `preserve`.
* 0.11.7 - Added `*.tsx` to the regex determining whether to run a file through the TypeScript compiler.
* 0.11.6 - Updated dependencies and devDependencies to latest.
* 0.11.5 - Fixed emit of `file` event to trigger watchify even when there are fatal compilation errors.
* 0.11.4 - Added `--typescript` option.
* 0.11.3 - Updated to TypeScript 1.5.
* 0.11.2 - Blacklisted `--out` and `--outDir` compiler options.
* 0.11.1 - Added `tsconfig.json` support.
* 0.11.0 - Altered behavior to pass through all compiler options to tsc by default.
* 0.10.2 - Fixed output of global error messages.  Fixed code generation in ES6 mode.
* 0.10.1 - Fixed display of nested error messages, e.g. many typing errors.
* 0.10.0 - Added `stopOnError` option and changed default behavior to continue building when there are typing errors.
* 0.9.0 - Updated to use TypeScript from npm (thanks @hexaglow!)
* 0.8.2 - Updated peerDependency for Browserify to allow any version >= 6.x.
* 0.8.1 - Updated peerDependency for Browserify 9.x.
* 0.8.0 - Updated to TypeScript 1.4.1.
* 0.7.1 - Updated peerDependency for Browserify 8.x.
* 0.7.0 - Updated error handling for compatibility with Watchify.
* 0.6.5 - Updated peerDependency for Browserify 7.x.
* 0.6.4 - Included richer file position information in syntax error messages.
* 0.6.3 - Updated to TypeScript 1.3.
* 0.6.2 - Included empty *.d.ts compiled files in bundle for Karma compatibility.
* 0.6.1 - Fixed compilation cache miss when given absolute filenames.
* 0.6.0 - Updated to TypeScript 1.1.
* 0.5.2 - Bugfix for 0.5.1 for files not included with expose.
* 0.5.1 - Handled *.d.ts files passed as entries. Fix for files included with expose.
* 0.5.0 - Updated to Browserify 6.x.
* 0.4.1 - Added npmignore to clean up published package.
* 0.4.0 - Dropped Browserify 4.x support. Fixed race condition causing pathological performance with some usage patterns, e.g. when used with [karma-browserify](https://github.com/Nikku/karma-browserify).
* 0.3.1 - Supported adding files with `bundler.add()`.
* 0.3.0 - Added Browserify 5.x support.
* 0.2.1 - Fixed paths for sources in sourcemaps.
* 0.2.0 - Made Browserify prioritize *.ts files over *.js files in dependency resolution.
* 0.1.4 - Handled case where the entry point is not a TypeScript file.
* 0.1.3 - Automatically added *.ts to Browserify's list of file extensions to resolve.
* 0.1.2 - Added sourcemap support.
* 0.1.1 - Fixed issue where intermediate *.js files were being written to disk when using `watchify`.
* 0.1.0 - Initial version.
