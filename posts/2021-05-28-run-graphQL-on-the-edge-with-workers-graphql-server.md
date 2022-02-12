---
title: run graphQL on the edge with workers-graphql-server
description: How to run a GraphQL on the edge with workers-graphql-server on Cloudflare Workers.
ate: 2021-05-28
tags:
  - cloudflare
  - graphql
  - workers
  - apollo
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3ncjrcfgmnx1qgdsr50h.png
layout: layouts/post.njk
---

### Generate project

```bash
wrangler generate my-graphql-server https://github.com/signalnerve/workers-graphql-server
```

### Set your account_id in wrangler.toml

```toml
name = "workers-graphql-server"
type = "webpack"
zone_id = ""
account_id = ""
route = ""
workers_dev = true
webpack_config = "webpack.config.js"
```

### Publish to Cloudflare’s network

```bash
cd my-graphql-server
wrangler publish
```

```
✨  Built successfully, built project size is 560 KiB.
✨  Successfully published your script to
 https://workers-graphql-server.anthonycampolo.workers.dev
```

![01-pokemon-graphql](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mxhczudxzft7uek4ft3d.png)