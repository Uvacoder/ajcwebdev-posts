---
title: the bison stack
description: Bison is a starter repository created out of real-world apps at Echobind. It represents the team's "Greenfield Web Stack" used when creating web apps for clients.
date: 2021-03-19
tags:
  - bison
  - graphql
  - prisma
  - chakra
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/le1nafm65vasmli5ssog.png
layout: layouts/post.njk
---

Bison is a starter repository created out of real-world apps at [Echobind](https://echobind.com/). It represents the team's "Greenfield Web Stack" used when creating web apps for clients. This article examines some of the different libraries that make up the Bison framework.

## Nexus for building GraphQL API's

![nexus-logo](https://dev-to-uploads.s3.amazonaws.com/i/d1hycuz9fuetopddyc8j.jpg)

[Nexus](https://nexusjs.org/) was designed with [TypeScript](https://www.typescriptlang.org/) intellisense in mind with full auto-generated type coverage out of the box.
* TypeScript generics
* Conditional types
* Type merging

It builds upon the primitives of `graphql-js` and aims to combine:
* Simplicity of SDL schema-first approach like [graphql-tools](https://www.apollographql.com/docs/graphql-tools/generate-schema.html)
* Power of a full language runtime like [graphql-js](https://github.com/graphql/graphql-js)

## GraphQL Codegen for generating types

![graphql-code-generator](https://dev-to-uploads.s3.amazonaws.com/i/ib54c5jtq49wmm7g2xoq.png)

[GraphQL Code Generator](https://graphql-code-generator.com/) is a CLI tool that can generate TypeScript typings out of a GraphQL schema. By analyzing the schema and parsing it, GraphQL Code Generator can output code at a wide variety of formats. Output can be based on:
* Pre-defined plugins
* Custom user-defined ones

```graphql
type Post {
  id: Int!
  title: String!
  author: Author!
}

type Query {
  posts: [Post]
}

schema {
  query: Query
}
```

This schema will allow GraphQL Code Generator to generate the following TypeScript typings with the `typescript` plugin:

```typescript
export type Maybe<T> = T | null;

export type Scalars = {
  ID: string,
  String: string,
  Boolean: boolean,
  Int: number,
  Float: number,
};

export type Post = {
  __typename?: 'Post',
  id: Scalars['Int'],
  title: Scalars['String'],
  author: Author,
};

export type Query = {
  __typename?: 'Query',
  posts?: Maybe<Array<Maybe<Post>>>,
};
```

## Prisma for databases

![prisma-logo](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/v84g8g29pltcbb1efx3p.png)

[Prisma](https://www.prisma.io/) is an [open source](https://github.com/prisma/prisma) next-generation ORM consisting of the following parts:

- **Prisma Client**: Auto-generated and type-safe query builder for Node.js & TypeScript
- **Prisma Migrate**: Declarative data modeling & migration system
- **Prisma Studio**: GUI to view and edit data in your database

### The Prisma schema

The Prisma schema defines the application models with a data modeling language. It provides the connection to a database and defines a generator. You configure three things:

**Data source** specifies database connection via an environment variable

```
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

**Generator** indicates you want to generate Prisma Client

```
generator client {
  provider = "prisma-client-js"
}
```

**Data model** defines application models

```
model Post {
  id        Int     @id @default(autoincrement())
  title     String
  content   String?
  published Boolean @default(false)
  author    User?   @relation(fields:  [authorId], references: [id])
  authorId  Int?
}

model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}
```

### The Prisma data model

The data model is a collection of models with two major functions:

- Represent a table in the underlying database
- Provide the foundation for the queries in the Prisma Client API

Once the data model is defined, you can generate Prisma Client which will expose CRUD and more queries for the defined models. There are two ways to do this:

- Generate the data model from introspecting a database
- Manually write the data model and map it to the database with Prisma Migrate

## Chakra for styling

![chakra-ui](https://dev-to-uploads.s3.amazonaws.com/i/nthyelkvm2dt4lxrdu35.png)

[Chakra UI](https://chakra-ui.com/) is established on principles that keep its components fairly consistent. Its goal is to design simple, composable components that cater to real-life UI design problems.

* **Style Props** - All component styles can be overriden or extended via style props to reduce the use of `css` prop or `styled()`. Compose new components from `Box`.
* **Simplicity** - Strive to keep the component API fairly simple and show real world scenarios of using the component.
* **Composition** - Break down components into smaller parts with minimal props composed together to keep complexity low.
* **Accessibility** - When creating a component, keep accessibility top of mind including keyboard navigation, focus management, color contrast, voice over,  and correct `aria-*` attributes.
* **Dark Mode** - Make components dark mode compatible.
* **Naming Props** - Generally, ensure a prop name is indicative of what it does.