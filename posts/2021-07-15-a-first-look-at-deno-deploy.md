---
title: a first look at deno deploy
description: Deno Deploy is a distributed system that runs JavaScript, TypeScript, and WebAssembly at the edge, worldwide. It is a multi-tenant JavaScript engine running in 25 data centers across the world.
date: 2021-07-15
tags:
  - deno
  - serverless
  - github
  - deployctl
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/s18d6gxjiwevtheab9bf.png
layout: layouts/post.njk
---

[Deno Deploy](https://deno.com/deploy) is a distributed system that runs JavaScript, TypeScript, and WebAssembly at the edge, worldwide. It is a multi-tenant JavaScript engine running in [25 data centers across the world](https://deno.com/deploy/docs/regions) that integrates cloud infrastructure with the V8 virtual machine, allowing users to quickly script distributed HTTPS servers.

## Create Deno script

Create an `index.js` file for our function which will respond to all incoming HTTP requests with plain text and a `200 OK` HTTP status.

```bash
touch index.js
```

Deno Deploy lets you listen for incoming HTTP requests using the same [`FetchEvent` API](https://deno.com/deploy/docs/runtime-fetchevent) used in Service Workers.

```javascript
// index.js

addEventListener("fetch", (event) => {
  event.respondWith(
    new Response("ajcwebdev-deno", {
      status: 200,
      headers: {
        "content-type": "text/plain"
      },
    }),
  )
})
```

This API works by registering a listener using the global `addEventListener`, with the event type `fetch`. The second parameter provided to `addEventListener` is a callback that is called with a `FetchEvent` property for every request.

### Create GitHub repo

Go to [repo.new](https://repo.new) and create a new repository.

```bash
git init
git add .
git commit -m "Just a denonstration"
git remote add origin https://github.com/ajcwebdev/ajcwebdev-deno.git
git push -u origin main
```

### Install the Deno Deploy GitHub App

We will be deploying this script from a URL on GitHub. Install the [Deno Deploy GitHub App](https://github.com/apps/deno-deploy).

![01-deno-deploy-github-app](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ee3rm7ipo08hdpoynu9k.png)

## Deploy Deno script

[Sign up](https://dash.deno.com/signin) for a Deno Deploy account and connect the account to your GitHub. You will be taken to a page for your projects.

![02-deno-deploy-projects](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vdtdsaacp2a5cqdvo80k.png)

### Create Deno Deploy Project

Click "New Project" and enter a name for your project.

![03-create-a-new-project](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/594ajj5o9u9hwxxgl086.png)

After creating your project you can use the hello world example template or deploy a simple chat server with the [BroadcastChannel API](https://deno.com/deploy/docs/runtime-broadcast-channel).

![04-ajcwebdev-deno-project-page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dxd9jsksbvn2iw8g3pxu.png)

### Connect GitHub repository

Scroll down to connect the GitHub repository we just created.

![05-deploy-from-github](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wluocbtl9ijbq7kyj2mi.png)

Paste the raw GitHub url for your function.

![06-git-integration](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/75qq91wec3k69a2rzk0y.png)

Go to [ajcwebdev-deno.deno.dev](https://ajcwebdev-deno.deno.dev).

![07-ajcwebdev-deno-dev](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rtzn410i15i2lyaom6sm.png)

You can use the project overview page to review your project's settings, add/remove domains, or view the deployment logs.

![08-project-overview-page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/p8il2h458e5d9i2dejgc.png)

## Send HTML and JSX

Change `content-type` to `text/html` and include an `<h2>` header in the `Response`.

```javascript
// index.js

addEventListener("fetch", (event) => {
  event.respondWith(
    new Response("<h2>ajcwebdev-deno</h2>", {
      status: 200,
      headers: {
        "content-type": "text/html"
      },
    }),
  )
})
```

![09-html-hello-world](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fpu9s3srcrdsbyl941s3.png)

Deno Deploy supports JSX (and TSX) out of the box so you don't need an additional transform step. It is important that you use `.jsx` or `.tsx` file extensions for Deno Deploy to transform JSX code.

```bash
mv index.js index.jsx
```

Deno Deploy uses the `h` factory function instead of `React.createElement`. `renderToString` generates an html string from JSX components.

```jsx
// index.jsx

import { h } from "https://x.lcas.dev/preact@10.5.12/mod.js"
import { renderToString } from "https://x.lcas.dev/preact@10.5.12/ssr.js"

function App() {
  return (
    <html>
      <head>
        <title>ajcwebdev-deno</title>
      </head>
      <body>
        <h1>ajcwebdev-deno</h1>
      </body>
    </html>
  )
}

addEventListener("fetch", (event) => {
  const response = new Response(
    renderToString(<App />), {
      headers: {
        "content-type": "text/html; charset=utf-8"
      },
    }
  )
  event.respondWith(response)
})
```

You will need to unlink and link the GitHub URL in your project since we changed the file name.

![10-jsx-hello-world](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hu6t6t3z2jph6zmoihxn.png)

## Test locally with `deployctl`

With [`deployctl`](https://deno.com/deploy/docs/deployctl) you can run your Deno Deploy scripts on your local machine. Your scripts are run in the Deno CLI, with the correct TypeScript types for Deno Deploy.

### Install `deployctl`

If you already have [Deno installed](https://deno.land/#installation) then `deployctl` can be installed using the following command.

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

You can run your script like it would run on Deno Deploy. Include `--no-check` so TypeScript doesn't explode in a giant fireball of errors.

```bash
deployctl run \
  https://raw.githubusercontent.com/ajcwebdev/ajcwebdev-deno/master/index.jsx \
  --no-check
```

```
Listening on http://0.0.0.0:8080
```

![11-deployctl-run](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/79hlhkbmq27of6mbfsjq.png)

For local development, you can also add the `--watch` flag to automatically restart the script when any modules in the module graph change.

```bash
deployctl run \
  https://raw.githubusercontent.com/ajcwebdev/ajcwebdev-deno/master/index.jsx \
  --no-check \
  --watch
```