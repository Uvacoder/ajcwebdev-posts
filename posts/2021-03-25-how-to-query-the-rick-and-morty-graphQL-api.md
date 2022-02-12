---
title: how to query the rick and morty graphQL API
description: How to query the Rick and Morty GraphQL API with the fetch API, graphql-request, and Apollo Client.
date: 2021-03-25
tags:
  - graphql
  - api
  - javascript
  - fetch
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ksq0lckoen89nql30g1x.jpeg
layout: layouts/post.njk
---

I've been going deep into GraphQL ever since I first started learning Redwood, and it's been an interesting experiment because I started with a fully complete GraphQL project with a server and client included and integrated.

As I've gotten deeper into GraphQL I've realized this is an incredible exception to the rule, the norm is everyone creating their own bespoke combination of clients and/or servers to fit their own purposes.

So if we wanted to take it to the total basics, you'd want to start with actually making a GraphQL query. For example, if you were to go to the following [link](https://rickandmortyapi.com/graphql) you'll see this:

![01-rick-and-morty-graphiql](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/p71t3vphq4c9vt1r3suo.png)

We want to make a query, so we'll enter the following `query` for `characters`, specifically their `name` (the `results` array is a quirk of this specific GraphQL schema).

```graphql
query {
  characters {
    results {
      name
    }
  }
}
```

This returns an array of names.

![02-character-names](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sdvapzwb74l4bw38p73w.png)

Watch out for Abradolf Lincler, he's a bad dude.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8x4jgd4v9bfpilv5dzut.png)

## Make a fetch requests

The next layer would be making a `fetch` request.

### Create directories

```bash
mkdir public src
```

### Create index.html and index.js

```bash
touch public/index.html src/index.js
```

### index.html

Enter the following `html` boilerplate with a `script` tag for `index.js`.

```html
<!-- fetch-api/public/index.html -->

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0"
    >
    <meta
      http-equiv="X-UA-Compatible"
      content="ie=edge"
    >

    <title>How to Query the Rick and Morty GraphQL API</title>

    <script src="../src/index.js" defer></script>
  </head>

  <body>
    <noscript>
      You need to enable JavaScript to run this app.
    </noscript>

    <h1>
      Fetch API
    </h1>

    <div id="root"></div>
  </body>
</html>
```

### index.js

Make a `fetch` request to `https://rickandmortyapi.com/graphql` including:
* A `POST` request with `Content-Type` of `application/json`
* The `characters` query we wrote above asking for their `name` included in the `body` and [stringified](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)
* The `results` displayed with `console.log()`

```javascript
// fetch-api/src/index.js

fetch('https://rickandmortyapi.com/graphql', {
  method: 'POST',

  headers: {
    "Content-Type": "application/json"
  },

  body: JSON.stringify({
    query: `
      query getCharacters {
        characters {
          results {
            name
          }
        }
      }
    `
  })
})
.then(res => res.json())
.then(data => console.log(data.data))
```

![data-from-rick-and-morty-api](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/avaw3gtjz2ljp6ozo6mt.png)

This doesn't require installing dependencies, or even creating a *`package.json`* file. However, there are many GraphQL client libraries which explore a wide range of trade offs. Use cases may include providing concise abstractions for common GraphQL functionality or adding additional features such as caching.

## graphql-request

[graphql-request](https://github.com/prisma-labs/graphql-request) is a minimal GraphQL client that supports Node and browsers.

### Create directories

```bash
mkdir graphql-request graphql-request/public graphql-request/src
```

### Create index.html and index.js

```bash
touch graphql-request/public/index.html graphql-request/src/index.js
```

### Initialize project and install dependencies

```bash
cd graphql-request
yarn init -y
yarn add graphql graphql-request react react-dom react-scripts
```

### Add scripts and browsers list

```json
{
  "name": "how-to-query-a-graphql-api-with-graphql-request",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "dependencies": {
    "graphql": "^15.5.0",
    "graphql-request": "^3.4.0",
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    "react-scripts": "^4.0.3"
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

### index.html

```html
<!-- graphql-request/public/index.html -->

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0"
    >
    <meta
      http-equiv="X-UA-Compatible"
      content="ie=edge"
    >

    <title>How to Query the Rick and Morty GraphQL API</title>
  </head>

  <body>
    <noscript>
      You need to enable JavaScript to run this app.
    </noscript>

    <div id="root"></div>
  </body>
</html>
```

### index.js

```jsx
// graphql-request/src/index.js

import React from "react"
import { render } from "react-dom"
import { GraphQLClient, gql } from 'graphql-request'

async function main() {
  const endpoint = 'https://rickandmortyapi.com/graphql'
  const graphQLClient = new GraphQLClient(endpoint)

  const GET_CHARACTERS_QUERY = gql`
    query getCharacters {
      characters {
        results {
          name
        }
      }
    }
  `

  const data = await graphQLClient.request(GET_CHARACTERS_QUERY)
  console.log(JSON.stringify(data, undefined, 2))
}

main()

render(
  <React.StrictMode>
    <h1>graphql-request</h1>
  </React.StrictMode>,
  document.getElementById("root")
)
```

## Apollo Client

[Apollo Client](https://github.com/apollographql/apollo-client) is a caching GraphQL client with integrations for React and other popular frontend libraries/frameworks.

### Create directories

```bash
cd ..
mkdir apollo-client apollo-client/public apollo-client/src
```

### Create index.html and index.js

```bash
touch apollo-client/public/index.html apollo-client/src/index.js
```

### Initialize project and install dependencies

```bash
cd apollo-client
yarn init -y
yarn add graphql react react-dom react-scripts @apollo/react-hooks apollo-boost
```

### Add scripts and browser list

```json
{
  "name": "how-to-query-a-graphql-api-with-apollo-client",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "dependencies": {
    "@apollo/react-hooks": "^4.0.0",
    "apollo-boost": "^0.4.9",
    "graphql": "^15.5.0",
    "graphql-request": "^3.4.0",
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    "react-scripts": "^4.0.3"
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  },
  "keywords": [],
  "author": "",
  "license": "MIT"
}
```

### index.html

```html
<!-- apollo-client/public/index.html -->

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0"
    >
    <meta
      http-equiv="X-UA-Compatible"
      content="ie=edge"
    >

    <title>How to Query the Rick and Morty GraphQL API</title>
  </head>

  <body>
    <noscript>
      You need to enable JavaScript to run this app.
    </noscript>

    <div id="root"></div>
  </body>
</html>
```

### Create Apollo Client

```jsx
// apollo-client/src/index.js

import React from "react"
import { render } from "react-dom"
import { ApolloProvider } from "@apollo/react-hooks"
import ApolloClient from "apollo-boost"
import gql from "graphql-tag"
import { useQuery } from "@apollo/react-hooks"

const endpoint = 'https://rickandmortyapi.com/graphql'

export const client = new ApolloClient({
  uri: endpoint
})

export const GET_CHARACTERS_QUERY = gql`
  query getCharacters {
    characters {
      results {
        name
    }
  }
}
`

function Characters() {
  const {
    data,
    loading,
    error
  } = useQuery(GET_CHARACTERS_QUERY)

  const characters = data?.characters

  if (loading) return <p>Almost there...</p>
  if (error) return <p>{error.message}</p>

  return (
    <>
      <pre>
        {JSON.stringify(characters, null, "  ")}
      </pre>
    </>
  )
}

render(
  <React.StrictMode>
    <ApolloProvider client={client}>
      <h1>Apollo Client</h1>
      <Characters />
    </ApolloProvider>
  </React.StrictMode>,
  document.getElementById("root")
)
```



### Extract client

```bash
touch src/client.js
```

```jsx
// apollo-client/src/client.js

import ApolloClient from "apollo-boost"

const endpoint = 'https://rickandmortyapi.com/graphql'

export const client = new ApolloClient({
  uri: endpoint
})
```

### Import client

```jsx
// apollo-client/src/index.js

import React from "react"
import { render } from "react-dom"
import { ApolloProvider } from "@apollo/react-hooks"
import { client } from "../src/client.js"
import gql from "graphql-tag"
import { useQuery } from "@apollo/react-hooks"

export const GET_CHARACTERS_QUERY = gql`
  query getCharacters {
    characters {
      results {
        name
    }
  }
}
`

function Characters() {
  const {
    data,
    loading,
    error
  } = useQuery(GET_CHARACTERS_QUERY)

  const characters = data?.characters

  if (loading) return <p>Almost there...</p>
  if (error) return <p>{error.message}</p>

  return (
    <>
      <pre>
        {JSON.stringify(characters, null, "  ")}
      </pre>
    </>
  )
}

render(
  <React.StrictMode>
    <ApolloProvider client={client}>
      <h1>Apollo Client</h1>
      <Characters />
    </ApolloProvider>
  </React.StrictMode>,
  document.getElementById("root")
)
```

### Extract query

```bash
touch src/getCharacters.js
```

```jsx
// apollo-client/src/getCharacters.js

import gql from "graphql-tag"

export const GET_CHARACTERS_QUERY = gql`
  query getCharacters {
    characters {
      results {
        name
    }
  }
}
`
```

### Import query

```jsx
// apollo-client/src/index.js

import React from "react"
import { render } from "react-dom"
import { ApolloProvider } from "@apollo/react-hooks"
import { client } from "../src/client.js"
import { GET_CHARACTERS_QUERY } from "../src/getCharacters.js"
import { useQuery } from "@apollo/react-hooks"

function Characters() {
  const {
    data,
    loading,
    error
  } = useQuery(GET_CHARACTERS_QUERY)

  const characters = data?.characters

  if (loading) return <p>Almost there...</p>
  if (error) return <p>{error.message}</p>

  return (
    <>
      <pre>
        {JSON.stringify(characters, null, "  ")}
      </pre>
    </>
  )
}

render(
  <React.StrictMode>
    <ApolloProvider client={client}>
      <h1>Apollo Client</h1>
      <Characters />
    </ApolloProvider>
  </React.StrictMode>,
  document.getElementById("root")
)
```

### Extract Characters component

```bash
touch src/Characters.js
```

```jsx
// apollo-client/src/Characters.js

import { GET_CHARACTERS_QUERY } from "../src/getCharacters.js"
import { useQuery } from "@apollo/react-hooks"

export default function Characters() {
  const {
    data,
    loading,
    error
  } = useQuery(GET_CHARACTERS_QUERY)

  const characters = data?.characters

  if (loading) return <p>Almost there...</p>
  if (error) return <p>{error.message}</p>

  return (
    <>
      <pre>
        {JSON.stringify(characters, null, "  ")}
      </pre>
    </>
  )
}
```

### Import Characters component

```jsx
// apollo-client/src/index.js

import React from "react"
import { render } from "react-dom"
import { ApolloProvider } from "@apollo/react-hooks"
import { client } from "../src/client.js"
import Characters from "../src/Characters.js"

render(
  <React.StrictMode>
    <ApolloProvider client={client}>
      <h1>Apollo Client</h1>
      <Characters />
    </ApolloProvider>
  </React.StrictMode>,
  document.getElementById("root")
)
```