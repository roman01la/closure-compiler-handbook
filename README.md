# Introduction 👋

<img src="logo.png" alt="Closure Compiler handbook logo" width="322" height="114" />

This handbook is designed to help you understand how to use Closure Compiler and learn its features.

> The Closure Compiler is a tool for making JavaScript download and run faster. Instead of compiling from a source language to machine code, it compiles from JavaScript to better JavaScript. It parses your JavaScript, analyzes it, removes dead code and rewrites and minimizes what's left. It also checks syntax, variable references, and types, and warns about common JavaScript pitfalls.

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" src="https://i.creativecommons.org/l/by-sa/4.0/80x15.png" /></a>

[Closure Compiler](https://developers.google.com/closure/compiler/) can compile, optimize and bundle JavaScript code. You can use it for building your JavaScript apps for production. It is the most advanced JavaScript optimization tool. It generates smallest bundle and emits efficient JavaScript code by doing whole program analysis and optimization, removing closures, inlining function calls, reusing variable names and pre-computing constant expressions.

React + ReactDOM build comparison

Tool            | Output size | Gzipped |
----------------|-------------|---------|
Webpack 2       |139KB        |42KB     |
Closure Compiler|90KB         |32KB     |

Here's an example of compiler's compilation output:

*input*
```js
function hello(name) {
  const message = `Hello, ${name}!`;
  console.log(message);
}
hello('New user');
```

*output*
```js
console.log("Hello, New user!");
```

# Table of Contents 📖
- [Introduction](#introduction-)
- [Table of Contents](#table-of-contents-)
- [Getting started](#getting-started-)
  - [Installation](#installation)
  - [Using with Node](#using-with-node)
  - [Using with Webpack](#using-with-webpack)
  - [Using with Gulp](#using-with-gulp)
  - [Splittable bundler](#splittable-bundler)
- [Compilation levels](#compilation-levels-)
- [Advanced compilation](#advanced-compilation-)
  - [Referencing external code](#referencing-external-code)
  - [Referencing compiled code](#referencing-compiled-code)
  - [Referencing to object properties using both dot and bracket notations](#referencing-to-object-properties-using-both-dot-and-bracket-notations)
- [Compiler optimizations](#compiler-optimizations-)
  - [Dead code elimination and Tree-shaking](#dead-code-elimination-and-tree-shaking)
  - [Unreachable and redundant code elimination](#unreachable-and-redundant-code-elimination)
  - [Cross-module code motion](#cross-module-code-motion)
  - [Constant folding](#constant-folding)
  - [Function call inlining](#function-call-inlining)
  - [Property flattening (collapsing)](#property-flattening-collapsing)
  - [Variable and property renaming](#variable-and-property-renaming)
  - [Statement fusion (merging) & variable declarations grouping](#statement-fusion-merging--variable-declarations-grouping)
  - [Alternate syntax substitution](#alternate-syntax-substitution)
  - [RegExp optimization](#regexp-optimization)
  - [Known methods folding](#known-methods-folding)
  - [Property assignment collection](#property-assignment-collection)
  - [Anonymous functions naming](#anonymous-functions-naming)
- [Compiler flags](#compiler-flags-)
- [Supported languages](#supported-languages-)
- [JavaScript modules](#javascript-modules-)
- [Recipes](#recipes-)
  - [Exporting to global scope](#exporting-to-global-scope)
  - [Code splitting](#code-splitting)
- [Who is using it?](#who-is-using-it)

# Getting started 🏁

Closure Compiler is written in Java, but it also has JavaScript port.
The best way to use it is [Splittable](https://github.com/cramforce/splittable) bundler.

- [google/closure-compiler](https://github.com/google/closure-compiler)
- [google/closure-compiler-js](https://github.com/google/closure-compiler-js)
- [Splittable](https://github.com/cramforce/splittable)

## Installation

```bash
npm i google-closure-compiler-js
```

## Using with Node

```js
const compile = require('google-closure-compiler-js').compile;

const flags = {
  jsCode: [{src: 'const inc = (x) => x + 1;'}]
};

const out = compile(flags);

console.log(out.compiledCode); // 'var inc=function(a){return a+1};'
```

## Using with Webpack

```js
const ClosureCompiler = require('google-closure-compiler-js').webpack;
const path = require('path');

module.exports = {
  entry: [
    path.join(__dirname, 'entry.js')
  ],
  output: {
    path: path.join(__dirname, 'build'),
    filename: 'bundle.js'
  },
  plugins: [
    new ClosureCompiler({
      options: {
        languageIn: 'ECMASCRIPT6',
        languageOut: 'ECMASCRIPT3',
        compilationLevel: 'ADVANCED'
      }
    })
  ]
};
```

## Using with Gulp

```js
const compiler = require('google-closure-compiler-js').gulp();

gulp.task('build', function() {
  return gulp.src('enrty.js', {base: './'})
      .pipe(compiler({
          compilationLevel: 'ADVANCED',
          jsOutputFile: 'bundle.js',
          createSourceMap: true
        }))
      .pipe(gulp.dest('./build'));
});
```

## Splittable bundler

[Splittable](https://github.com/cramforce/splittable) is a module bundler for JavaScript based on Closure Compiler. Basically it's a wrapper with zero configuration which supports ES6 and code splitting out of the box.

```js
const splittable = require('splittable');

splittable({
  // Create bundles from 2 entry modules `./src/a` and `./src/b`.
  modules: ['./src/a', './src/b'],
  writeTo: 'dist/',
})
.then((info) => {
  console.info('Compilation successful');
  if (info.warnings) {
    console.warn(info.warnings);
  }
})
.catch((error) => console.error('Compilation failed', error));
```

# Compilation levels 🎚

Compilation level is a compiler setting which denotes optimizations level to be applied to JavaScript code.

`WHITESPACE_ONLY`

Removes comments, line breaks, unnecessary spaces and other whitespace.

`SIMPLE` *(default)*

Includes `WHITESPACE_ONLY` optimizations and renames variable names to shorter names to reduce the size of output code. It renames only variables local to functions, which means that it won't break references to third party code from global scope.

`ADVANCED`

Includes `SIMPLE` optimizations and performs aggressive transformations such as closures elimination, inlining function calls, reusing variable names, pre-computing constant expressions, tree-shaking, cross-module code motion and dead code elimination.

This kind of aggressive compression makes some assumptions about your code. If it doesn't conform to those assumptions, Closure Compiler will produce output that does not run.

# Advanced compilation 👷

With `ADVANCED` compilation level Closure Compiler renames global variables, function names and properties and removes unused code. This can lead to output that will not run if your code doesn't follow certain rules.

## Referencing external code

If you want to use globally defined variables and functions in your code safely, you must tell the compiler about those references.

*input*
```js
// `moment` is declared in global scope
window.moment().subtract(10, 'days').calendar();
```

*output*
```js
// `moment` was renamed to `a`
// this will not run
window.a().b(10, 'days').calendar();
```

## Referencing compiled code

If you are about to build a library that exports to global scope, you must tell the compiler about variables that should be exported safely.

*input*
```js
// `MY_APP` is declared in global scope
window.MY_APP = {};
```

*output*
```js
// `MY_APP` was renamed to `a`
// it's no longer possible to reference `MY_APP`
window.a = {};
```

## Referencing to object properties using both dot and bracket notations

Closure Compiler never rewrites strings. You should use only one way of declaring and accessing a property:
- declare with a symbol, access with dot notation
- declare with a string, access with a string

*input*
```js
// `msg` property is declared and accessed in different ways
obj = { msg: 'Hey!' };
console.log(obj['msg']);
```

*output*
```js
// `msg` symbol is renamed to `a`, but `'msg'` text is not
obj = { a: 'Hey!' };
console.log(obj.msg);
```

# Compiler optimizations 🎛

Closure Compiler performs a number of optimizations to produce a small output size. Some of them are being applied in intermediate compilation pass to produce AST which is suited best for further code optimization.

*NOTE: Below is a list of the most interesting optimizations that compiler does. But there are more of them, you can find them all in comments in the source code of the compiler.*

## Dead code elimination and Tree-shaking

“Dead code” is a code that is never going to be called in your program. Closure Compiler can efficiently determine and remove such code because it is a whole-program optimization compiler, which means that it performs analysis of the whole program (in comparison to less effective analysis on module level). It constructs a graph of all variables and dependencies which are declared in your code, does graph traversal to find what should be included and dismisses the rest, which is a dead code.

*input*
```js
/* log.js module */
export const logWithMsg = (msg, arg) => console.log(msg, arg);
export const log = (arg) => console.log(arg);

/* math.js module */
export const min = (a, b) => Math.min(a, b);
export const max = (a, b) => Math.max(a, b);
export const exp = (x) => x * x;
export const sum = (xs) => xs.reduce((a, b) => a + b, 0);

/* entry.js entry point */
import * as math from './math';
import * as logger from './log';

const nums = [0, 1, 2, 3, 4, 5];
const msg = 'Result:';

if (false) {
  logger.log('nothing');
} else {
  logger.logWithMsg(msg, math.sum(nums));
}
```

*output*
```js
// Even though the entire modules namespace was imported,
// tree-shaking didn't include dependency code that is not used here.
// Also a dead code within `if (false) { ... }` was removed.
var c = function(a) {
  return a.reduce(function(a, b) {
    return a + b;
  }, 0);
}([0,1,2,3,4,5]);

console.log("Result:",c);
```

## Unreachable and redundant code elimination

This optimization is a little different from dead code elimination. It removes “live code” that doesn't have an impact on a program, e.g. statements without side effects (`true;`), useless `break`, `continue` and `return`.

## Cross-module code motion

Closure Compiler moves variables, functions and methods between modules. It moves the code along modules tree down to those modules where this code is needed. This can reduce parsing time before actual execution. This technique is especially useful for advanced code splitting which works on variables level, rather than modules level.

*input*
```js
/* math.js module */
export const sum = (xs) => xs.reduce((a, b) => a + b, 0);
export const mult = (xs) => xs.reduce((a, b) => a * b, 1);

/* entry-1.js module */
import * as math from './math';

console.log(math.sum(nums));

/* entry-2.js module */
import * as math from './math';

console.log(math.sum([4, 5, 6]) + math.mult([3, 5, 6]));
```

*output*
```js
/* common.js shared code */
function c(a) {
  return a.reduce(function(a, b) {
    return a + b;
  }, 0);
};

/* entry-1.bundle.js */
console.log(c(nums)); // using shared code

/* entry-2.bundle.js */
console.log(
  // using shared code
  c([4,5,6]) +
  // moved from `math.js` module
  function(a) {
    return a.reduce(function(a, b) {
      return a * b;
    }, 1);
  }([3,5,6])
);
```

## Constant folding

> Constant folding is the process of recognizing and evaluating constant expressions at compile time rather than computing them at runtime. Terms in constant expressions are typically simple literals, such as the integer literal 2 , but they may also be variables whose values are known at compile time.

*input*
```js
const years = 14;
const monthsInYear = 12;
const daysInMonth = 30;

console.log(years * monthsInYear * daysInMonth);
```

*output*
```js
console.log(5040);
// `years * monthsInYear * daysInMonth` computed at compile time
// because they are known as constants
```

## Function call inlining

To inline a function means to replace a function call with its body. The function definition can be dismissed and it also eliminates an additional function call. If the compiler couldn't perform this type of inlining, it can inline function declaration with a call it in place.

*input*
```js
const person = {
  fname: 'John',
  lname: 'Doe',
};

function getFullName({ fname, lname }) {
  return fname + ' ' + lname;
}

console.log(getFullName(person));
```

*output*
```js
var a = { a: "John", b: "Doe" };
console.log(a.a + " " + a.b);
```

## Property flattening (collapsing)

Collapsing object properties into separate variables enables such optimizations as variable renaming, inlining and better dead code removal.

*input*
```js
const person = {
  fname: 'John',
  lname: 'Doe'
};

console.log(person.fname);
```

*output*
```js
var person$fname = "John",
    person$lname = "Doe"; // <- is not used, can be removed

console.log(person$fname);
```

## Variable and property renaming

Small output size is partially achieved by renaming all variables and object properties. Because the compiler renames object properties you have to make sure that you are referencing and declaring properties either with symbol (`obj.prop`) or string (`obj['prop']`).

*input*
```js
const user = window.session.user;
console.log(user.apiToken, user.tokenExpireDate);
```

*output*
```js
var a = window.b.f;
console.log(a.a, a.c);
```

## Statement fusion (merging) & variable declarations grouping

Statement fusion tries to merge multiple statements in a single one. And variable declarations grouping groups multiple variable declarations into a single one.

*input*
```js
const fname = 'John';
const lname = 'Doe';

if (fname) {
  console.log(fname);
}
```

*output*
```js
var fname = "John", lname = "Doe";
fname && console.log(fname);
```

## Alternate syntax substitution

Simplifies conditional expressions, replaces `if`s with ternary operator, object and array constructs with literals and simplifies `return`s.

## RegExp optimization

Removes unnecessary flags and reorders them for better gzip.

## Known methods folding

This precomputes known methods such as `join`, `indexOf`, `substring`, `substr`, `parseInt` and `parseFloat` when they are called with constants.

*input*
```js
[0, 1, 2, 3, 4, 5].join('');
```

*output*
```js
"012345"
```

## Property assignment collection

Looks for assignments to properties of object/array immediately following its creation using the abbreviated syntax and merges assigned values into object/array creation construct.

*input*
```js
const coll = [];
coll[0] = 0;
coll[2] = 5;

const obj = { x: 1 };

obj.y = 2;
```

*output*
```js
var coll = [0, , 5], obj = { x:1, y:2 };
```

## Anonymous functions naming

Gives anonymous function names. This makes it way easier to debug because debuggers and stack traces use the function names.

*input*
```js
math.simple.add = function(a, b) {
  return a + b;
};
```

*output*
```js
math.simple.add = function $math$simple$add$(a, b) {
  return a + b;
};
```

# Compiler flags 🚩

There are much more compiler flags, see all of them in [google/closure-compiler-js](https://github.com/google/closure-compiler-js#flags) repo.

| Flag                             | Default | Usage |
|----------------------------------|---------|-------|
| applyInputSourceMaps | `true` | Compose input source maps into output source map |
| assumeFunctionWrapper | `false` | Enable additional optimizations based on the assumption that the output will be wrapped with a function wrapper. This flag is used to indicate that "global" declarations will not actually be global but instead isolated to the compilation unit. This enables additional optimizations. |
| compilationLevel | `SIMPLE` | Specifies the compilation level to use: `WHITESPACE_ONLY`, `SIMPLE`, `ADVANCED` |
| env | `BROWSER` | Determines the set of builtin externs to load: `BROWSER`, `CUSTOM` |
| languageIn | `ES6` | Sets what language spec that input sources conform to. |
| languageOut | `ES5` | Sets what language spec the output should conform to. |
| newTypeInf | `false` | Checks for type errors using the new type inference algorithm. |
| outputWrapper | `null` | Interpolate output into this string, replacing the token `%output%` |
| processCommonJsModules | `false` | Process CommonJS modules to a concatenable form, i.e., support `require` statements. |
| rewritePolyfills | `false` | Rewrite ES6 library calls to use polyfills provided by the compiler's runtime. |
| warningLevel | `DEFAULT` | Specifies the warning level to use: `QUIET`, `DEFAULT`, `VERBOSE` |
| jsCode | `[]` | Specifies the source code to compile. |
| externs | `[]` | Additional externs to use for this compile. |
| createSourceMap | `false` | Generates a source map mapping the generated source file back to its original sources. |

# Supported languages 🙊

| Language | Option name          | Input | Output |
|----------|----------------------|-------|--------|
| ES3      | `ECMASCRIPT3`        | ✅    |   ✅   |
| ES5      | `ECMASCRIPT5`        | ✅    |   ✅   |
| ES5      | `ECMASCRIPT5_STRICT` | ✅    |   ✅   |
| ES2015   | `ECMASCRIPT6`        | ✅    |        |
| ES2015   | `ECMASCRIPT6_STRICT` | ✅    |        |
| ES2015   | `ECMASCRIPT6_TYPED`  | ✅    |   ✅   |

# JavaScript modules 🌯

# Recipes 🍜

## Exporting to global scope

If you are building a library and not using JavaScript modules, you can export functions and variables safely using bracket notation and quoted property names, since Closure Compiler doesn't rename strings.

*input*
```js
function logger(x) {
  console.log('LOG:', x);
}

window['logger'] = logger;
```

*output*
```js
// `logger` function is exported properly into global scope
window.logger = function(a) {
  console.log('LOG:', a);
};
```

## Code splitting

Code splitting is a technique used to reduce initial script loading time by splitting a program into a number of bundles that are loaded later. This is especially important for modern web apps. With code splitting you could have a separate bundle for every route in a program, so initially browser will load only minimal amount of code to run and show requested view to a user and other code can be lazy-loaded later.

# Who is using it?

- [ClojureScript](https://clojurescript.org/)
- [Scala.js](https://www.scala-js.org/)
- [Dart](https://www.dartlang.org/)
- [GWT](http://www.gwtproject.org/)
- [Tsickle — TypeScript to Closure Annotator](https://github.com/angular/tsickle)
