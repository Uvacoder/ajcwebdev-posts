---
title: deno - secure runtime for javascript
description: Deno is a secure runtime for JavaScript and TypeScript.
date: 2020-05-31
tags:
  - deno
  - javascript
  - typescript
  - node
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/metayyl3pb53nfxjjvxi.jpg
layout: layouts/post.njk
---

Deno is a secure runtime for JavaScript and TypeScript. The original creator of Node, Ryan Dahl, started the project intending to build a modern, server-side JavaScript framework that incorporates the knowledge he gained building out the initial Node ecosystem, while also incorporating many of the features of JavaScript that have become standardized since 2015.

In 10 Things I Regret About Node, Ryan Dahl gave the following list of 8 regrets:

1. Not sticking with Promises
2. Security
3. The Build System: GYP
4. package.json
5. require("somemodule") is not specific
6. node_modules
7. require("module") without the extension ".js"
8. index.js

To translate for non-Node developers:

* Node is fundamentally divergent from the standardization of JavaScript in the last 5 years
* Node has too much access to the computer it's running on
* The module/package system has become a bit of a nightmare

![sad-deno](https://cdn.substack.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2Fb32fea2d-ac02-4670-9954-010c98b03500_300x300.gif)

*[I imagine him standing in the rain at night - stoically facing the dark battle that is software engineering.](https://github.com/denolib/animated-deno-logo/)*

>*Why would you do this? What's the point, isn't this disrupting the community? Isn't this bifurcating the module system? Nothing good can come from this, isn't this a Python 3 situation where we're going to end up with 10 years of difficulty with two different forks of the language?*
>
>*And I think that those are good concerns, but I think it's time to reevaluate.*
>
>*JavaScript has changed quite a bit in the last 10 years since I originally sat down to do Node. The main important features that have been added are typed arrays which give you true access to binary data. That was not around in 2009. ES Modules, the module system is now standardized.*
>
>***Ryan Dahl - [Deno, a new way to JavaScript (August 28, 2019)](https://www.youtube.com/watch?v=HjdJzNoT_qg)***

![Image](https://sedaily-topics.s3.amazonaws.com/topic_images/0_9104501175592736.jpg)

## Deno Principles

1. Secure by default. No file, network, or environment access, unless explicitly enabled.
2. Supports TypeScript out of the box.
3. Ships only a single executable file.
4. Built-in utilities including dependency inspector (deno info) and code formatter (deno fmt).
5. Standard modules are audited and guaranteed to work with Deno: deno.land/std

## Rust and Tokio

Node is a JavaScript runtime implemented in C++. In contrast, Deno is a TypeScript runtime implemented in Rust. It uses a new event loop in Rust called Tokio. Ryan hadn’t been coding much JavaScript since around 2012 when he moved on from Node.

If you know a lot of programmers you almost certainly know at least one guy who really, really likes Rust. Ryan is that guy, and Deno is what happens when that guy makes a new Node. Like Node, Deno is a strange mix of somewhat un-JavaScripty programmers working together with diehard JavaScript monoglots to progress the web platform.

## Oak

Oak is a middleware framework for Deno's http server inspired by Koa. Koa itself aimed to be a more modern implementation of the Express framework and included contributors from Express. Middleware enables communication and management of data in distributed applications.

Types of middleware include web servers, application servers, content management systems, and similar tools that support application development and delivery. Oak also includes a router middleware inspired by koa-router for defining how an application’s endpoints (URIs) respond to the client's requests.

When learning Node it is extremely common to learn Express along with it and many never learn to use Node without Express. Many programmers coming to Deno will already have this mental model which means Oak is likely to be a very consequential framework for the Deno ecosystem.