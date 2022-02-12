---
title: a first look at keystoneJS
description: KeystoneJS is a CMS for developers that provides a GraphQL API & Management UI for content and data based on your schema.
date: 2021-08-25
tags:
  - keystonejs
  - prisma
  - graphql
  - cms
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t154djw4rn2nk4xs5o59.png
layout: layouts/post.njk
---

[KeystoneJS](https://keystonejs.com/) is a CMS for developers that provides a GraphQL API & Management UI for content and data based on your schema. In this tutorial we will use `create-keystone-app` to scaffold a basic blog dashboard with users. In a future part we will deploy this project.

## Create Keystone App

[`create-keystone-app`](https://github.com/keystonejs/create-keystone-app) is a CLI app for getting started with Keystone. It will generate a couple of files for you and install all the dependencies you need to run the Admin UI and start using the GraphQL API.

```bash
yarn create keystone-app
```

```
success Installed "create-keystone-app@4.0.12" with binaries:
      - create-keystone-app

ℹ️ You're about to generate a project using Keystone
Next packages. If you'd like to use Keystone 5, please
use `create-keystone-5-app` instead. Learn more about
the changes between Keystone 5 and Keystone Next on
our website.

https://next.keystonejs.com/guides/keystone-5-vs-keystone-next​
```

Give your project a name. Mine will be called `ajcwebdev-keystone`.

```
✔ What directory should create-keystone-app generate your app into? · ajcwebdev-keystone
```

You will need a database URL. If you want one fast you can use [Railway](https://railway.app/), [Supabase](https://supabase.io/), [Fly](https://fly.io/), or any of the other couple dozen "spin up Postgres in 30 seconds" services out there. I'll be using Railway.

You can follow the first half of [this guide](https://dev.to/ajcwebdev/a-first-look-at-postgraphile-with-railway-1k9d) to see how to spin up the database. The only important difference is when you copy your connection string from Railway, change the first part from `postgresql://` to `postgres://`.

Your connection string will look like this with a password instead of `xxxx`:

```
✔ What database url should we use? · postgres://postgres:xxxx@containers-us-west-3.railway.app:6787/railway
```

This will create the app and provide further instructions.

```
🎉  Keystone created a starter project in: ajcwebdev-keystone

To launch your app, run:

  - cd ajcwebdev-keystone
  - yarn dev

Next steps:

  - Edit ajcwebdev-keystone/keystone.ts to customize your app.
  - Open the Admin UI - ​http://localhost:3000​
  - Read the docs - ​https://next.keystonejs.com​
  - Star Keystone on GitHub - https://github.com/keystonejs/keystone​
```

## Develop Keystone App Locally

Run `yarn dev` to start your local development server.

```bash
cd ajcwebdev-keystone
yarn dev
```

This will run `keystone-next dev`.

```
$ keystone-next dev

✨ Starting Keystone
⭐️ Dev Server Ready on http://localhost:3000
✨ Generating GraphQL and Prisma schemas
✨ Your database is now in sync with your schema. Done in 3.17s
✨ Connecting to the database
✨ Generating Admin UI code
✨ Creating server
✨ Preparing GraphQL Server
✨ Preparing Admin UI Next.js app

info  - Using webpack 4. Reason: custom webpack configuration in next.config.js https://nextjs.org/docs/messages/webpack5
event - compiled successfully
👋 Admin UI and GraphQL API ready
```

Open [localhost:3000](http://localhost:3000).

![01-welcome-to-keystonejs-localhost](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0jq96e0axkl9t12800hj.png)

### Create a user

![02-create-first-user](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vrqf1v1ld3k0mvu4wf0c.png)

You are then asked if you want to sign up for the KeystoneJS newsletter. Bold move!!!

![03-sign-up-for-keystone-mailing-list](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ssogwju2fktos0x5hgz0.png)

After you decide whether to reward such brash behavior you will be taken to the Keystone dashboard where you can view your users and posts.

![04-keystone-dashboard](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m6ilf2nijrwa7xokwb8f.png)

You will see the user you initially created.

![05-keystone-dashboard-users](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ubli8yfm6l6dqta0mycc.png)

### Create a Post

You will see that there are no posts yet. What ever shall we do?

![06-keystone-dashboard-posts](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q2wp4vkyrynrh6wv82e9.png)

After clicking "Create Post" you will be able to create a post.

![07-keystone-dashboard-create-post-template](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/x392bth7vb710agr2x2c.png)

Write a post. Make it a masterpiece.

![08-a-new-post](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/om435cg30kg38oejah5g.png)

Look back at your database to make sure it didn't explode.

![09-post-in-railway](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g8j1q9un113ke4bguros.png)

### Query your GraphQL API

Open [localhost:3000/api/graphql](http://localhost:3000/api/graphql) and run the following `posts` query.

```graphql
query {
  posts {
    title
  }
}
```

![10-graphql-posts-query](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g6ze9l44gewfzo1h5nhy.png)

## Deploy Keystone App

Turns out deploying a Keystone app is kind of complicated unless you follow a separate guide called [How to embed Keystone + SQLite in a Next.js app](https://keystonejs.com/docs/walkthroughs/embedded-mode-with-sqlite-nextjs#bonus-deploy-to-vercel). If it's possible to deploy this thing to Vercel, then in theory it should be possible to [deploy it to Netlify](https://www.netlify.com/blog/2020/11/30/how-to-deploy-next.js-sites-to-netlify/). Tune in next time for that!

You can also view all the code for this project [on my GitHub](https://github.com/ajcwebdev/ajcwebdev-keystone) but if you clone it then it's not actually gonna work if you try and spin it up cause by default `create-keystone-app` hardcodes your `DATABASE_URL` and figuring out how to make it work without doing that is kind of complicated.