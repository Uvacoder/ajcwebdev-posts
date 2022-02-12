---
title: a first look at oak
description: Oak is a middleware framework for Deno’s native HTTP server and Deno Deploy.
date: 2021-11-02
tags:
  - oak
  - deno
  - http
  - router
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tr4w9dv5rvsicbln1x4f.png
layout: layouts/post.njk
---

[Oak](https://oakserver.github.io/oak/) is a middleware framework for Deno’s native [HTTP server](https://dev.to/ajcwebdev/five-deno-web-frameworks-23k1) and [Deno Deploy](https://dev.to/ajcwebdev/a-first-look-at-deno-deploy-3hmc). It is influenced by [Koa](https://github.com/koajs/koa) (hence the anagram) and includes a middleware router inspired by [@koa/router](https://github.com/koajs/router).

All the code for this article can be found [on my GitHub](https://github.com/ajcwebdev/ajcwebdev-oak).

## Setup

Deno ships as a single executable with no dependencies. However, we will also need to install `deployctl` for Deno Deploy.

### Install Deno Executable

There are various ways of installing Deno depending on your operating system. If you are using Mac or Linux, the official Shell install script is recommend.

```bash
curl -fsSL https://deno.land/x/install/install.sh | sh
```

You can find a list of different installation methods on the official [deno.land documentation](https://deno.land/#installation) and the [`deno_install`](https://github.com/denoland/deno_install) repo.

### Install `deployctl`

With [`deployctl`](https://deno.com/deploy/docs/deployctl) you can run your Deno Deploy scripts on your local machine. Your scripts are run in the Deno CLI, with the correct TypeScript types for Deno Deploy. After installing Deno, `deployctl` can be installed using the following command.

```bash
deno install \
  --allow-read \
  --allow-write \
  --allow-env \
  --allow-net \
  --allow-run \
  --no-check \
  --force \
  https://deno.land/x/deploy/deployctl.ts
```

You may need to set the `PATH` variable as seen below but with your own path to the `deno` excecutable in place of `/Users/ajcwebdev`.

```bash
export PATH="/Users/ajcwebdev/.deno/bin:$PATH"
```

### Create Project Files

All we need is a directory containing an `index.js` file.

```bash
mkdir ajcwebdev-oak
cd ajcwebdev-oak
touch index.js
echo '.DS_Store' > .gitignore
```

## Create Deno Server

Before diving into Oak, it's useful to see the underlying server code they are building upon. The [Deno Standard Library](https://deno.land/std) has an [http](https://deno.land/std/http) module with a basic hello world application. Save the following code in `index.js`:

```typescript
// index.js

import { listenAndServe } from "https://deno.land/std@0.111.0/http/server.ts"

listenAndServe(
  ":8080",
  () => new Response("Hello from Deno on Localhost 8080")
)

console.log("Server running on localhost:8080")
```

### Run Deno server

```bash
deno run \
  --allow-net index.js \
  --watch \
  --no-check
```

```
Server running on localhost:8080
```

Open [localhost:8080](https://localhost:8080).

![01-hello-from-deno-localhost-8080](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5qhbpey6rzsx7a9fc5al.png)

## Create Oak Server

Oak is a third party module for Deno hosted on their [deno.land/x](https://deno.land/x) service. To import one of these modules, use the following format for code URLs:

```
https://deno.land/x/IDENTIFIER@VERSION/FILE_PATH
```

If you leave out the version it will default to the most recent version released for the module. The Deno docs recommend pinning to a specific version to avoid unexpected breaking changes. At the time of this writing, the current version of Oak is `https://deno.land/x/oak@v9.0.1/mod.ts`.

### Application Class

The `Application` class coordinates managing the HTTP server, running
middleware, and handling errors that occur when processing requests. Two methods are generally used:
* Middleware is added via the `.use()` method.
* The `.listen()` method starts the server and then processes requests with the registered middleware.

Once the server is open, before it starts processing requests, the application will fire a `"listen"` event, which can be listened for via the `.addEventListener()` method.

```js
// index.js

import { Application } from "https://deno.land/x/oak@v9.0.1/mod.ts"

const app = new Application()

app.use((ctx) => {
  ctx.response.body = "Hello from Oak on Localhost 8080"
})

app.addEventListener('listen', () => {
  console.log(`Server running on localhost:8080`)
})

app.listen({ port: 8080 })
```

The middleware is processed as a stack, where each middleware function can control the flow of the response. When the middleware is called, it is passed a context (`ctx`) and reference to the "next" method in the stack.
* `.response` accesses the `Response` object to form the response sent back to the requestor.
* The `.body` method returns a representation of the request body.

Return to [localhost:8080](https://localhost:8080).

![02-hello-from-oak-localhost-8080](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0vibugmkjovcgc0x8lfi.png)

### Respond with HTML

To respond with HTML instead of plain text, use `.headers.set` on `ctx.response` and set the `Content-Type` to `text/html`.

```js
// index.js

import { Application } from "https://deno.land/x/oak@v9.0.1/mod.ts"

const app = new Application()

app.use((ctx) => {
  ctx.response.body = "<h2>Hello from Oak on Localhost 8080</h2>"
  ctx.response.headers.set("Content-Type", "text/html")
})

app.addEventListener('listen', () => {
  console.log(`Server running on localhost:8080`)
})

app.listen({ port: 8080 })
```

Return to [localhost:8080](https://localhost:8080) to see the change.

![03-hello-with-html-localhost-8080](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g09my7pcmf719kqzhjqp.png)

## Router

The `Router` class produces middleware which can be used with an `Application` to enable routing based on the pathname of the request.

```js
// index.js

import { Application, Router } from "https://deno.land/x/oak@v9.0.1/mod.ts"

const router = new Router()
const app = new Application()

router.get("/", (ctx) => {
  ctx.response.body = "<h2>Hello from Router on Localhost 8080</h2>"
  ctx.response.headers.set("Content-Type", "text/html")
})

router.get("/about", (ctx) => {
  ctx.response.body = "<h2>This page tells you about stuff</h2>"
  ctx.response.headers.set("Content-Type", "text/html")
})

app.use(router.routes())
app.use(router.allowedMethods())

app.addEventListener('listen', () => {
  console.log(`Server running on localhost:8080`)
})

app.listen({ port: 8080 })
```

Return to [localhost:8080](https://localhost:8080) to see the change.

![04-hello-from-router-localhost-8080](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4egupo6ugjhozkdpd3rw.png)

Open [localhost:8080/about](https://localhost:8080/about) to see the new about page.

![05-about-page-localhost-8080](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/le15a8omlgirjovvfw91.png)

## Deno Deploy

[Deno Deploy](https://deno.com/deploy) is a distributed system that runs JavaScript, TypeScript, and WebAssembly at the edge, worldwide. It is a multi-tenant JavaScript engine running in [25 data centers across the world](https://deno.com/deploy/docs/regions).

### Run server with `deployctl run`

You can run your script like it would run on Deno Deploy with `deployctl`. Include `--no-check` so TypeScript doesn't explode in a giant fireball of errors. You can also add the `--watch` flag to automatically restart the script when any modules in the module graph change.

```bash
deployctl run \
  index.js \
  --no-check \
  --watch
```

If you return to [localhost:8080](https://localhost:8080) there will be no change in the application.

### Initialize GitHub Repository

```bash
git init
git add .
git commit -m "finally, another project named after a tree"
gh repo create ajcwebdev-oak
git push -u origin main
```

### Install the Deno Deploy GitHub App

We will be deploying this script from a URL on GitHub. Install the [Deno Deploy GitHub App](https://github.com/apps/deno-deploy).

![06-deno-deploy-github-app](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ee3rm7ipo08hdpoynu9k.png)

## Deploy Deno script

[Sign up](https://dash.deno.com/signin) for a Deno Deploy account and connect the account to your GitHub. You will be taken to a page for your projects.

![07-deno-deploy-projects](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rr6dgseaurjwmztb13d2.png)

### Create Deno Deploy Project

Click "New Project" and enter a name for your project.

![08-create-a-new-project](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/og4k861uc8mfxugb5kk7.png)

After creating your project you are given the option of using existing templates.

![09-ajcwebdev-oak-project-page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qagjh06bscbuvyl3hmhe.png)

### Connect GitHub repository

Since we will not be using a template, scroll down to connect the GitHub repository we just created.

![10-deploy-from-github](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bgluc2k2esfv4ny5hm8b.png)

Paste the raw GitHub url for your function.

![11-git-integration](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/aoq4i2myic9j9nhthzaf.png)

Click "Link" and go to [ajcwebdev-oak.deno.dev](https://ajcwebdev-oak.deno.dev).

![12-ajcwebdev-oak-deno-dev](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fqvoegko08vwyrr105oi.png)

You can use the project overview page to review your project's settings, add/remove domains, or view the deployment logs.

![13-project-overview-page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qdz492gxt09q32x24u1k.png)