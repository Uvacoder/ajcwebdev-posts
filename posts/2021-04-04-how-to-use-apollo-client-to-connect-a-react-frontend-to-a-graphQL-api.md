---
title: How to use Apollo Client to Connect a React Frontend to a GraphQL API
canonical_url: https://stepzen.com/blog/how-to-use-apollo-client
description: How to create a React frontend from scratch and query a GraphQL API with Apollo Client
imageUrl: https://stepzen.com/images/blog/share-cards/apollo-react-graphql.png
date: 2021-04-08
tags:
  - stepzen
  - apollo
  - graphql
  - react
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1ywyg23nsdlaj6hqi0yy.jpeg
layout: layouts/post.njk
---

In our previous article, [Exploring JavaScript Client Libraries for GraphQL](https://stepzen.com/blog/javascript-graphql-libraries), we examined a variety of different GraphQL clients. In this article we will build a React frontend from scratch and focus on integrating Apollo Client.

The client will query a GraphQL API created from the JSONPlaceholder API. We are able to use the [`@rest` directive](https://stepzen.com/blog/how-to-connect-any-rest-backend) to easily translate the JSONPlaceholder API into a GraphQL schema.

![05-unordered-list-of-users](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6667k39b37vmhz2xokwc.png)

You can find all of the code at the [following repo](https://github.com/stepzen-samples/stepzen-react-tutorial).

## 1. Project Setup

We will not use a CLI tool like `create-react-app` to setup our project. Instead we will create all of our files and directories from scratch.

### Create root directory and initialize `package.json`

Our project will be called **`stepzen-react-tutorial`**.

```bash
mkdir stepzen-react-tutorial
cd stepzen-react-tutorial
npm init -y
```

### Add necessary dependencies

```bash
npm i @apollo/react-hooks apollo-boost graphql react react-dom react-scripts
```

Our dependencies will include the following React packages:
* `react` - Contains only the functionality necessary to define components
* `react-dom` - Entry point to the DOM
* `react-scripts` - Scripts and configuration used by [CRA](https://github.com/facebook/create-react-app)

And the following GraphQL and Apollo packages:
* `@apollo/react-hooks` - Common hooks for queries, mutations, and subscriptions
* `apollo-boost` - Pre-configured defaults for Apollo Client
* `graphql` - JavaScript reference implementation for GraphQL

### Add scripts to `package.json`

```json
"scripts": {
  "start": "react-scripts start",
  "build": "react-scripts build",
  "test": "react-scripts test",
  "eject": "react-scripts eject"
},
```

* `npm start` - Runs the app in the development mode on `http://localhost:3000`
* `npm test` - Launches the test runner in interactive watch mode
* `npm run build` - Builds the app for production to the `build` folder
* `npm run eject` - Removes the single build dependency from your project and exports configuration files and dependencies into your project as dependencies in `package.json`.

### Add browserslist to `package.json`

```json
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
```

Your entire `package.json` should look something like this:

```json
{
  "name": "stepzen-react-tutorial",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
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
  "description": "",
  "dependencies": {
    "@apollo/react-hooks": "^4.0.0",
    "apollo-boost": "^0.4.9",
    "graphql": "^15.5.0",
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    "react-scripts": "^4.0.3"
  }
}
```

## 2. Setup React App

To setup our React frontend create directories for `src`, `src/pages`, and `public`.

```bash
mkdir src src/pages public
```

Create an `index.js` file for our root component, an `index.html` file to load the root component, and a `HomePage` component.

```bash
touch src/index.js src/pages/HomePage.js public/index.html
```

### Create a home page with a title

```jsx
// src/pages/HomePage.js

export default function HomePage() {
  return (
    <>
      <h1>StepZen React Tutorial</h1>
    </>
  )
}
```

### Create root component to render the HomePage component

```jsx
// src/index.js

import React from "react"
import { render } from "react-dom"
import HomePage from "./pages/HomePage"

render(
  <React.StrictMode>
    <HomePage />
  </React.StrictMode>,
  document.getElementById("root")
)
```

### Create html boilerplate with root div

Our `index.html` file includes a simple boilerplate with a root `<div>` to load in our application.

```html
<!-- public/index.html -->

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1"
    />
    <meta
      name="theme-color"
      content="#000000"
    />
    <meta
      name="description"
      content="How to use Apollo Client to Connect a React Frontend to a GraphQL API"
    />

    <title>React + StepZen App</title>
  </head>
  
  <body>
    <noscript>
      You need to enable JavaScript to run this app.
    </noscript>

    <div id="root"></div>
    <!--
      This HTML file is a template.
    -->
  </body>
</html>
```

Start the development server on `localhost:3000` with the following command:

```bash
npm start
```

![01-index-stepzen-react-tutorial](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8kgkskweraeninudvgru.png)

### Create a Users component

The `Users` component will fetch user data from the [JSONPlaceholder API](https://jsonplaceholder.typicode.com).

```bash
mkdir src/components
touch src/components/Users.js
```

We will need to create our backend before we can fetch any data. For now just include a placeholder title for Users.

```jsx
// src/components/Users.js

export default function Users() {
  return (
    <>
      <h2>Users</h2>
    </>
  )
}
```

Import the `Users` component into `HomePage.js` and return `<Users />`.

```jsx
// src/pages/HomePage.js

import Users from "../components/Users"

export default function HomePage() {
  return (
    <>
      <h1>StepZen React Tutorial</h1>
      <Users />
    </>
  )
}
```

Return to the browser to see the Users title displayed beneath the initial title.

![02-import-users-component](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2clw8xkphk7z694ze4pk.png)

## 3. Setup GraphQL API

To setup our StepZen API create a `stepzen` directory containing a `schema` directory.

```bash
mkdir stepzen stepzen/schema
```

Create an `index.graphql` file for our `schema` and a `users.graphql` file for our `User` type and `Query` type.

```bash
touch stepzen/schema/users.graphql stepzen/index.graphql
```

### index.graphql defines all files making up the GraphQL schema

Every StepZen project requires an `index.graphql` that ties together all of our schemas. For this example we just have the `users.graphql` file included in our `@sdl` directive. The `@sdl` directive is a StepZen directive that specifies the list of files to assemble.

```graphql
# stepzen/index.graphql

schema
  @sdl(
    files: [ "schema/users.graphql" ]
  ) {
  query: Query
}
```

The `User` type includes an `id` for each `User` and information about the `User` such as their `name` and `email`.

```graphql
# stepzen/schema/users.graphql

type User {
  id: ID!
  name: String!
  username: String!
  email: String!
  phone: String!
  website: String!
}

type Query {
  getUsers: [User]
    @rest(
      endpoint:"https://jsonplaceholder.typicode.com/users"
    )
}
```

For our `Query` we just have a single query called `getUsers` that returns an array of `User` objects. The `@rest` directive accepts the `endpoint` from JSONPlaceholder.

### Deploy API

The `stepzen start` command uploads and deploys your API automatically.

```bash
stepzen start
```

It will ask you to include a name with the format `folder/name`. This is the destination you specify for your endpoint. I named my endpoint `stepzen-react/users`.

A browser window with a GraphiQL query editor can be used to query your new endpoint on `localhost:5000`. Enter the following query to test your endpoint:

```graphql
query getUsers {
  getUsers {
    id
    name
  }
}
```

![03-graphql-api-explorer](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/i7bbyjnbkgb05u1i6w75.png)

### Set environment variables

This also deployed our API to `https://username.stepzen.net/stepzen-react/users/__graphql`. We will use this endpoint for our environment variables. Create a `.env` file along with a `.gitignore` file.

```bash
touch .env .gitignore
```

Include `.env` and `node_modules` in your `.gitignore` file to prevent them from being included if we push the project to a public GitHub repository.

```gitignore
node_modules
.env
```

In `.env` set:
* Your StepZen API key to `REACT_APP_STEPZEN_API_KEY`
* The endpoint with your username provided by `stepzen start` to `REACT_APP_STEPZEN_ENDPOINT`

```
REACT_APP_STEPZEN_API_KEY=<YOUR_API_KEY_HERE>
REACT_APP_STEPZEN_ENDPOINT=<YOUR_ENDPOINT_HERE>
```

## 4. Setup Apollo Client

There are many different ways to make GraphQL requests as described in Brian's [previously mentioned article](https://stepzen.com/blog/javascript-graphql-libraries). We will use Apollo Client since it is familiar to many React developers, but future articles will explore alternative implementations.

### Create `utils` directory and `client.js` file

```bash
mkdir src/utils
touch src/utils/client.js
```

Import `ApolloClient` and instantiate a new `client` instance. Set your API key with an authorization header. Set your endpoint to `uri`.

```jsx
// src/utils/client.js

import ApolloClient from "apollo-boost"

const {
  REACT_APP_STEPZEN_API_KEY,
  REACT_APP_STEPZEN_ENDPOINT
} = process.env

export const client = new ApolloClient({
  headers: {
    Authorization: `Apikey ${REACT_APP_STEPZEN_API_KEY}`,
  },
  uri: REACT_APP_STEPZEN_ENDPOINT,
})
```

### Wrap `HomePage` with `ApolloProvider`

Return to `index.js` and import `ApolloProvider` and `client` from the `utils` directory. Wrap the `HomePage` component with `ApolloProvider` and pass the `client` prop with our environment variables to the provider.

```jsx
// src/index.js

import React from "react"
import { render } from "react-dom"
import { ApolloProvider } from "@apollo/react-hooks"
import { client } from "./utils/client"
import HomePage from "./pages/HomePage"

render(
  <React.StrictMode>
    <ApolloProvider client={client}>
      <HomePage />
    </ApolloProvider>
  </React.StrictMode>,
  document.getElementById("root")
)
```

### Create `queries` directory and `getUsers.js` file

```bash
mkdir src/queries
touch src/queries/getUsers.js
```

Import `gql` and initialize `GET_USERS_QUERY` with a `query` called `getUsers`. This query returns the `id` and `name` of the users.

```jsx
// src/queries/getUsers.js

import { gql } from "graphql-tag"

export const GET_USERS_QUERY = gql`
  query getUsers {
    getUsers {
      id
      name
    }
  }
`
```

### Pass `GET_USERS_QUERY` into `useQuery`

Return to the `Users` component and import the `useQuery` hook and `GET_USERS_QUERY` from `getUsers.js`.

```jsx
// src/components/Users.js

import { useQuery } from "@apollo/react-hooks"
import { GET_USERS_QUERY } from "../queries/getUsers.js"
```

To perform the query, pass `GET_USERS_QUERY` into the `useQuery` hook.

```jsx
// src/components/Users.js

export default function Users() {
  const {
    data,
    loading,
    error
  } = useQuery(GET_USERS_QUERY)
}
```

We have three possible results from our query.
* Check `loading` and display loading message if `true`.
* Check `error` and display error message if `true`.
* If `data` was received successfully set the response to `users`.

```jsx
// src/components/Users.js

export default function Users() {
  const {
    data,
    loading,
    error
  } = useQuery(GET_USERS_QUERY)

  const users = data?.getUsers
  
  if (loading) return <p>Almost there...</p>
  if (error) return <p>{error.message}</p>
}
```

The `JSON.stringify()` method converts a JavaScript object or value to a JSON string, optionally replacing values if a replacer function is specified. The value of `null` is provided so all properties of the object are included in the resulting JSON string.

```jsx
// src/components/Users.js

  return (
    <>
      <h2>Users</h2>
      <pre>
        {JSON.stringify(users, null, "  ")}
      </pre>
    </>
  )
```

Here is our complete component.

```jsx
// src/components/Users.js

import { useQuery } from "@apollo/react-hooks"
import { GET_USERS_QUERY } from "../queries/getUsers.js"

export default function Users() {
  const {
    data,
    loading,
    error
  } = useQuery(GET_USERS_QUERY)

  const users = data?.getUsers
  
  if (loading) return <p>Almost there...</p>
  if (error) return <p>{error.message}</p>
  
  return (
    <>
      <h2>Users</h2>
      <pre>
        {JSON.stringify(users, null, "  ")}
      </pre>
    </>
  )
}
```

![04-fetch-users-from-jsonplaceholder](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xw3dvkovbdbikjxrride.png)

### Map over `users` array

To display a list of each user we can map over the `users` array. This will return:
* An unordered list of each `user` with the user's `id` set to the `key` property
* Each user's `name` set to a list item

```jsx
return (
  <>
    <h2>Users</h2>

    {users.map(user => (
      <ul key={user.id}>
        <li>
          {user.name}
        </li>
      </ul>
    ))}
  </>
)
```

![05-unordered-list-of-users](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6667k39b37vmhz2xokwc.png)

## Next Steps

We have now seen how to setup our React frontend with Apollo Client to query a GraphQL API deployed to StepZen. With this foundation we could continue building out the project with other libraries such as [React Router](https://github.com/ReactTraining/react-router) for linking between pages, [Next.js](https://github.com/vercel/next.js/) for server-side rendering, or [Tailwind CSS](https://github.com/tailwindlabs/tailwindcss) for styling.

However, we could not deploy our application in its current state because our client has access to our API keys. We would need to deploy a [secure API route](https://stepzen.com/blog/how-to-secure-api-routes-for-jamstack-sites) with a function containing our keys. This can be easily accomplished with frameworks such as RedwoodJS as we will see in future articles. This is because it includes an API side auto-configured out of the box to deploy a GraphQL handler to AWS Lambda.

You can find more example repositories at [StepZen Samples](http://github.com/stepzen-samples). If you're not already part of the private alpha, you can request an invitation [here](https://www.stepzen.com/request-invite).