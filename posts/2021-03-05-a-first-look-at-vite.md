---
title: a first look at vite
description: Vite is a frontend build tool and open source project created by Evan You.
date: 2021-03-05
tags:
  - vite
  - vue
  - esbuild
  - javascript
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ir8uhnpqn61q0t36c3lh.png
layout: layouts/post.njk
---

[Vite](https://vitejs.dev/) (French word for "fast", pronounced `/vit/`, rhymes with "street") is a frontend build tool and open source project created by Evan You on [April 20, 2020](https://github.com/vitejs/vite/commit/820c2cfbefd376b7be2d0ba5ad1fd39d3e45347e) in between his second and third viewing of Dazed and Confused. [Vite 2.0](https://dev.to/yyx990803/announcing-vite-2-0-2f0a) was officially released on February 16, 2021 and aims to provide a faster and leaner development experience for modern web projects. It consists of two parts:

* A dev server with Hot Module Replacement (HMR) that serves your source files over [native ES modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
* A build command that bundles your code with [Rollup](https://rollupjs.org), pre-configured to output highly optimized static assets for production

## Outline

* [Create a Project from Scratch](#create-a-project-from-scratch)
  * [Create HTML Entry File](#create-html-entry-file)
  * [Install Vite Dependency](#install-vite-dependency)
  * [Add Dev Script](#add-dev-script)
  * [Start Development Server](#start-development-server)
  * [Create JavaScript Entry File](#create-javascript-entry-file)
  * [Create CSS Stylesheet](#create-css-stylesheet)
  * [Create a Single Page App that Renders a Root Component](#create-a-single-page-app-that-renders-a-root-component)
* [Create Vue App](#create-vue-app)
  * [Initialize Project](#initialize-project)
  * [Project Structure](#project-structure)
  * [App Vue Component](#app-vue-component)
  * [HelloWorld Component](#helloworld-component)
  * [Deploy to Netlify](#deploy-to-netlify)

## Create a Project from Scratch

You can find all the code for this article on my [GitHub](https://github.com/ajcwebdev/ajcwebdev-vite).

```bash
mkdir ajcwebdev-vite
cd ajcwebdev-vite
```

### Create HTML Entry File

```bash
touch index.html
```

```html
<!-- index.html -->

<h1>ajcwebdev</h1>
```

### Install Vite Dependency

```bash
yarn add -D vite
```

### Add Dev Script

Open `package.json` and add the follow script.

```json
{
  "scripts": {
    "dev": "vite"
  },
  "devDependencies": {
    "vite": "^2.0.5"
  }
}
```

### Start Development Server

```bash
yarn dev
```

```
vite v2.0.5 dev server running at:

> Local:    http://localhost:3000/
> Network:  http://10.0.0.175:3000/

ready in 258ms.
```

Open [localhost:3000](http://localhost:3000).

![01-index-html-hello-world](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fnnqefitacecm3d2q8dy.png)

Don't forget the `<title>`.

```html
<!-- index.html -->

<head>
  <title>ajcwebdev</title>
</head>

<body>
  <h1>ajcwebdev</h1>
</body>
```

![02-index-html-hello-world-with-title](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ukr408z1rnrjqinntd12.png)

We can import modules directly inside our `<script>` tags thanks to ES Modules.

```html
<!-- index.html -->

<head>
  <title>ajcwebdev</title>
</head>

<body>
  <h1>ajcwebdev</h1>

  <script type="module">
    import './main.js'
  </script>
</body>
```

We are trying to import `main.js` but we haven't created it yet. This will cause the server to display one of the most aesthetically pleasing error messages you will ever see.

![03-failed-to-resolve-import](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5zclwakxn74pfs4nu2bo.png)

### Create JavaScript Entry File

```bash
touch main.js
```

Console log a message to your dude.

```javascript
// main.js

console.log('sah dude')
```

![04-console-log-main-js](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/iqbiw2pwfyp0evnatgcv.png)

### Create CSS Stylesheet

```bash
touch style.css
```

You only get one color so make it count.

```css
/* style.css */

h1 {
  color: purple
}
```

![05-index-html-with-style-css](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xu5txx35767lsd1905ds.png)

### Create a Single Page App that Renders a Root Component

Cause it's the only thing they ever taught you.

```html
<!-- index.html -->

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0"
    />
    <title>
      ajcwebdev
    </title>
  </head>
  
  <body>
    <div id="app"></div>
    <script type="module" src="/main.js"></script>
  </body>
</html>
```

If we look back at `localhost:3000` we will see our blank canvas. Alternatively known as "the only thing your website does" for people with JavaScript turned off.

![06-blank-root-div](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6lkx1x2wa3vqstyrn6vr.png)

We will paint the canvas with our imperative DOM manipulations just as Elder Resig instructed.

```javascript
// main.js

import './style.css'

document.querySelector('#app').innerHTML = `
  <h1>ajcwebdev</h1>
  <a
    href="https://dev.to/ajcwebdev"
    target="_blank"
  >
    Blog
  </a>
`
```

![07-vanilla-javascript-component](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/oljrpcpxzplhep8041r2.png)

And that's the whole website, that'll be $10,000 dollars.

## Create Vue App

Huh, what's that? You came here expecting a Vue site? What gave you that idea?

It's from the creator of Vue, it starts with a V, and it was used as literally a drop in replacement for the word Vue in [VitePress](https://github.com/vuejs/vitepress). I'm sure that's all just a coincidence.

### Initialize Project

```bash
yarn create @vitejs/app ajcwebdev-vite --template vue
```

Output:

```
success Installed "@vitejs/create-app@2.2.5" with binaries:
      - create-app
      - cva

Scaffolding project in /Users/ajcwebdev/ajcwebdev-vite...

Done. Now run:

  cd ajcwebdev-vite
  yarn
  yarn dev

✨  Done in 2.18s.
```

Start the development server.

```bash
cd ajcwebdev-vite
yarn
yarn dev
```

```
  vite v2.2.4 dev server running at:

  > Local:    http://localhost:3000/
  > Network:  http://10.0.0.175:3000/

  ready in 256ms.
```

![08-create-vite-app](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2cjrtow31cillpopfha1.png)

### Project Structure

```
├── public
│   └── favicon.ico
├── src
│   ├── assets
│   │   └── logo.png
│   ├── components
│   │   └── HelloWorld.vue
│   ├── App.vue
│   └── main.jsx
├── .gitignore
├── index.html
├── package.json
├── vite.config.js
└── yarn.lock
```

Our `package.json` includes scripts for starting the development server, building for production, and serving local previews of production builds.

```json
{
  "name": "ajcwebdev-vite",
  "version": "0.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "serve": "vite preview"
  },
  "dependencies": {
    "vue": "^3.0.5"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^1.2.2",
    "@vue/compiler-sfc": "^3.0.5",
    "vite": "^2.2.3"
  }
}
```

### App Vue Component

```html
<!-- src/App.vue -->

<template>
  <img
    alt="Vue logo"
    src="./assets/logo.png"
  />

  <HelloWorld msg="Hello Vue 3 + Vite" />
</template>

<script setup>
  import HelloWorld from './components/HelloWorld.vue'
</script>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>
```

### HelloWorld Component

```html
<!-- src/components/HelloWorld.vue -->

<template>
  <h1>{{ msg }}</h1>

  <p>
    <a
      href="https://vitejs.dev/guide/features.html"
      target="_blank"
    >
      Vite Documentation
    </a>
    |
    <a
      href="https://v3.vuejs.org/"
      target="_blank"
    >
      Vue 3 Documentation
    </a>
  </p>

  <button @click="state.count++">
    count is: {{ state.count }}
  </button>

  <p>
    Edit
    <code>components/HelloWorld.vue</code> to test hot module replacement.
  </p>
</template>

<script setup>
  import { defineProps, reactive } from 'vue'

  defineProps({
    msg: String
  })

  const state = reactive({ count: 0 })
</script>

<style scoped>
  a {
    color: #42b983;
  }
</style>
```

Modify the components.

```html
<!-- src/components/HelloWorld.vue -->

<template>
  <h1>{{ msg }}</h1>

  <p>
    <a
      href="https://dev.to/ajcwebdev"
      target="_blank"
    >
      Blog
    </a>
    |
    <a
      href="https://github.com/ajcwebdev"
      target="_blank"
    >
      GitHub
    </a>
  </p>
</template>

<script setup>
  import { defineProps } from 'vue'

  defineProps({
    msg: String
  })
</script>

<style scoped>
  a {
    color: #42b983;
  }
</style>
```

```html
<!-- src/App.vue -->

<template>
  <img
    alt="Vue logo"
    src="./assets/logo.png"
  />

  <HelloWorld msg="ajcwebdev" />
</template>

<script setup>
  import HelloWorld from './components/HelloWorld.vue'
</script>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>
```

![09-create-vite-app-edited](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jt1l97olmsnyhy290okz.png)

### Deploy to Netlify

Create a `netlify.toml` file to define the build command and publish directory for the static assets.

```bash
touch netlify.toml
```

```toml
[build]
  publish = "dist"
  command = "yarn build"
```

Connect to your Git provider.

![10-connect-to-Git-provider](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jraetzgxnkoyhvgwrn4b.png)

Select the repository.

![11-pick-a-repository](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fjyjgeyo5uygvkmsy3yr.png)

Go to site settings to create custom domain name.

![12-site-settings](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y4ngh8quj4ds9k1tyvav.png)

Pick a domain name.

![13-create-custom-domain-name](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/exj75pgvrnc4arenk4bi.png)

Check the build time for bragging rights.

```
────────────────────────────────────────────────────────────────
  1. build.command from netlify.toml
────────────────────────────────────────────────────────────────

$ yarn build
yarn run v1.22.4
warning package.json: No license field

$ vite build
vite v2.2.4 building for production...
transforming...
✓ 13 modules transformed.
rendering chunks...
dist/assets/logo.03d6d6da.png    6.69kb
dist/index.html                  0.47kb
dist/assets/index.20c6b699.css   0.20kb / brotli: 0.12kb
dist/assets/index.a12a9eaa.js    1.19kb / brotli: 0.59kb
dist/assets/vendor.de5b410a.js   42.41kb / brotli: 15.34kb

Done in 3.43s.

(build.command completed in 3.7s)

────────────────────────────────────────────────────────────────
  2. Deploy site 
────────────────────────────────────────────────────────────────

Starting to deploy site from 'dist'
Creating deploy tree 
Creating deploy upload records
4 new files to upload
0 new functions to upload
Site deploy was successfully initiated

(Deploy site completed in 332ms)

────────────────────────────────────────────────────────────────
  Netlify Build Complete 
────────────────────────────────────────────────────────────────

(Netlify Build completed in 4.1s)
```

Open [ajcwebdev-vite.netlify.app](https://ajcwebdev-vite.netlify.app) to see the deployed site.

![14-website-deployed-on-netlify](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ivxqxsc3fe62ttgkpmed.png)