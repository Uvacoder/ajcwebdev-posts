---
title: a first look at nuxt 3
description: The goal of Nuxt is to make web development intuitive and performant while keeping great developer experience in mind.
date: 2021-10-15
tags:
  - nuxt
  - vue
  - nitro
  - vite
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1xgexxq7w0ejq02zqd5r.png
layout: layouts/post.njk
---

>*We are excited to open source Nuxt 3 after more than a year of intense development. On top of supporting Vue 3 or Vite, Nuxt 3 contains a new server engine, unlocking new full-stack capablities to Nuxt server and beyond.*
>
>*[Introducing Nuxt 3 Beta](https://nuxtjs.org/announcements/nuxt3-beta/)*

The goal of Nuxt is to make web development intuitive and performant while keeping great developer experience in mind. The original version was created by [Sébastien Chopin in October 2016](https://github.com/nuxt/nuxt.js/commit/0072ed31da6ce39d21046e05898f956cff190390) to emulate the features of Next.js but with Vue instead of React. Version 3 has been [over a year in the making](https://nuxtjs.org/announcements/nuxt3-beta/) and is composed of the following [core packages](https://github.com/nuxt/framework/tree/main/packages):

- Core Engine: [nuxt3](https://github.com/nuxt/framework/tree/main/packages/nuxt3)
- Bundlers: [@nuxt/vite-builder](https://github.com/nuxt/framework/tree/main/packages/vite) and [@nuxt/webpack-builder](https://github.com/nuxt/framework/tree/main/packages/webpack)
- Command line interface: [nuxi](https://github.com/nuxt/framework/tree/main/packages/nuxi)
- Server engine: [@nuxt/nitro](https://github.com/nuxt/framework/tree/main/packages/nitro)
- Development kit: [@nuxt/kit](https://github.com/nuxt/framework/tree/main/packages/kit)
- Nuxt 2 Bridge: [@nuxt/bridge](https://github.com/nuxt/framework/tree/main/packages/bridge)

Together these packages provide a selection of libraries for managing many common concerns for developers building on the web today such as:

- A JavaScript framework to bring reactivity and web components - [Vue.js](https://v3.vuejs.org)
- A bundler to support hot module replacement in development and bundling for production - [Webpack 5](https://webpack.js.org/) and [Vite](https://vitejs.dev/) both supported
- A transpiler for writing the latest JavaScript syntax while supporting legacy browsers - [esbuild](https://esbuild.github.io)
- A server that can serve your application in development and support [server-side rendering](https://v3.vuejs.org/guide/ssr/introduction.html#what-is-server-side-rendering-ssr) or API routes - [h3](https://github.com/unjs/h3)
- A routing library to handle client-side navigation - [vue-router](https://next.router.vuejs.org)

In addition to curating and integrating these tools, Nuxt also provides directory structure conventions for managing pages and components.

### How to Migrate a Nuxt 2 Project to Nuxt 3

*If you don't have a Nuxt 2 project, skip ahead to the section, "Create a Nuxt 3 project from scratch."*

At the moment, there is no Nuxt 2 to Nuxt 3 migration guide nor is it recommended due to potentially more changes coming. The team is working to provide a stable migration guide and tooling to make it as smooth as possible in the form of Nuxt Bridge. If you have an existing Nuxt 2 project, the team **strongly recommends** you begin by using Nuxt Bridge to experiment with new features while avoiding breaking changes. Bridge is a forward-compatibility layer that allows you to experience new Nuxt 3 features by installing and enabling a Nuxt module.

All Nuxt 2 modules should be forward compatible with Nuxt 3 as long as they migrate to bridge or if they are already following guidelines. All (upcoming) modules made with `@nuxt/kit` should be backward compatible with Nuxt 2 projects (even without bridge) as long as they are not depending on a Nuxt 3 / Bridge-only feature. Since Nuxt 3 natively supports TypeScript and ECMAScript Modules, it is useful to avoid CommonJS syntax such as `__dirname`, `__filename`, `require()`, and `module.exports` as much as possible.

### Try online

We will be building a Nuxt application from scratch. However, you can start playing with Nuxt 3 online in your browser on either [StackBlitz](https://stackblitz.com/github/nuxt/starter/tree/v3-stackblitz) or [CodeSandBox](https://codesandbox.io/s/github/nuxt/starter/tree/v3-codesandbox).

## Create a Nuxt 3 project from scratch

The project will start with only three files:
* `.gitignore` to avoid accidentally committing stuff that shouldn't be committed
* `package.json` to define our scripts and dependencies
* `app.vue` to display our Vue application

We will create a directory for `pages` and an `api` later on.

```bash
mkdir ajcwebdev-nuxt3
cd ajcwebdev-nuxt3
touch package.json app.vue
echo 'node_modules\n.DS_Store\n*.log\n.nuxt\nnuxt.d.ts\n.output' > .gitignore
```

All the code for this article can be found [on my GitHub](https://github.com/ajcwebdev/ajcwebdev-nuxt3).

### App file - `app.vue`

The `app.vue` file is the main component in your Nuxt 3 applications. With Nuxt 3, the `pages/` directory is optional, if not present, Nuxt won't include [vue-router](https://next.router.vuejs.org/) dependency. This is useful when working on a landing page or an application that does not need routing.

```html
<!-- app.vue -->

<template>
  <div>
    <h2>ajcwebdev-nuxt3</h2>
  </div>
</template>
```

`app.vue` acts as the main component of your Nuxt application. This means anything you add to it, such as JavaScript or CSS, will be global and included in every page.

### `package.json`

In your `package.json`, add the following scripts (`dev`, `build`, and `start`) along with the `latest` version of `nuxt3` as a development dependency.

```json
{
  "scripts": {
    "dev": "nuxi dev",
    "build": "nuxi build",
    "start": "node .output/server/index.mjs"
  },
  "devDependencies": {
    "nuxt3": "latest"
  }
}
```

Nuxi is the new CLI for Nuxt 3 and has two main commands:
1. `nuxi dev` - Start development server
2. `nuxi build` - Make production assets

We also created a `start` script that uses Node to run the bundled output generated for the server by `nuxi build`.

### Start development server

The `yarn dev` command starts your Nuxt app in development mode and includes hot module replacement. You can include the `--open` flag to automatically open the browser after starting up.

```bash
yarn dev
```

The CLI will display links to the running application and performance metrics.

```
Nuxt CLI v3.0.0-27237303.6acfdcd

  > Local:    http://localhost:3000/
  > Network:  http://192.168.1.242:3000/

ℹ Vite warmed up in 592ms
✔ Generated nuxt.d.ts
✔ Vite server built in 903ms
✔ Nitro built in 112 ms
```

Open [localhost:3000](http://localhost:3000) to see your application.

![01-ajcwebdev-nuxt3-localhost-3000](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9oc49oolv9d31ljml352.png)

### Build for production

The `yarn build` command builds your Nuxt application for production. It creates a `.output` directory with your application, server, and dependencies ready to be deployed.

```bash
yarn build
```

Nitro produces a standalone server dist that is independent of `node_modules`. The output is combined with both runtime code to run your Nuxt server in any environment and serve you static files. A native storage layer is also implemented for supporting multi source, drivers and local assets.

## Pages directory

The `pages/` directory is optional, meaning that if you only use `app.vue`, `vue-router` won't be included, reducing your application bundle size. However, if you do include it, Nuxt will automatically integrate [Vue Router](https://next.router.vuejs.org/) and map `pages/` directory into the routes of your application.

```bash
rm -rf app.vue
mkdir pages
touch pages/about.vue pages/index.vue
```

We deleted `app.vue` and will include the previous home page's content in `index.vue`.

```html
<!-- pages/index.vue -->

<template>
  <div>
    <h2>ajcwebdev-nuxt3</h2>
  </div>
</template>
```

We also created `about.vue` for an about page. Include the following code to make sure that people know stuff about your things.

```html
<!-- pages/about.vue -->

<template>
  <div>
    <h2>This page tells you stuff about things!</h2>
  </div>
</template>
```

Open [localhost:3000/about](http://localhost:3000/about).

![02-about-page-localhost-3000](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lyngyw7ae27x213gfu4f.png)

## Server Engine

Nuxt 3 is powered by a new server engine called Nitro. Nitro is used in development and production. It includes cross-platform support for Node.js, Browsers, and service-workers and serverless support out-of-the-box. Other features include API routes, automatic code-splitting, async-loaded chunks, and hybrid static/serverless modes. Server API endpoints and Middleware that internally uses [h3](https://github.com/unjs/h3) are added by Nitro.

* Handlers can directly return objects/arrays for an automatically-handled JSON response
* Handlers can return promises, which will be awaited (`res.end()` and `next()` are also supported)
* Helper functions for body parsing, cookie handling, redirects, and headers

Nitro allows 'direct' calling of routes via the globally-available `$fetch` helper. This will make an API call to the server if run on the browser, but will simply call the relevant function if run on the server, **saving an additional API call**. The `$fetch` API uses [ohmyfetch](https://github.com/unjs/ohmyfetch) to:

* Automatically parse JSON responses (with access to raw response if needed)
* Automatically handle request body and params with correct `Content-Type` headers added

### Server directory for API routes

The `server/` directory contains API endpoints and server middleware for your project. It is used to create any backend logic for your Nuxt application. Nuxt will automatically read in any files in the `~/server/api` directory to create API endpoints. Each file should export a default function that handles API requests.

```bash
mkdir -p server/api
touch server/api/hello.js
```

Add the following code to `hello.js`.

```js
// server/api/hello.js

export default (req, res) => '<h2>Hello from Nuxt 3</h2>'
```

Open [localhost:3000/api/hello](http://localhost:3000/api/hello).

![03-hello-api-route-localhost-3000](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9g8nobz0clkvvmhw8e5o.png)

## Deploy to Netlify

What's the point of a framework if you can't deploy it on a Jamstack platform?

```bash
touch netlify.toml
```

```toml
[build]
  command = "yarn build"
  publish = "dist"
  functions = ".output/server"

[[redirects]]
  from = "/*"
  to = "/.netlify/functions/index"
  status = 200
```

### Create GitHub Repository

```bash
git init
git add .
git commit -m "the nuxt best thing"
gh repo create ajcwebdev-nuxt3
git push -u origin main
```

Connect your repo to your Netlify account.

![04-connect-repo-to-netlify](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/z592uzltp4s5l3oyhzqv.png)

The build command and publish directory will be included automatically from the `netlify.toml` file.

![05-netlify-site-settings](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/859hdgl9xdp8sxxmiosx.png)

Lastly, give yourself a custom domain.

![06-add-custom-domain](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g8o7fjemh6o7f8n8hbhd.png)

Open [ajcwebdev-nuxt3.netlify.app/](https://ajcwebdev-nuxt3.netlify.app/).

![07-ajcwebdev-nuxt3-netlify-deploy](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mrlsbltxfvo6rayxcbtl.png)