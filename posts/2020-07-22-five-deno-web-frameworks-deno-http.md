---
title: five deno web frameworks - deno http
description: There are many frameworks currently being built for Deno that take inspiration from Node frameworks and frameworks from other languages.
date: 2020-07-22
tags:
  - deno
  - node
  - http
  - frameworks
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/36xnwmd3259gnh8h5e1q.png
layout: layouts/post.njk
---

When building a web server with NodeJS, most developers use a framework such as Express, Koa, or (the newly resurrected) Hapi. There are many frameworks currently being built for Deno that take inspiration from Node frameworks and frameworks from other languages like Flask or Sinatra.

Popularity is not always the best metric, but when it comes to open source projects it signals a greater level of developers are working with a framework which means more bug fixes, more feature requests, and more resources online to learn that framework. The five most popular frameworks right now are:

1. [Oak](https://github.com/oakserver/oak)
2. [Drash](https://github.com/drashland/deno-drash)
3. [Servest](https://github.com/keroxp/servest)
4. [Opine](https://github.com/asos-craigmorten/opine)
5. [abc](https://github.com/zhmushan/abc)

Craig Morten's excellent overview [What Is The Best Deno Web Framework?](https://dev.to/craigmorten/what-is-the-best-deno-web-framework-2k69) explores these five frameworks along with a range of others.

## Deno HTTP

Before diving into the frameworks it's useful to see the underlying server code they are building upon. The [Deno Standard Library](https://deno.land/std) has an [http](https://deno.land/std/http) module with a basic hello world application. First make sure you have [Deno installed](https://deno.land/#installation).

### Create project directory and `hello.ts`

```bash
mkdir ajcwebdev-deno
cd ajcwebdev-deno
touch hello.ts
```

Save the following code in `hello.ts`:

```typescript
// hello.ts

import { serve } from "https://deno.land/std/http/server.ts"

const server = serve({ port: 8000 })

for await (const req of server) {
  req.respond({
    body: "<h1>Hello World<h1>"
  })
}
```

### Run Deno server

```bash
deno run --allow-net hello.ts
```

Open your browser to `localhost:8000`

![01-deno-hello-world](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/737nt5xcqnu39on700wu.png)

We can contrast this with a similar example in Node. Save the following code in a file called `hello.js`:

```javascript
// hello.js

const http = require('http');

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/html');
  res.end('<h1>Hello World</h1>');
});

server.listen(3000);
```

### Run Node server

```bash
node hello.js
```

Open your browser to `localhost:3000`.

![02-node-hello-world](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gyh9swapvgo2sqv4rhrh.png)