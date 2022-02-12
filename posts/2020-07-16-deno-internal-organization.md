---
title: deno internal organization
description: These are notes I took while watching Ryan Dahl's Deno Israel. He explains the internal organization of the Deno project.
date: 2020-07-16
tags:
  - deno
  - rust
  - v8
  - typescript
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/bau4l9mknnpwmt9735pg.png
layout: layouts/post.njk
---

These are notes I took while watching Ryan Dahl's Deno Israel talk which can be [found online here](https://www.youtube.com/watch?v=1b7FoBwxc7E). He explains the internal organization of the Deno project. All mentions of JavaScript are abbreviated as JS. I've included relevant links for terms that I think may be unfamiliar to most developers or that I was unfamiliar with myself.

# Origins

Even though it wasn't announced until 2018, Deno originated back in 2015 with a project called V8 worker which was a binding to V8 and Go. The basic idea was to disallow Go programmers from using V8's APIs to create random JS objects. Instead they would be forced into a message passing system that would pass messages between Go and V8.

# Rust Crates

Unlike Node, which is a monolithic project, Deno is organized into a collection of [Rust crates](https://crates.io/).

![01-deno-rust-crates](https://dev-to-uploads.s3.amazonaws.com/i/y226hs3vzuvtt62whtrb.jpg)

The Deno executable is what most people are referencing when talking about Deno. Not every project wants to use Node or Deno as a top level executable. There's many system where you might want to embed a JS runtime. Deno itself is a system with an embedded JS runtime.

### Why Avoid Monolithic Design?

Better separation of concerns inside the CLI
* Sub-systems can be tested independently
* Provides principled binding layer for CLI

There are many use cases for embedding a JS virtual machine including:
* Web servers customizable with JS (serverless)
* Databases using JS for map-reduce
* GUI applications like Electron

## rusty_v8

V8 is a C++ library with 800,000 lines of code. rusty_v8 is a framework for adding bindings to V8 C++. It includes:

* Zero cost Rust interface to V8
  * Objects manipulated in rusty_v8 are exactly the objects in C++.

* Safe bindings to V8
  * The type system forces [Local handles](https://stackoverflow.com/questions/9510822/what-is-the-design-rationale-behind-handlescope) to be created in a [HandleScope](https://v8docs.nodesource.com/node-0.8/d3/d95/classv8_1_1_handle_scope.html).
  * [Isolates](https://stackoverflow.com/questions/9131902/what-were-node-js-isolates-and-why-are-they-now-dead) are pegged to a thread for optimal usage.

* Ability to recompile V8 with different compilation settings
  * Prebuilt binaries created through github actions are provided for most users.
  * V8 source code is distributed inside a crate file. It does not depend on [depot_tools](https://dev.chromium.org/developers/how-tos/depottools).

## deno_core

C++ code that manipulates JS objects is often slower than the equivalent code in JS because boundary crossing from one language to the other is slow.

#### Node Problems

* Since there is no centralized [binding interface](https://en.wikipedia.org/wiki/Language_binding) in Node there is no way to monitor metrics or perform security checks.
* In Node, many callbacks are issued from C++ without being requested from the JS side such as push events instead of pull events. This gives users the opportunity to create code without [back pressure](https://medium.com/@jayphelps/backpressure-explained-the-flow-of-data-through-software-2350b3e77ce7).

#### Deno Solutions

Native bindings are done through the "Op" interface provided by deno_core. An op is a native function called from JS. Everything in the deno executable is built on top of ops. This provides superior performance to Node.
* **Binary data**: Parameters and return value are [ArrayBuffers](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)
* **Zero-copy**: ArrayBuffers pass pointers back and forth between JS and Rust
* **Integrated with event loop**: An op either completes synchronously with a result or returns a promise corresponding to a [Rust Future](https://docs.rs/futures/0.3.5/futures/)

#### Resources

A resource is similar to a [file descriptor in POSIX](https://www.gnu.org/software/libc/manual/html_node/Streams-and-File-Descriptors.html). It is an integer identifier given to JS to reference some object allocated in Rust such as an open socket or file. Resources are stored in the resource table and are removed from the table by `Deno.close()`

![02-deno-bindings](https://dev-to-uploads.s3.amazonaws.com/i/10jgpntop6yyj9pxlx5h.jpg)

## deno_typescript

deno_core is JS-only, so deno_typescript is where the TypeScript compiler is incorporated. This is currently a work in progress and should not be used in production.