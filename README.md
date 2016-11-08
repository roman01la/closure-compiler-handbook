# Introduction

<img src="logo.png" alt="Closure Compiler handbook logo" width="322" height="114" />

This handbook is designed to help you understand how to use Closure Compiler and learn its features.

> The Closure Compiler is a tool for making JavaScript download and run faster. Instead of compiling from a source language to machine code, it compiles from JavaScript to better JavaScript. It parses your JavaScript, analyzes it, removes dead code and rewrites and minimizes what's left. It also checks syntax, variable references, and types, and warns about common JavaScript pitfalls.

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" src="https://i.creativecommons.org/l/by-sa/4.0/80x15.png" /></a>

[Closure Compiler](https://developers.google.com/closure/compiler/) can compile, optimize and bundle JavaScript code. You can use it for building your JavaScript apps for production. It is the most advanced JavaScript optimization tool. It generates smallest bundle and emits efficient JavaScript code by doing whole program analysis and optimization, removing closures, inlining function calls, reusing variable names and pre-computing constant expressions.

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

# Table of Contents
- [Introduction](#introduction)
- [Table of Contents](#table-of-contents)
- [Getting started](#getting-started)
  - [Installation](#installation)
  - [Using with Node](#using-with-node)
  - [Using with Webpack](#using-with-webpack)
  - [Using with Gulp](#using-with-gulp)
- [Compilation levels](#compilation-levels)
- [Advanced compilation](#advanced-compilation)
- [Flags](#flags)
- [Supported languages](#supported-languages)

# Getting started

Closure Compiler is written in Java, but it also has JavaScript port. Here we will use the latter.

- [google/closure-compiler](https://github.com/google/closure-compiler)
- [google/closure-compiler-js](https://github.com/google/closure-compiler-js)

## Installation

```bash
npm i google-closure-compiler-js
```

## Using with Node

```js
const compile = require('google-closure-compiler-js').compile;

const flags = {
  jsCode: [{src: 'const x = 1 + 2;'}],
};

const out = compile(flags);

console.info(out.compiledCode);  // will print 'var x = 3;\n'
```

## Using with Webpack

```js
const ClosureCompiler = require('google-closure-compiler-js').webpack;
const path = require('path');

module.exports = {
  entry: [
    path.join(__dirname, 'app.js')
  ],
  output: {
    path: path.join(__dirname, 'dist'),
    filename: 'app.min.js'
  },
  plugins: [
    new ClosureCompiler({
      options: {
        languageIn: 'ECMASCRIPT6',
        languageOut: 'ECMASCRIPT5',
        compilationLevel: 'ADVANCED',
        warningLevel: 'VERBOSE',
      },
    })
  ]
};
```

## Using with Gulp

```js
const compiler = require('google-closure-compiler-js').gulp();

gulp.task('script', function() {
  return gulp.src('./path/to/src.js', {base: './'})
      // your other steps here
      .pipe(compiler({
          compilationLevel: 'SIMPLE',
          warningLevel: 'VERBOSE',
          outputWrapper: '(function(){\n%output%\n}).call(this)',
          jsOutputFile: 'output.min.js',  // outputs single file
          createSourceMap: true,
        }))
      .pipe(gulp.dest('./dist'));
});
```

# Compilation levels

Compilation level is compiler setting which denotes optimizations level to be applied to JavaScript code.

`WHITESPACE_ONLY`

Removes comments, line breaks, unnecessary spaces and other whitespace.

`SIMPLE_OPTIMIZATIONS` *(default)*

Includes `WHITESPACE_ONLY` optimizations and renames variable names to shorter names to reduce the size of output code. It renames only variables local to functions, which means that it won't break references to third party code from global scope.

`ADVANCED_OPTIMIZATIONS`

Includes `SIMPLE_OPTIMIZATIONS` optimizations and performs aggressive transformations such as closures elimination, inlining function calls, reusing variable names, pre-computing constant expressions, tree-shaking, cross-module code motion and dead code elimination.

This kind of aggressive compression makes some assumptions about your code. If it doesn't conform to those assumptions, Closure Compiler will produce output that does not run.

# Advanced compilation

In `ADVANCED_OPTIMIZATIONS` compilation level Closure Compiler renames global variables, function names and properties and removes unused code. This can lead to output that will not run if your code doesn't follow certain rules.

# Flags
| Flag                             | Default | Usage |
|----------------------------------|---------|-------|
| angularPass | false | Generate $inject properties for AngularJS for functions annotated with @ngInject |
| applyInputSourceMaps | true | Compose input source maps into output source map |
| assumeFunctionWrapper | false | Enable additional optimizations based on the assumption that the output will be wrapped with a function wrapper. This flag is used to indicate that "global" declarations will not actually be global but instead isolated to the compilation unit. This enables additional optimizations. |
| checksOnly | false | Don't generate output. Run checks, but no optimization passes. |
| compilationLevel | SIMPLE | Specifies the compilation level to use.<br /> Options: WHITESPACE_ONLY, SIMPLE, ADVANCED |
| dartPass | false | |
| defines | null | Overrides the value of variables annotated with `@define`, an object mapping names to primitive types |
| env | BROWSER | Determines the set of builtin externs to load.<br /> Options: BROWSER, CUSTOM |
| exportLocalPropertyDefinitions | false | |
| generateExports | false | Generates export code for those marked with @export. |
| languageIn | ES6 | Sets what language spec that input sources conform to. |
| languageOut | ES5 | Sets what language spec the output should conform to. |
| newTypeInf | false | Checks for type errors using the new type inference algorithm. |
| outputWrapper | null | Interpolate output into this string, replacing the token `%output%` |
| polymerPass | false | Rewrite Polymer classes to be compiler-friendly. |
| preserveTypeAnnotations | false | |
| processCommonJsModules | false | Process CommonJS modules to a concatenable form, i.e., support `require` statements. |
| renamePrefixNamespace | | Specifies the name of an object that will be used to store all non-extern globals. |
| rewritePolyfills | false | Rewrite ES6 library calls to use polyfills provided by the compiler's runtime. |
| useTypesForOptimization | false | Enable or disable the optimizations based on available type information. Inaccurate type annotations may result in incorrect results. |
| warningLevel | DEFAULT | Specifies the warning level to use.<br /> Options: QUIET, DEFAULT, VERBOSE |
| jsCode | [] | Specifies the source code to compile. |
| externs | [] | Additional externs to use for this compile. |
| createSourceMap | false | Generates a source map mapping the generated source file back to its original sources. |

# Supported languages

| Language | Option name          | Input | Output |
|----------|----------------------|-------|--------|
| ES3      | `ECMASCRIPT3`        | ✅    |   ✅   |
| ES5      | `ECMASCRIPT5`        | ✅    |   ✅   |
| ES5      | `ECMASCRIPT5_STRICT` | ✅    |   ✅   |
| ES2015   | `ECMASCRIPT6`        | ✅    |        |
| ES2015   | `ECMASCRIPT6_STRICT` | ✅    |        |
| ES2015   | `ECMASCRIPT6_TYPED`  | ✅    |   ✅   |
