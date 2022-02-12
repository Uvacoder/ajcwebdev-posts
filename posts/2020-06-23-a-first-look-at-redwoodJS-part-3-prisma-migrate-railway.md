---
title: a first look at redwoodJS part 3 - prisma migrate, railway
description: Get our database up and running and learn to create, retrieve, update, and destroy blog posts
date: 2020-06-23
tags:
  - redwoodjs
  - react
  - postgres
  - railway
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/7x38o4nssnlf0d9hdo5h.png
layout: layouts/post.njk
---

>*What I wanted was to codify and standardize the types of things that we were already doing and just remove choice and remove friction and just give people the ability to sit down and say:*
>
>*"All right, I know these technologies already; I have the prerequisite knowledge to do this."*
>
> ***Tom Preston-Werner*** - *[Full Stack Radio](https://www.fullstackradio.com/episodes/138)*

# Part 3 - Prisma Migrate, Railway

* [Part 1](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-1017) - created our first RedwoodJS application and pages
* [Part 2](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-part-2-44ph) - created links to our different page routes and a reusable layout for our site

In this part we'll get our database up and running and learn to create, retrieve, update, and destroy (CRUD) blog posts.

## 3.1 Railway

Railway is an infrastructure provider that allows you to develop code in the cloud and deploy from anywhere. When you run apps on Railway, configs are intelligently provided to you based on your project and environment. We can spin up our database with the Railway CLI or the Railway dashboard and I will demonstrate both. First you need to [create a Railway account](http://railway.app/).

### Use the Railway CLI

Install the [Railway CLI](https://docs.railway.app/cli/installation).

```bash
railway login
```

Initialize project.

```bash
railway init
```

Give your project a name.

```
✔ Create new Project
✔ Enter project name: ajcwebdev-redwood
✔ Environment: production
🎉 Created project ajcwebdev-redwood
```

### Provision PostgreSQL

Add a plugin to your Railway project.

```bash
railway add
```

Select PostgreSQL.

```
✔ Plugin: postgresql 
🎉 Created plugin postgresql
```

### Set environment variable

Create a `.env` file with your DATABASE_URL.

```bash
echo DATABASE_URL=`railway variables get DATABASE_URL` > .env
```

### Use the Railway Dashboard

![01-](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qb4q3381cb8w8ek5mrks.png)

### ⌘ + K

![02-](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jhr9oamt57al7gyew4yc.png)

### Provision PostgreSQL

![03-](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vqa630ipwswqkbv4uijp.png)

### Select PostgreSQL on left sidebar

![04-](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ofjwi80trb0yhz6d1xnm.png)

### Click Connect

![05-](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1qt68etqubrue4tnktg1.png)

### Copy paste your database URL (this db was deleted)

![06-](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/60ogrpz247ex6jpcdnrz.png)

### Set environment variable

Open your `.env` file. Copy the environment variable from the Railway dashboard and set it in place of `YOUR_URL_HERE` in the below example.

```
DATABASE_URL=YOUR_URL_HERE
```

## 3.2 `schema.prisma`

So far we've been working in the `web` folder. In our `api` folder there is a folder called `db` for our Prisma schema.

```
└── api
    └── db
        ├── schema.prisma
        └── seeds.js
```

Prisma is an [ORM](https://www.prisma.io/docs/understand-prisma/why-prisma) that provides a type-safe API for submitting database queries which return JavaScript objects. It was selected by Tom in the hopes of emulating [Active Record's](https://guides.rubyonrails.org/active_record_basics.html) role in Ruby on Rails.

The Prisma schema file is the main configuration file for your Prisma setup. It is typically called `schema.prisma`

```
// api/db/schema.prisma

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

generator client {
  provider      = "prisma-client-js"
  binaryTargets = "native"
}

model UserExample {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
}
```

In order to set up Prisma Client, you need a Prisma schema file with:
* your database connection
* the Prisma Client `generator`
* at least one `model`

Change the database provider from `sqlite` to `postgresql` and delete the default `UserExample` model. Make a `Post` model with an `id`, `title`, `body`, and `createdAt` time.

```
// api/db/schema.prisma

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

### `seeds.js`

`seeds.js` is used to populate your database with any data that needs to exist for your app to run at all (maybe an admin user or site configuration).

```js
// api/db/seeds.js

const { PrismaClient } = require('@prisma/client')
const dotenv = require('dotenv')

dotenv.config()
const db = new PrismaClient()

async function main() {
  console.warn('Please define your seed data.')

  // return Promise.all(
  //   data.map(async (user) => {
  //     const record = await db.user.create({
  //       data: { name: user.name, email: user.email },
  //     })
  //     console.log(record)
  //   })
  // )
}

main()
  .catch((e) => console.error(e))
  .finally(async () => {
    await db.$disconnect()
  })
```

## 3.3 `redwood prisma migrate`

Running `yarn rw prisma migrate dev` generates the folders and files necessary to create a new migration. We will name our migration `nailed-it`.

```bash
yarn rw prisma migrate dev --name nailed-it
```

```
Running Prisma CLI:
yarn prisma migrate dev --name nailed-it --preview-feature --schema "/Users/ajcwebdev/projects/ajcwebdev-redwood/api/db/schema.prisma" 

Prisma schema loaded from db/schema.prisma
Datasource "DS": PostgreSQL database "railway", schema "public" at "containers-us-west-1.railway.app:5884"

The following migration(s) have been applied:

migrations/
  └─ 20210307113114_nailed_it/
    └─ migration.sql

✔ Generated Prisma Client (2.16.1) to ./../node_modules/@prisma/client in 89ms

Everything is now in sync.
```

The `migrate` command creates and manages database migrations. It can be used to create, apply, and rollback database schema updates in a controlled manner.

```
└── api
    └── db
        ├── migrations
        │   ├── 20210307113114_nailed_it
        │   │   └── migration.sql
        │   └── migration_lock.toml
        ├── schema.prisma
        └── seeds.js
```

### `migration.sql`

```sql
CREATE TABLE "Post" (
    "id" INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
    "title" TEXT NOT NULL,
    "body" TEXT NOT NULL,
    "createdAt" DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

## 3.4 `redwood generate scaffold`

A scaffold quickly creates a CRUD interface for a model by generating all the necessary files and corresponding routes.

```bash
yarn rw g scaffold post
```

This will generate pages, SDL's, services, layouts, cells, and components based on a given database schema Model.

```
✔ Generating scaffold files...
  ✔ Successfully wrote file `./web/src/components/Post/EditPostCell/EditPostCell.js`
  ✔ Successfully wrote file `./web/src/components/Post/Post/Post.js`
  ✔ Successfully wrote file `./web/src/components/Post/PostCell/PostCell.js`
  ✔ Successfully wrote file `./web/src/components/Post/PostForm/PostForm.js`
  ✔ Successfully wrote file `./web/src/components/Post/Posts/Posts.js`
  ✔ Successfully wrote file `./web/src/components/Post/PostsCell/PostsCell.js`
  ✔ Successfully wrote file `./web/src/components/Post/NewPost/NewPost.js`
  ✔ Successfully wrote file `./api/src/graphql/posts.sdl.js`
  ✔ Successfully wrote file `./api/src/services/posts/posts.js`
  ✔ Successfully wrote file `./api/src/services/posts/posts.scenarios.js`
  ✔ Successfully wrote file `./api/src/services/posts/posts.test.js`
  ✔ Successfully wrote file `./web/src/scaffold.css`
  ✔ Successfully wrote file `./web/src/layouts/PostsLayout/PostsLayout.js`
  ✔ Successfully wrote file `./web/src/pages/Post/EditPostPage/EditPostPage.js`
  ✔ Successfully wrote file `./web/src/pages/Post/PostPage/PostPage.js`
  ✔ Successfully wrote file `./web/src/pages/Post/PostsPage/PostsPage.js`
  ✔ Successfully wrote file `./web/src/pages/Post/NewPostPage/NewPostPage.js`
✔ Adding layout import...
✔ Adding set import...
✔ Adding scaffold routes...
✔ Adding scaffold asset imports...
```

Look at all the stuff I'm *not* doing! Open the browser and enter `localhost:8910/posts`.

![07-posts-page](https://dev-to-uploads.s3.amazonaws.com/i/lhz94ndsl9olzxe9fa43.jpg)

We have a new page called Posts with a button to create a new post. If we click the new post button we are given an input form with fields for title and body.

![08-new-post-form](https://dev-to-uploads.s3.amazonaws.com/i/wzd7fuj7095skqrvcjc8.jpg)

We were taken to a new route, `/posts/new`. Let's create a blog post about everyone's favorite dinosaur.

![09-new-deno-post](https://dev-to-uploads.s3.amazonaws.com/i/4232p8zg5u8p3lhrumlc.jpg)

If we click the save button we are brought back to the posts page.

![10-deno-post-created](https://dev-to-uploads.s3.amazonaws.com/i/txd7ni7eggb7pmw1tdtm.jpg)

We now have a table with our first post.

![11-deno-post-edit](https://dev-to-uploads.s3.amazonaws.com/i/hw56jz0iedg3lzmwfzh6.jpg)

When we click the edit button we are taken to a route for the individual post that we want to edit. Each post has a unique id.

Lets add another blog post:

![12-fauna-post](https://dev-to-uploads.s3.amazonaws.com/i/lt8ao1j6n93ntb0jlevm.jpg)

And one more:

![13-next-post](https://dev-to-uploads.s3.amazonaws.com/i/b22d01flnjys1nh355zs.jpg)

![14-all-posts](https://dev-to-uploads.s3.amazonaws.com/i/5z5ecpynu0qhkm4jo62y.jpg)

## 3.5 `api/src`

We've seen the `prisma` folder under `api`, now we'll look at the `src` folder containing our GraphQL code. Redwood comes with GraphQL integration built in to make it easy to get our client talking to our serverless functions.

```
└── api
    └── src
        ├── functions
        │   └── graphql.js
        ├── graphql
        │   └── posts.sdl.js
        ├── lib
        │   ├── auth.js
        │   ├── db.js
        │   └── logger.js
        └── services
            └── posts
                ├── posts.js
                ├── posts.scenarios.js
                └── posts.test.js
```

## 3.6 `posts.sdl.js`

GraphQL schemas for a service are specified using the GraphQL ([schema definition language](https://graphql.org/learn/schema/)) which defines the API interface for the client. Our `schema` has five object types each with their own fields and types.

```graphql
# api/src/graphql/posts.sdl.js

type Post {
  id: Int!
  title: String!
  body: String!
  createdAt: DateTime!
}

type Query {
  posts: [Post!]!
  post(id: Int!): Post
}

input CreatePostInput {
  title: String!
  body: String!
}

input UpdatePostInput {
  title: String
  body: String
}

type Mutation {
  createPost(input: CreatePostInput!): Post!
  updatePost(id: Int!, input: UpdatePostInput!): Post!
  deletePost(id: Int!): Post!
}
```

## 3.7 `posts.js`

`services` contain business logic related to your data. A service implements the logic of talking to the third-party API. This is where your code for querying or mutating data with GraphQL ends up. Redwood will automatically import and map resolvers from the `services` file onto your SDL.

```js
// api/src/services/posts/posts.js

import { db } from 'src/lib/db'
import { requireAuth } from 'src/lib/auth'

export const beforeResolver = (rules) => {
  rules.add(requireAuth)
}

export const posts = () => {
  return db.post.findMany()
}

export const post = ({ id }) => {
  return db.post.findUnique({
    where: { id },
  })
}

export const createPost = ({ input }) => {
  return db.post.create({
    data: input,
  })
}

export const updatePost = ({ id, input }) => {
  return db.post.update({
    data: input,
    where: { id },
  })
}

export const deletePost = ({ id }) => {
  return db.post.delete({
    where: { id },
  })
}
```

## 3.8 `db.js`

`db.js` contains the code that instantiates the Prisma database client. This is the `db` object that was imported in the services file.

```js
// api/src/lib/db.js

import { PrismaClient } from '@prisma/client'
import { emitLogLevels, handlePrismaLogging } from '@redwoodjs/api/logger'
import { logger } from './logger'

export const db = new PrismaClient({
  log: emitLogLevels(['info', 'warn', 'error']),
})

handlePrismaLogging({
  db,
  logger,
  logLevels: ['info', 'warn', 'error'],
})
```

In the [next part](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-part-4-2m0g) we'll learn about [Cells](https://redwoodjs.com/tutorial/cells). We will set up our frontend to query data from our backend to render a list of our blog posts to the front page.