---
title: what databases will prisma support next?
description: There are many databases.
date: 2021-02-13
tags:
  - prisma
  - orm
  - database
  - mongodb
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/g706n112zckhcnz6ymaw.png
layout: layouts/post.njk
---

## Front runners

* MongoDB
* DynamoDB
* Neo4j
* Aurora Serverless Data API
* FaunaDB

## Moderate interest

* CockroachDB
* CockroachDB in Migrate
* Cassandra/ScyllaDB
* In-memory SQLite

## Low interest

* RethinkDB
* ArangoDB
* Google Cloud Spanner
* TimescaleDB
* BedrockDB

## Least interest

* Azure CosmosDB
* Notion as database
* MemSQL

## Front runners

### [MongoDB support for Prisma 2](https://github.com/prisma/prisma/issues/1277)
229 👍 - 62 comments

>We want to have a design proposal to share by `2.16.0`. The implementation will take awhile longer.

>Sorry to get your hopes up, but please know that we've started working on this and implementing Mongo is a top priority for us!

### [DynamoDB connector](https://github.com/prisma/prisma/issues/1676)
153 👍 - 8 Comments

>AWS just announced [PantiSQL over dynamodb](https://aws.amazon.com/about-aws/whats-new/2020/11/you-now-can-use-a-sql-compatible-query-language-to-query-insert-update-and-delete-table-data-in-amazon-dynamodb/). I guess this makes DynamoDB support easier.

### [Neo4j support](https://github.com/prisma/prisma/issues/1575)
82 👍 - 31 Comments

>Now I've come to understand that it's not a mutually exclusive GRANDStack - or - Nexus Framework decision. Fundamentally each project has a distinct area of focus.
>
>That said they certainly overlap. For instance both handle schema definition/construction. However Nexus achieves this via code. And the `neo4j-graphql-js` graphql middleware is bootstrapped via GraphQL SDL. Both result in a graphql endpoint that I can generate TypeScript types from for nearly compatible typesafe usage on my frontend.
>
>Anyways this is just an intro. I suppose it's also worth noting that I was running a Kubernetes cluster for my Nexus Framework endpoint and Prisma Postgres instance, and now I'm literally running a single google cloud run serverless service which hosts the neo4j-graphql-js middleware. The Neo4j instance I outsourced to their own Neo4j managed "Aura" (provisioned through GCP)
>
>I sense intuitively that this integration should/will happen. Graph-native databases are fundamental staple of modern software bag-o-tricks, meaningful Nexus-Prisma-GRANDstack integration is a must. As I get deeper into GRANDstack I'd like to really seriously ask the question what this Neo4j connector could look like. Especially the "WHY?" And then the "HOW?" and ultimately the concrete "WHAT?"

### [Add support for Data API (AWS Aurora Serverless)](https://github.com/prisma/prisma/issues/1964)
62 👍 - 13 Comments

>>*There is Data API client which is also used by [TypeORM](https://github.com/ArsenyYankovsky/typeorm-aurora-data-api-driver)*
>
>>*The Data API seems to be plagued with some issues related to casting types like json, dates etc which is handled by the [above client](https://github.com/jeremydaly/data-api-client/pull/57)*
>
>>*It seems like to be able to use this Prisma would have to explicitly provide the types to the Data API client.*

>So, I asked a prisma developer this directly, and he took the question to the product team. I specified that I had a project launching in July, and if the integration would be done by summer, we would build with prisma. They said that currently they have no plans to prioritize this, and it would not likely be done this year. Obviously the more interest the more likely they are to prioritize, but right now I don’t think it is really on the radar.

### [FaunaDB support](https://github.com/prisma/prisma/issues/2445)
47 👍 - 4 Comments

>Fauna currently offers multiple different ways to access your data, most notably (1) Using FQL directly or via a language SDK (2) Using their GraphQL API.
>
>In collaboration with Fauna, we've started working on a Rust driver for Fauna which will eventually enable Prisma to add support for Fauna based on (1). Fauna's GraphQL API (2) is a very high-level abstraction and comes with a notable set of tradeoffs that doesn't make it a great fit as a Prisma connector since it hides a lot of Fauna's critical lower-level controls.

## Moderate interest

### [Support for CockroachDB](https://github.com/prisma/prisma/issues/2523)
16 👍 - 9 Comments

### [Support CockroachDB in Migrate](https://github.com/prisma/prisma/issues/4702)
8 👍 - 16 Comments

>Prisma Client, which is the part which queries the data, already works with Cockroach with the Postgres mode.
>
>The part which doesn't work right now is Prisma Migrate (see prisma/migrate#426). Migrate queries the information schema to figure out the state of the database and it is using some postgres functions which are not supported by cockroach it seems.
>
>Right now there is not really an API for creating a custom Prisma connector for the public. So that way is not really feasible right now.
>
>We are also overhauling migrate, making quite a lot of changes to it so that we can take it out from the experimental state so this will not be a priority for us until then.

### [Support for Cassandra/ScyllaDB](https://github.com/prisma/prisma/issues/2879)
18 👍 - 2 Comments

>>*I am working on a large enterprise-embeddd application where Scylla already has a presence. This feature would really help promote Prisma in that application.*

>Is there any comment on this from the Prisma team? For our project, we've had to move from Postgres to Cassandra, and would like to continue using Prisma.

### [Support in-memory SQLite connector](https://github.com/prisma/prisma/issues/732)
11 👍 - 1 Comment

>I think we can treat this in the same way that we treat the normal SQLite mode. I think in-memory SQLite doesn't make sense in migrate, since it's reset every run.

## Low interest

### [RethinkDB support](https://github.com/prisma/prisma/issues/2524)
6 👍 - 5 Comments

>I would like this too, this could be an easy way to get realtime data

### [ArangoDB support](https://github.com/prisma/prisma/issues/2535)
7 👍 - 1 Comment

>>*Its like redis + MongoDB + Neo4j all in one engine.*

>And don't forget Search. It's one of the best multi-model DB available to date, Document, Key-Value, Graph and Search. That makes ArangoDB.com truly awesome. Support for it would be much appreciated.
>
>Their official [Typescript client](https://github.com/arangodb/arangojs) is well maintained and quite powerful, it would be very useful in an eventual integration with Prisma.

### [Support Google Cloud Spanner](https://github.com/prisma/prisma/issues/717)
7 👍 - 0 Comments

>Fully managed relational database with unlimited scale, strong consistency, and up to 99.999% availability.

### [Support (PostgreSQL) TimescaleDB](https://github.com/prisma/prisma/issues/3228)
4 👍 - 3 Comments

>It's possible to use TimescaleDB and prisma together with some manual work and using raw queries.
>* This was already possible with prisma1: [prisma/prisma1#1672 (comment)](https://github.com/prisma/prisma1/issues/1672#issuecomment-511410796)
>* This has been made easier with the implementation of composite keys: [#1389](https://github.com/prisma/prisma/issues/1389)
>
>However it is still needed to manually install the extension, then manually convert tables into hypertables using raw sql, and some TimescaleDB features are implemented using Postgres Functions.

### [Support BedrockDB](https://github.com/prisma/prisma/issues/2911)
0 👍 - 14 Comments

>Bedrock is a WAN-replicated, Blockchain-based data foundation for global-scale applications. Bedrock was built by Expensify, and is a networking and distributed transaction layer built atop SQLite. Bedrock supports the MySQL protocol.

## Least interest

### [Azure Cosmos DB support](https://github.com/prisma/prisma/issues/2713)
1 👍 - 5 Comments

>>*This is most likely to get solved via #1277 as Cosmos has support for monogodb wire protocol.*

>Cosmos also has a SQL flavor that is more widely supported in the Azure ecosystem and I would be very interested to see support for that as well.
>
>Of the teams I know using it, it seems split down the middle between the SQL and Mongo flavor. It does support other models as well like a graph db but not sure how popular those are. The SQL one is more well supported in the Azure ecosystem but the Mongo flavor I can see people choosing to be Mongo compatible.
>
>My vote would be SQL. This is relatively moot for our team as we are currently converting from Cosmos to Postgres and switching our custom ORM over to Prisma.

### [Support Notion as database](https://github.com/prisma/prisma/issues/665)
1 👍 - 0 Comments

>At @prisma we're using Notion heavily as our internal knowledge base and are running many "processes" in Notion (think internal apps). With Notion soon releasing an API, I think it could be really interesting to explore a Notion connector for Prisma to make it easier to use Notion as a database for some (internal) applications.

### [Support MemSQL](https://github.com/prisma/prisma/issues/2935)
0 👍 - 1 Comment

>SingleStore (formerly known as MemSQL) is a distributed, relational, SQL database management system (RDBMS) that features ANSI SQL support and is known for speed in data ingest, transaction processing, and query processing.