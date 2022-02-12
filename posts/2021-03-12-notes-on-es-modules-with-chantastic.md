---
title: notes on es modules with chantastic
description: A collection of examples demonstrating static and dynamic imports, export lists, and import aliases.
date: 2021-03-12
tags:
  - javascript
  - esm
  - node
  - esbuild
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jp1vudlgyydaexu3c9wq.png
layout: layouts/post.njk
---

ES Modules (JavaScript Modules, if you like) help us structure, organize and isolate code.

[These examples](https://github.com/chantastic/esmodule-reference-course) focus on the ES Modules features available in a Node.js environment. Most JavaScript applications today (early 2021) still go through some form of bundling before being sent to the browser. The features we cover should be common to all Node.js bundling tools (Webpack, Rollup, esbuild) (the latest LTS) Node.js environments (v14+).

### index.js - main file

* This is where all exercises are run
* Examples in `readme.md` can be copy/pasted into `index.js`

### ./modules - prepared module files

* The names relate to the type of content inside
* Various examples might use these prepared modules to explore a concept

### assignment.mjs - examples

* `./modules/assignment.mjs` is an empty module.

## 1. Import a module from the file system

Modules are imported using the `import` keyword and a string path to that module. This is predominantly done at the opening of a file. We can import any module that exists on the file system.

### Import the `assignment.mjs` module using the `import` keyword and file path

```javascript
// index.js

import "./modules/assignment.mjs"
```

## 2. Make your module leaky

The first thing to know about modules is that they leak. They don't perfectly encapsulate all code. Global code is global code, even if in a module. This might sounds bad (and it can be) but it's an important feature.

### Add globally executing code to the `assignment.mjs` module

Globally executing code can be `console.log("booo!!")` or an assignment like `global.leak = "oh no!"`.

```javascript
// modules/assignment.mjs

console.log("sah dude")
global.leak = "oops"
```

```javascript
// index.js

import "./modules/assignment.mjs"

console.log(global.leak)
```

```
sah dude
oops
```

## 3. Import a module dynamically

The `import` keyword has two variants: static and dynamic. You can change a static import to a dynamic import by adding parenthesis around the path string.

```javascript
import "./modules/assignment.mjs" // static
import("./modules/assignment.mjs") // dynamic
```

### Change the static import to a dynamic import

Dynamic `import` returns a `Promise`, so try handling that promise using `await`.

```javascript
// index.js

await import("./modules/assignment.mjs")

console.log(global.leak)
```

Use `.then()` to resolve the Promise.

```javascript
// index.js

import("./modules/assignment.mjs")
.then(
  () => console.log(global.leak)
)
```

```
sah dude
oops
```

## 4. Add a function declaration and variable to the module and export using an export list

Global code is executed at `import` but variables and function declarations are not. Even though a function or variable might exist in an imported module, it can't be accessed outside of that module.

### Add a variable and function declaration to the module

```javascript
// modules/assignment.mjs

let fallbackName = "there"

function greet(name = fallbackName) {
  return `Hey, ${name}!`
}
```

### Use `export` list to export `greet` function

We can export anything defined in our module by adding it to the comma separated `export` list.

```javascript
// modules/assignment.mjs

let fallbackName = "there"

function greet(name = fallbackName) {
  return `Hey, ${name}!`
}

export { fallbackName, greet }
```

Anything exported can also use the same list syntax, `{}`, for `import`. Importing specific imports from a module requires the `from` keyword before the path string.

```javascript
// index.js

import { greet } from "./modules/assignment.mjs"

console.log(greet())
```

```
Hey, there!
```

`export` list is commonly at the end of a file to guarantee that everything exported — or referenced — already exists.

### Export and import just the `greet` function using an `export` list

```javascript
// modules/assignment.mjs

let fallbackName = "dude"

function greet(name = fallbackName) {
  return `Sah, ${name}`
}

export { greet }
```

```javascript
// index.js

import { greet } from "./modules/assignment.mjs"

console.log(greet())
```

```
Sah, dude
```

`greet` still has access to `fallbackName`, even though `fallbackName` isn't exported.

### 5. Use `as` to alias (or rename) imports and exports

Modules might not share object syntax with modules but they still allow for aliasing (or renaming) of variables and functions using the `as` keyword.

```javascript
// modules/assignment.mjs

export { aGoodLocalName as aBetterExportedName }
```

It works identically on both `import` and `export` side.

```javascript
// index.js

import {
  aBetterExportedName as anEvenBetterContextualName
} from "./modules/assignment"
```

### Rename the `greet` function at `export`

At import, use the new function name you've exported and then rename it back to `greet` at `import`.

```javascript
// modules/assignment.mjs

let fallbackName = "dude";

function greet(name = fallbackName) {
  return `Sah, ${name}`
}

export { greet as sahDude }
```

```javascript
// index.js

import { sahDude as greet } from "./modules/assignment.mjs"

console.log(greet())
```

```
Sah, dude
```