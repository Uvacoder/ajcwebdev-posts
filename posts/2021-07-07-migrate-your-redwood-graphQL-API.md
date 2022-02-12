---
title: migrate your redwood graphQL API with graphQL-helix and envelop
description: GraphQL Helix is a collection of utility functions for building your own GraphQL HTTP server. Envelop is a library for extending a GraphQL server's execution layer.
date: 2021-07-07
tags:
  - redwoodjs
  - graphql
  - apollo
  - envelop
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9z5ff9ttw1vbhtueynrs.png
layout: layouts/post.njk
---

>*Note: As of Redwood v0.37, [GraphQL Helix and Envelop are included by default in the Redwood API instead of Apollo Server](https://community.redwoodjs.com/t/redwood-v0-37/2447#graphql-levels-way-up-envelophelix-2).*
>
>*If you are on v0.37 or later than you will not need the instructions in this article or the instructions in the official migration guide, [Using GraphQL Envelop+Helix in Redwood v0.35+](https://community.redwoodjs.com/t/using-graphql-envelop-helix-in-redwood-v0-35/2276).*
>
>*These instructions are only needed when upgrading from an older version of Redwood. This article is now mostly relevant as a historical snapshot of the development of the framework.*

Back in January the Redwood team decided to modify the internals of Redwood to [allow users to specify their own client instead of using Apollo Client](https://github.com/redwoodjs/redwood/pull/1639). Within weeks @marceloalves created a new package for a [React Query Provider](https://community.redwoodjs.com/t/react-query-provider-for-redwood/1709) and @Tobbe showed how you could [Switch to another GraphQL Client with graphql-hooks](https://community.redwoodjs.com/t/switch-to-another-graphql-client/1714).

### But what if you wanted to use your GraphQL server of choice?

Over the last two months [Dotan Simha](https://github.com/dotansimha) from [The Guild](https://the-guild.dev/) along with assistance from certified Redwood Whisperer @dthyresson have been working on similar modifications which will allow users to migrate away from Apollo Server to a different GraphQL server.

>*Hi, people of the Redwood! :)*
>
>*I created an initial PR for migrating from apollo-server-lambda to Envelop and GraphQL-Helix. The goal of this PR is to normalize the incoming HTTP requests and try to handle them in a generic way. Also, since the request is detached from the handler, we can use any GraphQL library for execution.*
>
>*While `graphql-helix` provides the basic pipeline and the initial request normalization, `envelop` provides the connection to the GraphQL execution, and allow to enrich the entire GraphQL execution pipeline with custom code (custom context building, parser cache, validation cache, tracing, metrics collection and more).*
>
>***[Dotan Simha](https://github.com/dotansimha)*** - *[Partial normalization of Lambda request (April 25, 2021)](https://github.com/redwoodjs/redwood/pull/2351)*

The initial PR, [Partial normalization of Lambda request for migration to Envelop](https://github.com/redwoodjs/redwood/pull/2351), laid the foundation for using GraphQL-Helix and Envelop.

* [GraphQL Helix](https://github.com/contrawork/graphql-helix) is a framework and runtime agnostic collection of utility functions for building your own GraphQL HTTP server.
* [Envelop](https://www.envelop.dev/) is a lightweight library allowing developers to easily develop, share, collaborate and extend their GraphQL execution layer. Envelop is the missing GraphQL plugin system.

>*Earlier this week I released [GraphQL Helix](https://github.com/contrawork/graphql-helix), a new JavaScript library that lets you take charge of your GraphQL server implementation.*
>
>*There's a couple of factors that pushed me to roll my own GraphQL server library:*
>
>* *I wanted to use bleeding-edge GraphQL features like `@defer`, `@stream` and `@live` directives.*
>* *I wanted to make sure I wasn't tied down to a specific framework or runtime environment.*
>* *I wanted control over how server features like persisted queries were implemented.*
>* *I wanted to use something other than WebSocket (i.e. SSE) for subscriptions.*
>
>*Unfortunately, popular solutions like [Apollo Server](https://github.com/apollographql/apollo-server), [express-graphql](https://github.com/graphql/express-graphql) and [Mercurius](https://github.com/mercurius-js/mercurius) fell short in one or more of these regards, so here we are.*
>
>***[Daniel Rearden](https://github.com/danielrearden)*** - *[Building a GraphQL server with GraphQL Helix (November 5, 2020)](https://dev.to/danielrearden/building-a-graphql-server-with-graphql-helix-2k44)*

## Create Redwood App

The code for this project can be found [on my GitHub](https://github.com/ajcwebdev/redwood-envelop).

```bash
yarn create redwood-app redwood-envelop
cd redwood-envelop
```

Open `schema.prisma` in `api/db` and add the following schema.

```
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider      = "prisma-client-js"
  binaryTargets = "native"
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  body      String
  createdAt DateTime @default(now())
}
```

### Provision a PostgreSQL database with Railway

First you need to [create a Railway account](http://railway.app/) and install the [Railway CLI](https://docs.railway.app/cli/installation).

```bash
railway login
railway init
railway add
```

Add a PostgreSQL plugin to your Railway project and then set the `DATABASE_URL` inside your `.env` file.

```bash
echo DATABASE_URL=`railway variables get DATABASE_URL` > .env
```

### Setup database with `prisma migrate dev` and `generate scaffold`

Running `yarn rw prisma migrate dev` generates the folders and files necessary to create a new migration. We will name our migration `posts-table`.

```bash
yarn rw prisma migrate dev --name posts-table
yarn rw g scaffold post
yarn rw dev
```

Open `http://localhost:8910/posts` to create a couple blog posts.

![01-redwood-envelop](upload://5QKjgy9UfCN3TPaex9LsBdUePcK.png)

## Configure project to use Envelop

Add `useEnvelop=true` to the `[experimental]` section in your `redwood.toml` config. This lets the `dev-server` know how to handle the response.

```toml
[web]
  port = 8910
  apiProxyPath = "/.redwood/functions"
[api]
  port = 8911
[browser]
  open = true
[experimental]
  esbuild = false
  useEnvelop = true
```

### Add `@redwoodjs/graphql-server` to `api` dependencies

```bash
yarn workspace api add @redwoodjs/graphql-server
```

### Define logger in `/api/src/lib/logger.js` and update import to `graphql-server` package

```javascript
// api/src/lib/logger.js

import { createLogger } from '@redwoodjs/graphql-server/logger'

export const logger = createLogger({
  options: { level: 'info', prettyPrint: true },
})
```

### Add `graphql-server` package to `graphql.js` function and `loggerConfig` to `createGraphQLHandler`

```javascript
// api/src/functions/graphql.js

import {
  createGraphQLHandler,
  makeMergedSchema,
  makeServices,
} from '@redwoodjs/graphql-server'

import schemas from 'src/graphql/**/*.{js,ts}'
import { db } from 'src/lib/db'
import { logger } from 'src/lib/logger'
import services from 'src/services/**/*.{js,ts}'

export const handler = createGraphQLHandler({
  loggerConfig: {
    logger,
    options: {
      operationName: true,
      tracing: true
    }
  },
  schema: makeMergedSchema({
    schemas,
    services: makeServices({ services }),
  }),
  onException: () => {
    db.$disconnect()
  },
})
```

Apollo plugins are not currently supported and must be removed. However, there may be equivalent [Envelop plugins](https://www.envelop.dev/plugins). These can be added in the `createGraphQLHandler` configuration options in `extraPlugins`. `extraPlugins` accepts an array of plugins.

### Currently used plugins

```javascript
'@envelop/depth-limit'
'@envelop/disable-introspection'
'@envelop/filter-operation-type'
'@envelop/parser-cache'
'@envelop/validation-cache'
'@envelop/use-masked-errors'
```

Change the imports to use the new `graphql-server` package if your services raise any errors based on `ApolloError` such as `UserInputError` or `ValidationError`.

```javascript
import { UserInputError } from '@redwoodjs/graphql-server'
```

If you have any other `@redwoodjs/api` imports in your project make sure to change them to `@redwoodjs/graphql-server`.

### Restart development server and send a query

```bash
yarn rw dev
```

![02-testing-localhost](upload://vm9xHKYsZZMhQOFeGKoApKtpq9y.png)

### Add HomePage and PostsCell

```bash
yarn rw g page home /
yarn rw g cell posts
```

```jsx
// web/src/components/PostsCell/PostsCell.js

export const QUERY = gql`
  query PostsQuery {
    posts {
      id
      title
      body
      createdAt
    }
  }
`

export const Loading = () => <div>Almost there...</div>
export const Empty = () => <div>WHERE'S THE POSTS?</div>
export const Failure = ({ error }) => <div>{error.message}</div>

export const Success = ({ posts }) => {
  return posts.map((post) => (
    <article key={post.id}>
      <header>
        <h2>{post.title}</h2>
      </header>
      <p>{post.body}</p>
      <div>{post.createdAt}</div>
    </article>
  ))
}
```

```jsx
// web/src/pages/HomePage/HomePage.js

import PostsCell from 'src/components/PostsCell'

const HomePage = () => {
  return (
    <>
      <h1>Redwood+Envelop</h1>
      <PostsCell />
    </>
  )
}

export default HomePage
```

## Setup Netlify Deploy

Generate the configuration file needed for deploying to Netlify with the following setup command.

```bash
yarn rw setup deploy netlify
```

### Push Project to GitHub

Create a blank repository at [repo.new](https://repo.new) and push the project to your GitHub.

```bash
git init
git add .
git commit -m "the guilded age of redwood"
git remote add origin https://github.com/ajcwebdev/redwood-envelop.git
git push -u origin main
```

### Connect Repo to Netlify

Go to [Netlify](https://netlify.com/) and connect the repo. Include the `DATABASE_URL` environment variable and add `?connection_limit=1` to the end of the connection string. You can also give your site a custom domain such as `redwood-envelop`.

### Open [redwood-envelop.netlify.app](https://redwood-envelop.netlify.app/)

![03-redwood-envelop-netlify-deploy](upload://itA5jnPyY51Anmk8z2krz0BDC4a.png)

### Test [the API](https://redwood-envelop.netlify.app/.netlify/functions/graphql) with a query

```graphql
query getPosts {
  posts {
    id
    title
    body
    createdAt
  }
}
```

![04-testing-netlify-endpoint](upload://4O6ik79u7MF4vVuru7AfAWWPPZs.jpeg)

## Is Redwood still Redwood without Apollo?

RedwoodJS was originally architected around Apollo Client on the `web` side and Apollo Server on the `api` side. These two libraries were fundamental to the development of not only Redwood but the entire GraphQL ecosystem. Apollo itself was born from the ashes of the [Meteor Development Group](https://en.wikipedia.org/wiki/Meteor_(web_framework)#History).

Meteor pursued a similar philosophy of fullstack JavaScript now employed by Redwood. By bringing the decoupled Apollo pieces of client and server together into a single fullstack application it felt like the original dream of Meteor was finally coming to fruition. But GraphQL is itself about decoupling the frontend from the backend so that one side is never too heavily tied to the other.

This has allowed Redwood to pursue other GraphQL clients and servers that continue to evolve and improve and engage with the open source developer community. The free market of repositories is alive and well, and we will see many more experiments in the coming future.