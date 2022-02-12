---
title: can I use mongoDB with prisma yet?
description: MongoDB is a database. It does stuff with data and then puts it in a base. Prisma now lets you do that without writing Mongo stuff.
date: 2021-04-21
tags:
  - prisma
  - mongodb
  - atlas
  - orm
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/c53lwyayptdahtrf5qul.png
layout: layouts/post.njk
---

*Warning: The MongoDB connector for Prisma 2 is not ready for production and should not be used by anyone under any circumstances. It is in [Preview mode](https://github.com/prisma/prisma/releases/tag/2.27.0) as of v2.27.0.*

MongoDB is a database. It does stuff with data and then puts it in a base¹. But it doesn't do this the way a normal database does. That's because it uses documents instead of relations. At one point in time it was called NoSQL.

But don't worry about all that.

The whole reason you use Prisma in the first place is because you don't want to write SQL *or* whatever crazy query language they came up with at 10gen back in the day. You just want to write JavaScript like God intended.

And you're willing to write [lots and lots and lots and lots and lots and lots and lots of comments](https://dev.to/ajcwebdev/what-s-the-status-of-mongodb-and-prisma2-h20) on GitHub letting the world know this.

Well it's your lucky day...

## MongoDB Atlas

The first thing you'll need is a Mongo database (shocking). The easiest ([not cheapest](https://www.loginradius.com/blog/async/self-hosted-mongo/)) way to get a Mongo database is with [MongoDB Atlas](https://www.mongodb.com/cloud/atlas2). I apologize for the [click-ops](https://m.subbu.org/clickops-3cf0e5bc5ecf). After creating an account you will be asked to name your organization and project.

![01-organization-and-project-names](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fuk8pwjywtxpx46h2t32.png)

You will then be asked for your preferred language, a question that no one has ever argued about.

![02-preferred-language](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lm1h46jflnaeazuq62jg.png)

You will then be asked if you want a plan that costs money or a plan that doesn't cost money. I'm sure this will be a very hard decision to make.

![03-shared-clusters](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/drbvki17umjzaxr4nju7.png)

Pick a region close to you cause ain't nobody got time for the speed of light.

![04-cloud-provider-and-region](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mpul956uecxwvam90zu1.png)

Again, you can pick the one that's free or one that's not free.

![05-cluster-tier](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wjyztdbs7hxp5k9mvvaw.png)

Give your cluster a name that suitably explains just how thoroughly this project has been nailed.

![06-cluster-name](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nb3ojr7p3mqqch0tb02b.png)

Click "Create Cluster" to create a cluster. You will then be taken to your dashboard.

![07-atlas-dashboard](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pwcp1zl19qk29m1mvwqo.png)

Take a few minutes to contemplate your mortality. Once the existential dread starts to fade away you should have a cluster up and running.

![08-clusters](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/30zotn3hpybajcnfku7c.png)

If you want to do anything useful with this database like put data in the base then you need to connect to it. Click connect to connect.

![09-connect](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2gaf7csw3uf7b8zgia3h.png)

Give your database a username and a password that is totally secret and not published in a blog post on the internet. Click "Create Database User" to create a database user and then click "Add your current IP address" to add your current IP address.

![10-choose-a-connection-method](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xfnne902uqr92zn960ry.png)

Click "Choose a connection method" to choose a connection method.

![11-connect-your-application](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/p4eo0ti78q6o5vblre5g.png)

Click "Connect your application" to connect your application.

![12-environment-variable](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0vifnwasl1i0ahrohnj6.png)

## Prisma

Now that the boring stuff is out of the way we can get to the ORMing.

### Create a new blog

```bash
mkdir ajcwebdev-prismongo
cd ajcwebdev-prismongo
```

### Initialize package.json

```bash
npm init -y
```

### Install development dependencies

```bash
npm install -D prisma typescript ts-node @types/node
```

### Initialize Prisma Schema

```bash
npx prisma init
```

If you'd like to try MongoDB on an existing Prisma project, you'll need to make sure you're running Prisma 2.21.2 or above. Just remember, no one under any circumstances should be using this. Not even you!!!

### Prisma Schema

Your `schema.prisma` file will contain a `Post`.

```
datasource db {
  provider = "mongodb"
  url      = env("DATABASE_URL")
}

generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["mongodb"]
}

model Post {
  id    String @id @default(dbgenerated()) @map("_id") @db.ObjectId
  slug  String @unique
  title String
  body  String
}
```

What's that? You don't need a blog? I don't see what that has to do with anything.

### Generate Prisma Client

Now run that one command that does all the stuff. Don't worry about it too much, it just does the stuff.

```bash
npx prisma generate
```

### Use Prisma Client

Create a script to test that we can read and write data to our base in an `index.ts` file.

```bash
touch index.ts
```

```ts
import { PrismaClient } from "@prisma/client"
const prisma = new PrismaClient()

async function main() {
  await prisma.$connect()

  await prisma.post.create({
    data: {
      title: "If you really want something, just complain loudly on GitHub",
      slug: "this-is-a-terrible-lesson-to-teach",
      body: "That's a joke please don't do this.",
    },
  })

  await prisma.post.create({
    data: {
      title: "Second post",
      slug: "post-two",
      body: "This is the second post.",
    },
  })

  const posts = await prisma.post.findMany()

  console.dir(posts, { depth: Infinity })
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect())
```

### Environment Variable

Inside `.env` include your `DATABASE_URL`.

```
DATABASE_URL=mongodb+srv://ajcwebdev:dont-steal-my-db-ill-kill-you@nailed-it.5mngs.mongodb.net/myFirstDatabase?retryWrites=true&w=majority
```

### Hold your breath and run the script

```bash
npx ts-node index.ts
```

If you followed along correctly you should get the following output:

```
[
  {
    id: '6080d8df0011018100a0e674',
    slug: 'this-is-a-terrible-lesson-to-teach',
    title: 'If you really want something, just complain loudly on GitHub',
    body: "That's a joke please don't do this."
  },
  {
    id: '6080d8e00011018100a0e675',
    slug: 'post-two',
    title: 'Second post',
    body: 'This is the second post.'
  }
]
```

![13-collections](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dsdcizbjanx9l99e3hd8.png)

## Next Steps

* If you want to report an issue or file a feature request, please [open an issue on GitHub](https://github.com/prisma/prisma/issues/new/choose).
* Since this is a preview feature that should not be used by anyone ever you may want to check out the [known limitations](https://www.notion.so/Getting-Started-with-MongoDB-and-Prisma-58b41c67eb554787944c600d2239801f#5fa34972c92d4115a1c0bf687a45742e).
* You can [join the the Early Access Program](https://prisma103696.typeform.com/to/FriDuIeM) or [schedule a call to get in touch with the team](https://www.notion.so/Getting-Started-with-MongoDB-and-Prisma-58b41c67eb554787944c600d2239801f#97d8db32b8964687b72129f44e891c44).

[1] - Not an accurate definition of a database.