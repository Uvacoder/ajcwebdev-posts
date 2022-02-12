---
title: what is the status of mongoDB and prisma2?
description: A summary of the (no longer) current development on a MongoDB connector for Prisma.
date: 2020-09-17
tags:
  - mongodb
  - prisma
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/kbhq5eudjzx3c357edck.jpg
layout: layouts/post.njk
---

*Note: This post is no longer up to date. The MongoDB connector is undergoing [initial user testing as of April 2021](https://dev.to/ajcwebdev/can-i-use-mongodb-with-prisma-yet-50go).*

[Prisma](https://www.prisma.io/docs/) is a database toolkit that is amazing for relational databases and can be easily ported between Postgres, MySQL, MariaDB, SQLite, and Aurora. But right now Prisma does not currently support non-relational databases like MongoDB or DynamoDB.

I know many other developers are interested in bridging this gap and I was curious what work had already been done on the problem and how I could help contribute. I found a lengthy issue called [MongoDB support for Prisma 2](https://github.com/prisma/prisma/issues/1277) that was opened on January 6 and had 58 comments.

I read through all the comments and have done my best to summarize the situation for others.

# January - March

At the beginning of 2020, Nikolas Burk said that Prisma would be primarily focusing on [stabilizing their relational drivers](https://github.com/prisma/prisma/issues/1277#issuecomment-572065924) before working on other types of databases like MongoDB. The next month he also mentioned that MongoDB was working on a [Rust driver](https://github.com/prisma/prisma/issues/1277#issuecomment-582440399) that would provide the foundation for a MongoDB connector.

# April - May

Saghm Rossi from MongoDB provided an update on [April 23](https://github.com/prisma/prisma/issues/1277#issuecomment-618550580) that the process of rewriting the driver internals to be async was completed and that the beta would be released soon. The beta was released on [May 5](https://github.com/prisma/prisma/issues/1277#issuecomment-624290320).

Søren Bramer Schmidt from Prisma responded the [next day](https://github.com/prisma/prisma/issues/1277#issuecomment-624532966) to say that with the Rust driver being used in the Prisma Query Engine they were now working towards a MongoDB connector.

# June

Saghm announced the release of version 1.0.0 of the Rust driver on MongoDB's blog on [June 8](https://www.mongodb.com/blog/post/announcing-rust-driver-version-1). Nikolas Burk asked users the [next day](https://github.com/prisma/prisma/issues/1277#issuecomment-641502974) for more info about what they're building with MongoDB and some features they'd like to see.

Here's a few relevant comments:

[ppxb](https://github.com/prisma/prisma/issues/1277#issuecomment-641759200)

>*In my application I used Nest.js and MongoDB with `@nestjs/graphql`. It's a little bit difficult to organize the codes.*

[Nicklas Reincke](https://github.com/prisma/prisma/issues/1277#issuecomment-641785124)

>*Am doing something similar to @ppxb. I am using Nest.js as an API gateway (with Apollo Federation) to other microservices also written with Nest.js (with GraphQL endpoints).*
>
>*The gateway and the services are deployed to Google Cloud Run while the services can communicate over Cloud Pub/Sub topics.*
>
>*The microservices each should use Prisma 2 and have their own database, preferably MongoDB. But to work with Mongoose is a bit of a headache when it comes to structure the code correctly.*

[Adam Duro](https://github.com/prisma/prisma/issues/1277#issuecomment-642242839)

>*Having Prisma support Mongo aggregation pipeline in at least some fashion (either as a raw query type function, or with some kind of generic to be able to describe the expected output) as well as have support for geo queries, would be a huge win.*
>
>*There are many other "ORMs" that claim to have cross SQL/Mongo support often leave these features out, which drives people back to Mongoose, which is a pain to use in this new TypeScript heavy world we now live in.*

[Bradley Bain](https://github.com/prisma/prisma/issues/1277#issuecomment-648370387)

>*Having access to the aggregations pipeline through Prisma would be great (no need to get super fancy with it, even just a straightforward and barebones way to perform an aggregation with raw mongodb aggregation syntax would be miles ahead of existing solutions).*
>
>*Also another feature that would be great to have is multi-tenancy/clustering support.*
>
>*Mongoose didn't originally build with multi-tenancy in mind, so even though they technically support it now, it's not very pretty (having to explicitly switch the connection and then reload in the models after switching, since all mongoose schemas are scoped to a single connection). Something like:*

```
// prisma schema file

datasource mongdob {
  url      = env("DEFAULT_DB_URL") // or CLUSTER_URL?
  provider = "mongodb"
}
```
```
// main.js

const client1 = new PrismaClient()
const client2 = new PrismaClient('DIFFERENT_DB_URL')

const [ db1, db2 ] = await Promise.all([ client1.connect(), client2.connect() ])
```

# July - August

The entirety of comments on the issue in July and August were summed up by Alejandro Morales on [August 8](https://github.com/prisma/prisma/issues/1277#issuecomment-670975830):

>*I AM SAD I NEED PRISMA MONGO*

A lot of people appeared frustrated, but there were also many who came to Prisma's defense and encouraged commenters to think about maybe contributing themselves instead of just complaining about how lame their amazing free tool was.

Others pointed out that learning Rust is a high barrier to entry for this issue. The issue trails off from there without any further actionable information.

# Rusoto

Harry Manchanda suggested Rusoto as a possible solution for the DynamoDB connector issue [#1676](https://github.com/prisma/prisma/issues/1676#issuecomment-684558071).

>*I read this comment at mongoDB issue #1277 and seems like you would be looking for a Rust Driver for connecting to dynamoDB (similar to mongoDB)*
>
>*⏬ So would like to point to these links below ⏬*
>
>https://rusoto.org/index.html
>https://crates.io/crates/rusoto_dynamodb

I'll have to do more research into Rusoto, but on first glance it looks like it supports MongoDB equivalent things like DocumentDB since it's an AWS SDK but it doesn't support MongoDB proper.

# Questions

1. Does Prisma need anything else from MongoDB to implement a connector or is the MongoDB Rust driver good to go?
2. Is anyone at Prisma currently working on a MongoDB connector or accountable for tracking its progress?
3. If not what can contributors do to help Prisma out?

# Conclusion

Having gone through all 58 comments the only thing I came away certain of is that Saghm is a boss. On top of just shipping the Rust driver in the first place he's also created great docs [on GitHub](https://github.com/mongodb/mongo-rust-driver) and on [MongoDB's website](https://docs.mongodb.com/drivers/rust). He's also written [blog posts](https://www.mongodb.com/blog/post/announcing-rust-driver-version-1) to clearly communicate his progress.