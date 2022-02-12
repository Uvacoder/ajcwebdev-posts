---
title: a first look at remix
description: Remix is a React metaframework that primarily uses standard web APIs. In this example we will add graphql-request to the Netlify starter.
date: 2021-05-06
tags:
  - remix
  - react
  - netlify
  - graphql
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rjjif096a0t4b14wpmog.jpeg
layout: layouts/post.njk
---

[Remix](https://dev.to/ajcwebdev/remix-30h0) is a React metaframework that primarily uses standard web APIs. It originally required a paid license but has recently been released as a fully open source project. In this example we will add [graphql-request](https://github.com/prisma-labs/graphql-request) to the Netlify starter.

## Outline

* [Initialize Starter Project](#initialize-starter-project)
* [Start Development Server](#start-development-server)
* [Index Routes](#index-routes)
* [Include Water CSS for Styling Presets](#include-water-css-for-styling-presets)
* [Loader Functions](#loader-functions)
* [Deploy to Netlify](#deploy-to-netlify)

### Initialize Starter Project

```bash
npx create-remix@latest ajcwebdev-remix
```

I will select the Netlify starter.

```
💿 Welcome to Remix! Let's get you set up with a new project.

? Where would you like to create your app? ./ajcwebdev-remix
? Where do you want to deploy? Choose Remix if you're unsure, it's easy to change deployment targets. Netlify
? TypeScript or JavaScript? JavaScript
? Do you want me to run `npm install`? Yes

💿 That's it! `cd` into "ajcwebdev-remix" and check the README for development and deploy instructions!
```

### Start Development Server

`cd` into your project, install the Netlify CLI, and start the development server.

```bash
cd ajcwebdev-remix
npm i -g netlify-cli@latest
npm run build
npm run dev
```

```
◈ Netlify Dev ◈
◈ Ignored general context env var: LC_ALL (defined in process)
◈ No app server detected. Using simple static server
◈ Running static server from "ajcwebdev-remix/public"
◈ Loaded function server (​http://localhost:3000/.netlify/functions/server​).
◈ Functions server is listening on 57433

◈ Static server listening to 3999

   ┌─────────────────────────────────────────────────┐
   │                                                 │
   │   ◈ Server now ready on http://localhost:3000   │
   │                                                 │
   └─────────────────────────────────────────────────┘

Watching Remix app in development mode...
◈ Rewrote URL to /.netlify/functions/server
Request from ::1: GET /.netlify/functions/server
```

Open [localhost:3000](https://localhost:3000) to see the project.

![01-home-page-on-localhost-3000](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wzea0drtxlyrbd1ttpdq.png)

### Index Routes

[`index` routes](https://remix.run/guides/routing#index-routes) are routes that renders when the layout's path is matched exactly. If you have an `index.jsx` file in the `routes` directory it will be used as the home page. I've made a few edits to the boilerplate code.

```jsx
// app/routes/index.jsx

import { useLoaderData, json } from "remix"

export let loader = () => {
  let data = {
    resources: [
      { name: "A First Look at Remix Blog Post", url: "https://dev.to/ajcwebdev/a-first-look-at-remix-2cp1" },
      { name: "Example Repo", url: "https://github.com/ajcwebdev/ajcwebdev-remix" },
      { name: "Deployed Website", url: "https://ajcwebdev-remix.netlify.app" }
    ]
  }
  return json(data)
}

export let meta = () => {
  return {
    title: "ajcwebdev-remix",
    description: "Welcome to remix!"
  }
}

export default function Index() {
  let data = useLoaderData();

  return (
    <div className="remix__page">
      <main>
        <h1>ajcwebdev-remix</h1>
        <p>Woot!</p>
      </main>

      <section>        
        <h2>Resources</h2>
        <ul>
          {data.resources.map(resource => (
            <li key={resource.url}>
              <a href={resource.url}>{resource.name}</a>
            </li>
          ))}
        </ul>
      </section>
    </div>
  )
}
```

[`json`](https://remix.run/api/remix#json) provides a shortcut for creating `application/json` responses and [`meta`](https://remix.run/api/conventions#meta) sets meta tags for the HTML document.

![02-home-page-on-localhost-3000-edit](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lvnho8akn8yhps3li0u7.png)

### Include Water CSS for Styling Presets

```jsx
// app/root.jsx

import {
  Links,
  LiveReload,
  Meta,
  Outlet,
  Scripts,
  ScrollRestoration
} from "remix";

export function meta() {
  return { title: "New Remix App" };
}

export default function App() {
  return (
    <html lang="en">
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width,initial-scale=1" />
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/water.css@2/out/dark.css"></link>
        <Meta />
        <Links />
      </head>
      <body>
        <Outlet />
        <ScrollRestoration />
        <Scripts />
        {process.env.NODE_ENV === "development" && <LiveReload />}
      </body>
    </html>
  );
}
```

![03-home-page-with-water-css](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/demc5fj0o0spog5xz8do.png)

### Loader Functions

[Loaders](https://remix.run/api/conventions#loader) provide data to components and are only ever called on the server. You can connect to a database or run any server side code next to the component that renders it. Create a new route for a GraphQL query called `graphql.jsx`.

```bash
touch app/routes/graphql.jsx
npm i graphql-request graphql
```

Each route can define a "loader" function that will be called on the server before rendering to provide data to the route. The function will only run on the server, making it an ideal candidate for requests that include API secrets that cannot be exposed on the client.

```jsx
// app/routes/graphql.jsx

import { useLoaderData, json } from "remix"
import { GraphQLClient, gql } from "graphql-request"

const GET_CHARACTERS = gql`{
  characters {
    results {
      name
      id
    }
  }
}`

export let loader = async () => {
  const client = new GraphQLClient("https://rickandmortyapi.com/graphql")
  const { characters } = await client.request(GET_CHARACTERS)
  const { results } = characters
  return json({ results })
}

export default function Index() {
  let data = useLoaderData()

  return (
    <>
      <ul>
        {data.results.map(({ name, id }) => (
          <li key={id}>
            {name}
          </li>
        ))}
      </ul>
    </>
  )
}
```

![04-graphql-route-with-characters-data](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/swijfauollj5d2tutrk2.png)

### Deploy to Netlify

The starter already includes a `netlify.toml` file with build instructions.

```toml
[build]
  command = "remix build"
  functions = "netlify/functions"
  publish = "public"

[dev]
  command = "remix watch"
  port = 3000

[[redirects]]
  from = "/*"
  to = "/.netlify/functions/server"
  status = 200

[[headers]]
  for = "/build/*"
  [headers.values]
    "Cache-Control" = "public, max-age=31536000, s-maxage=31536000"
```

Create a GitHub repository.

```bash
git init
git add .
git commit -m "re-re-re-re-re-remix"
gh repo create ajcwebdev-remix
git push -u origin main
```

Initialize a Netlify site with the Netlify CLI.

```bash
netlify init
```

```
? What would you like to do? +  Create & configure a new site
? Team: Anthony Campolo's team
? Site name (optional): ajcwebdev-remix

Site Created

Admin URL: https://app.netlify.com/sites/ajcwebdev-remix
URL:       https://ajcwebdev-remix.netlify.app
Site ID:   e80b3ba6-3bc8-4018-8b27-4f1b62599ac9
? Your build command (hugo build/yarn run build/etc): npm run build
? Directory to deploy (blank for current dir): public
? Netlify functions folder: netlify/functions
Adding deploy key to repository...
Deploy key added!

Creating Netlify GitHub Notification Hooks...
Netlify Notification Hooks configured!

Success! Netlify CI/CD Configured!

This site is now configured to automatically deploy from github branches & pull requests

Next steps:

  git push       Push to your git repository to trigger new site builds
  netlify open   Open the Netlify admin URL of your site
```

The README includes a command to set the version of Node to v14.

```bash
netlify env:set AWS_LAMBDA_JS_RUNTIME nodejs14.x
netlify deploy --prod
```

```
Deploy path:        /Users/ajcwebdev/ajcwebdev-remix/public
Functions path:     /Users/ajcwebdev/ajcwebdev-remix/netlify/functions
Configuration path: /Users/ajcwebdev/ajcwebdev-remix/netlify.toml
Deploying to main site URL...
✔ No cached functions were found
✔ Finished hashing 35 files and 1 functions
✔ CDN requesting 24 files and 1 functions
✔ Finished uploading 25 assets
✔ Deploy is live!

Logs:              https://app.netlify.com/sites/ajcwebdev-remix/deploys/61a9732cec7b1a6162064d06
Unique Deploy URL: https://61a9732cec7b1a6162064d06--ajcwebdev-remix.netlify.app
Website URL:       https://ajcwebdev-remix.netlify.app
```

Open your [website URL](https://ajcwebdev-remix.netlify.app) to see the deployed project.

![05-ajcwebdev-remix-deployed-on-netlify](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tqhqscleamcboqsaiq08.png)