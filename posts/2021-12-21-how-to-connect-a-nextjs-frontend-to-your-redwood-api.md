---
title: how to connect a next.js frontend to your redwood API
description: This example contains two separate repositories, one for the Redwood API and another for the Next frontend.
date: 2021-12-21
tags:
  - redwoodjs
  - nextjs
  - vercel
  - railway
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pras1bcaa94smjr38rk6.jpg
layout: layouts/post.njk
---

This example contains two separate repositories, one for the Redwood API and another for the Next frontend. Deploying them both from the same repo is certainly possible but may pose a challenge with getting the correct build commands and publish directories, so I've punted on that for now. If anyone is curious to see an example with a monorepo containing both apps just let me know and I'll see what I can do.

## Outline

* [Create Redwood App](#create-redwood-app)
  * [Prisma Schema](#prisma-schema)
  * [Set database connection string](#set-database-connection-string)
  * [Apply migration to the database and scaffold admin dashboard](#apply-migration-to-the-database-and-scaffold-admin-dashboard)
  * [Create GitHub repository](#create-github-repository)
  * [Test your endpoint](#test-your-endpoint)
* [Create Next App](#create-next-app)
  * [Add posts query to Next frontend](#add-posts-query-to-next-frontend)

## Create Redwood App

We will be deploying this to Vercel.

```bash
yarn create redwood-app redwood-next
cd redwood-next
yarn rw setup deploy vercel
```

### Prisma Schema

Our schema has the same `Post` model used in the [Redwood tutorial](https://learn.redwoodjs.com/docs/tutorial/getting-dynamic#creating-the-database-schema).

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

### Set database connection string

Include `DATABASE_URL` in `.env`. See [this post](https://community.redwoodjs.com/t/setup-database-with-railway-cli/2025) for instructions on quickly setting up a remote database on Railway.

```
DATABASE_URL=postgresql://postgres:password@containers-us-west-10.railway.app:5513/railway
```

### Apply migration to the database and scaffold admin dashboard

```bash
yarn rw prisma migrate dev --name posts
yarn rw g scaffold post
```

Open [localhost:8910/posts/new](http://localhost:8910/posts/new) and create a post.

![01-create-new-blog-post-at-posts-new-route](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/iicib99r55zd4i4slb1j.png)

### Create GitHub repository

```bash
git init
git add .
git commit -m "Nextification"
gh repo create redwood-next --public
git remote add origin https://github.com/ajcwebdev/redwood-next.git
git push -u origin main
```

Select the Redwood framework preset and include your database connection string. Make sure to include `?connection_limit=1` at the end of the connection string.

![02-configure-project-on-vercel](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kfqvsuihbfsg0efth1uv.png)

### Test your endpoint

Our API is now deployed. Hit [https://redwood-next.vercel.app/api/graphql](https://redwood-next.vercel.app/api/graphql) with your favorite API tool or curl.

```bash
curl \
  --request POST \
  --header 'content-type: application/json' \
  --url 'https://redwood-next.vercel.app/api/graphql' \
  --data '{"query":"{ redwood { version currentUser prismaVersion } }"}'
```

```graphql
{
  redwood {
    version
    currentUser
    prismaVersion
  }
}
```

```json
{
  "data": {
    "redwood": {
      "version": "0.39.3",
      "currentUser": null,
      "prismaVersion": "3.5.0"
    }
  }
}
```

```bash
curl \
  --request POST \
  --header 'content-type: application/json' \
  --url 'https://redwood-next.vercel.app/api/graphql' \
  --data '{"query":"{ posts { id title body createdAt } }"}'
```

```graphql
{
  posts {
    id
    title
    body
    createdAt
  }
}
```

```json
{
  "data": {
    "posts": [
      {
        "id": 1,
        "title": "Redwood+Next",
        "body": "The next best thing?",
        "createdAt": "2021-12-09T19:55:33.292Z"
      }
    ]
  }
}
```

## Create Next App

Give your project a name, `cd` into it, and install Apollo Client.

```bash
yarn create next-app redwood-next-frontend
cd redwood-next-frontend
yarn add @apollo/client graphql
```

Start your development server.

```bash
yarn dev
```

Open [localhost:3000/](http://localhost:3000/).

![03-create-next-app-boilerplate](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/77i5k3yjrb2yreivwdey.png)

### Add posts query to Next frontend

Open `pages/index.js` and include the following code with your endpoint.

```jsx
// pages/index.js

import Head from 'next/head'
import { ApolloClient, InMemoryCache, gql } from '@apollo/client'
import styles from '../styles/Home.module.css'

export default function Home({ posts }) {
  return (
    <div className={styles.container}>
      <Head>
        <title>Redwood+Next</title>
        <link rel="icon" href="/favicon.ico" />
      </Head>

      <main>
        <h1>Redwood+Next ▲</h1>

        <div>
          {posts.map(post => {
            return (
              <ul key={post.id}>
                <li>
                  <h3>{post.title}</h3>
                  <p>{post.body}</p>
                </li>
              </ul>
            )
          })}
        </div>
      </main>
    </div>
  )
}

export async function getStaticProps() {
  const client = new ApolloClient({
    uri: 'https://redwood-next.vercel.app/api/graphql',
    cache: new InMemoryCache()
  })

  const { data } = await client.query({
    query: gql`
      query GetPosts {
        posts {
          id
          title
          body
        }
      }
    `
  })

  return {
    props: {
      posts: data.posts
    }
  }
}
```

![04-next-app-querying-redwood-api](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ngv5pi3qk3v28nxxck3c.png)

Follow the previous steps to create a GitHub repository and deploy it on Vercel. Visit [redwood-next-frontend.vercel.app](https://redwood-next-frontend.vercel.app) to see your project.

![05-next-app-querying-redwood-api-deployed-on-vercel](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/e7o70dn5xzc2wlxky0fy.png)