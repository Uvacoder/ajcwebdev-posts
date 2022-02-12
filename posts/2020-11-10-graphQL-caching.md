---
title: graphQL caching
description: The GraphQL specification aims to support a broad range of use cases. Caching has been considered out-of-scope for the spec itself since it wants to be as general as possible.
date: 2020-11-10
tags:
  - graphql
  - apollo
  - urql
  - caching
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/sabqxqayf0078eqkp2mm.png
layout: layouts/post.njk
---

In the words of the immortal Ken Wheeler:

> *GraphQL is kind of like the s\*\*\*. Actually, it's absolutely the s\*\*\*.*

I tend to agree with this sentiment but that doesn't mean I think GraphQL is perfect. One of the most persistent challenges that has faced GraphQL since its introduction 5 years ago is client side caching.

## Does the GraphQL Specification Address Caching?

The GraphQL specification aims to support a broad range of use cases. Caching has been considered out-of-scope for the spec itself since it wants to be as general as possible. Out of the roughly 30,000 words contained in the current working draft the word cache appears exactly once in section [3.5.5 on ID's](https://spec.graphql.org/draft/#sec-ID):

>*The ID scalar type represents a unique identifier, often used to refetch an object or as the key for a cache.*

In this article I'll try to answer a few high level questions around GraphQL caching including:

* Why does GraphQL struggle with client side caching?
* Why does this matter in GraphQL more so than REST?
* What solutions do we currently have for this problem and what potential solutions are people working on?

While the spec leaves caching to the imagination there is the next best thing to the spec, [GraphQL.org](https://graphql.org/). They have a page dedicated to [explaining caching with GraphQL](https://graphql.org/learn/caching/) that I'll summarize after a quick primer on HTTP caching.

## HTTP Caching

Before talking about strategies for GraphQL caching, it's useful to understand HTTP caching. Freshness and validation are different ways of thinking about how to control client and gateway caches.

### Client side and Gateway caches

* **Client side caches** (browser caches) use HTTP caching to avoid refetching data that is still fresh
* **Gateway caches** are deployed along with a server to check if the information is still up to date in the cache to avoid extra requests

### Freshness and Validation

* **Freshness** lets the server transmit the time a resource should be considered fresh (through `Cache-Control` and `Expires` headers) and works well for data that doesn’t change often
* **Validation** is a way for clients to avoid refetching data when they’re not sure if the data is still fresh or not (through `Last-Modified` and `Etags`)

## GraphQL Caching

Clients can use HTTP caching to easily avoid refetching resources in an endpoint-based API. The URL is a **globally unique identifier**.

It can be leveraged by the client to build a cache by identifying when two resources are the same. Only the combination of those two parameters will run a particular procedure on the server. Previous responses to GET requests can be cached and future requests can be routed through the cache. A historical response can be returned if possible.

### Globally Unique IDs

Since GraphQL lacks a URL-like primitive the API usually exposes a globally unique identifier for clients to use. One possible pattern for this is reserving a field (`id`).

```graphql
{
  starship(id:"3003") {
    id
    name
  }
  droid(id:"2001") {
    id
    name
    friends {
      id
      name
    }
  }
}
```

The `id` field provides a globally unique key. This is simple if the backend uses a UUID. But a globally unique identifier will need to be provided by the GraphQL layer if it is not provided by the backend. In simple cases this involves appending the name of the type to the ID and using that as the identifier.

### Compatibility with existing APIs

How will a client using the GraphQL API work with existing APIs? It will be tricky if our existing API accepts a type-specific `id` while our GraphQL API uses globally unique identifiers. The GraphQL API can expose the previous API in a separate field and GraphQL clients can rely on a consistent mechanism for getting a globally unique identifier.

### Alternatives

The client needs to derive a globally unique identifier for their caching. Having the server derive that `id` simplifies the client but the client can also derive the identifier. This can require combining the type of the object (queried with `__typename`) with some type-unique identifier.

Dhaivat Pandya wrote and spoke extensively [back in 2016](https://www.apollographql.com/blog/the-concepts-of-graphql-bc68bd819be3/) about how Apollo was tackling caching. We'll talk more about Apollo's cache later, but here is a high level summary of Dhaivat Pandya's thoughts.

>*__Query result trees__ represent a way to get trees out of your app data graph. Apollo Client applies two assumptions to cache query result trees.*
>* *__Same path, same object__ — Same query path usually leads to the same piece of information*
>* *__Object identifiers when the path isn't enough__ — Two results given for the same object identifier represent the same node/piece of information*
>
>*Apollo Client will update the query with a new result if any cache node involved in a query result tree is updated.*

## Apollo Client

Apollo Client stores the results of its GraphQL queries in a normalized, in-memory cache for responding sparingly to future queries for the same data.

Normalization constructs a partial copy of your data graph on your client. The format is optimized for reading and updating the graph as your application changes state. You can configure the cache's behavior for other use cases:

* Specify custom primary key fields
* Customize the storage and retrieval of individual fields
* Customize the interpretation of field arguments
* Define supertype-subtype relationships for fragment matching
* Define patterns for pagination
* Manage client-side local state

### InMemoryCache

```ts
import { InMemoryCache, ApolloClient } from '@apollo/client'

const client = new ApolloClient({
  cache: new InMemoryCache(options)
})
```

### Data normalization

`InMemoryCache` has an internal data store for normalizing query response objects before the objects are saved:

1. Cache [generates a unique ID](https://www.apollographql.com/docs/react/caching/cache-configuration/#generating-unique-identifiers) for every identifiable object in the response
2. Cache stores objects by ID in a flat lookup table
3. Whenever an incoming object is stored with a ***duplicate*** ID the fields of those objects are ***merged***
4. If incoming and existing object share fields, cached values for those fields are ***overwritten*** by incoming object
5. Fields in ***only*** existing or ***only*** incoming object are preserved

`InMemoryCache` can exclude normalization for objects of a certain type for metrics and other transient data that's identified by a timestamp and never receives updates. Objects that are not normalized are embedded within their parent object in the cache. These objects can be accessed via their parent but not directly.

### readQuery

[`readQuery`](https://www.apollographql.com/docs/react/caching/cache-interaction/#readquery) enables you to run a GraphQL query directly on your cache. If the cache contains all necessary data it returns a data object in the shape of the query, otherwise it throws an error. It will never attempt to fetch data from a remote server.

#### Pass `readQuery` a GraphQL query string

```js
const { todo } = client.readQuery({
  query: gql`
    query ReadTodo {
      todo(id: 5) {
        id
        text
        completed
      }
    }
  `,
})
```

#### Provide GraphQL variables to `readQuery`

```js
const { todo } = client.readQuery({
  query: gql`
    query ReadTodo($id: Int!) {
      todo(id: $id) {
        id
        text
        completed
      }
    }
  `,

  variables: {
    id: 5,
  },
})
```

### readFragment

[`readFragment`](https://www.apollographql.com/docs/react/caching/cache-interaction/#readfragment) enables you to read data from any normalized cache object that was stored as part of any query result. Calls do not need to conform to the structure of one of your data graph's supported queries like with `readQuery`.

#### Fetch a particular item from a to-do list

```js
const todo = client.readFragment({
  id: 'Todo:5',

  fragment: gql`
    fragment MyTodo on Todo {
      id
      text
      completed
    }
  `,
})
```

### writeQuery, writeFragment

You can also write arbitrary data to the cache with [`writeQuery` and `writeFragment`](https://www.apollographql.com/docs/react/caching/cache-interaction/#writequery-and-writefragment). All subscribers to the cache (including all active queries) see this change and update the UI accordingly.

#### Same signature as `read` counterparts except with additional `data` variable

```js
client.writeFragment({
  id: '5',

  fragment: gql`
    fragment MyTodo on Todo {
      completed
    }
  `,

  data: {
    completed: true,
  },
})
```

### Combining reads and writes

`readQuery` and `writeQuery` can be combined to fetch currently cached data and make selective modifications. Create a new `Todo` item that is cached without sending it to the remote server.

```js
const query = gql`
  query MyTodoAppQuery {
    todos {
      id
      text
      completed
    }
  }
`

const data = client.readQuery({ query })

const myNewTodo = {
  id: '6',
  text: 'Start using Apollo Client.',
  completed: false,
  __typename: 'Todo',
}

client.writeQuery({
  query,
  data: {
    todos: [...data.todos, myNewTodo],
  },
})
```

### cache.modify

[`cache.modify`](https://www.apollographql.com/docs/react/caching/cache-interaction/#cachemodify) of `InMemoryCache` enables you to directly modify the values of individual cached fields, or even delete fields entirely. This is an escape hatch you want to avoid. Although, as we'll see at the end of the article, some people think we should only have an escape hatch.

## urql

Urql also modifies `__typename` like Apollo but it caches at the query level. It keeps track of the types returned for each query. If data modifications are performed on a type, the cache is cleared for all queries that hold that type.

```graphql
mutation {
  updateTask(id: 2, assignedTo: "Bob") {
    Task {
      id
      assignedTo
    }
  }
}
```

The metadata returned will show that a task was modified, and so all queries holding task results will be invalidated, and run against the network the next time they’re needed.

But urql has no way of knowing what the query holds. This means that if you run a mutation creating a task that’s assigned to Fred instead of Bob, the mutation result will not be able to indicate that this particular query needs to be cleared.

## micro-graphql-react

According to [Adam Rackis](https://adamrackis.dev/graphql-caching-and-micro/), Urql's problem can actually be solved with a build step that manually introspects the entire GraphQL endpoint. Adam couldn't get other GraphQL client cache's to behave the way he wanted.

He decided to build a GraphQL client with low-level control called `micro-graphql-react`. It provides the developer with building blocks for managing cache instead of adding metadata to queries to form a normalized, automatically-managed cache.

### Import client for global subscriptions to keep cache correct

```js
graphqlClient.subscribeMutation([
  {
    when: /updateX/,
    run: (op, res) => syncUpdates(Y, res.update, "allX", "X")
  },
  {
    when: /deleteX/,
    run: (op, r) => syncDeletes(Y, r.delete, "allX", "X")
  }
])

let { loading, loaded, data } = useQuery(
  buildQuery(
    Y,
    {
      publicUserId,
      userId
    },
    {
      onMutation: {
        when: /(update|delete)X/,
        run: ({ refresh }) => refresh()
      }
    }
  )
)
```

### Sync changes when relevant mutations happen

```js
let { loading, loaded, data } = useQuery(
  buildQuery(
    AllSubjectsQuery,
    {
      publicUserId,
      userId
    },
    {
      onMutation: {
        when: /(update|delete)Subject/,
        run: ({ refresh }) => refresh()
      }
    }
  )
)
```

### Cache Resetting

`micro-graphql-react` was written with the assumption that managing cache invalidation should not be a framework concern. It should be easy to manage yourself with a set of primitives for different types of cache resetting.

* **Hard reset** to clear cache and reload the query
* **Soft reset** to clear cache, but update, and leave current results on screen
* Can also update the raw cache

It does not parse your queries or mutations on the client-side like Apollo and Urql. This keeps the library small and omits the GraphQL queries from your bundle.

## Section and Distributed GraphQL

I know nothing about this and this article's length is already out of control but I found one nascent approach that seems worth mentioning. A company called [Section](https://www.section.io/blog/caching-distributed-graphql-at-the-edge/) is trying to build a distributed GraphQL solution.

It is fully configurable to address caching challenges without having to maintain a distributed system as the distributed system would be managed by them. They say that it's simultaneously similar to Apollo Federation but also solving a problem Apollo Federation doesn't solve, so I'm curious how exactly that works.

On first look it seems like they are taking the approach of `micro-graphql-react` and giving more cache control back to the developers.

## Persistent Queries

One more thing getting thrown around in this conversation that I'll need an addition article to cover is [persistent queries](https://www.apollographql.com/blog/persisted-graphql-queries-with-apollo-client-119fd7e6bba5/). The idea is to send a query `id` or hash instead of an entire GraphQL query string. This reduces bandwidth utilization and speeds up loading times for end-users.

## Resources

**Caching GraphQL**

* Mark Nottingham - [Caching Tutorial for Web Authors and Webmasters](https://www.mnot.net/cache_docs/)
* GraphQL.org - [Caching](https://graphql.org/learn/caching/)
* Sam Silver - [GraphQL Client-Side Caching](https://medium.com/@sasilver0051/graphql-client-side-caching-in-2019-76e97bffecfc)
* Scott Walkinshaw - [Caching GraphQL APIs](https://www.youtube.com/watch?v=MPRQRlrixls)
* Tanmai Gopal - [An approach to automated caching for public & private GraphQL APIs](https://www.youtube.com/watch?v=HJPYnUT5unw)

**Apollo**

* Dhaivat Pandya - [GraphQL Concepts Visualized](https://www.apollographql.com/blog/the-concepts-of-graphql-bc68bd819be3/)
* Marc-André Giroux - [GraphQL & Caching: The Elephant in the Room](https://www.apollographql.com/blog/graphql-caching-the-elephant-in-the-room-11a3df0c23ad/)
* Blessing Krofegha - [Understanding Client-Side GraphQl With Apollo-Client In React Apps](https://www.smashingmagazine.com/2020/07/client-side-graphql-apollo-client-react-apps/)
* John Haykto - [GraphQL Client-Side Caching with Apollo Links](https://www.youtube.com/watch?v=8ZKpIB1pDw8)
* Marc-André Giroux - [Caching & GraphQL: Setting the Story Straight](https://www.youtube.com/watch?v=CV3puKM_G14)
* Ben Newman - [Fine Tuning Apollo Client Caching for Your Data Graph](https://www.youtube.com/watch?v=n_j8QckQN5I)
* Khalil Stemmler - [Using Apollo Client 3 as a State Management Solution](https://www.youtube.com/watch?v=ptrlku3Khds)

**urql**

* Kurt Kemple - [Intro to Urql](https://www.youtube.com/watch?v=kq_CeyxDJxk)
* Ben Awad - [Urql -  a new GraphQL Client](https://www.youtube.com/watch?v=9Z0rMXSghs4)
* Ken Wheeler - [Introduction to urql - A new GraphQL Client for React](https://www.youtube.com/watch?v=oyVowI2bYYI)
* Gerard Sans - [Comparing Apollo vs Urql](https://www.youtube.com/watch?v=UoIpmaFxWkA)
* Phil Pluckthun, Jovi De Croock - [Client-Side GraphQL Using URQL](https://www.youtube.com/watch?v=MYHYv9IxllU)
* Ryan Gilbert - [Taking Flight with URQL](https://www.youtube.com/watch?v=159xRFHjruU)

**micro-graphql-react**

* Adam Rackis - [A Different Approach to GraphQL Caching](https://adamrackis.dev/graphql-caching-and-micro/)
* Adam Rackis - [An Alternate Approach to GraphQL Caching](https://www.youtube.com/watch?v=8vnRNVkyq9I)