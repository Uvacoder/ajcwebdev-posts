---
title: a first look at snowpack
description: Snowpack is a lightweight frontend build tool designed as an alternative to heavier, more complex bundlers like webpack or Parcel.
date: 2021-03-03
tags:
  - snowpack
  - esm
  - javascript
  - bundler
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/s8uv9v5ni4abpg6v7n15.png
layout: layouts/post.njk
---

[Snowpack](https://www.snowpack.dev/) is a lightweight frontend build tool designed as an alternative to heavier, more complex bundlers like webpack or Parcel. Snowpack leverages JavaScript's native module system, [ESM](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import).

## Setup

### Create project directory

```bash
mkdir ajcwebdev-snowpack
cd ajcwebdev-snowpack
```

### Initialize package.json and install Snowpack

```bash
npm init -y
npm i -D snowpack@^3.0.0
```

### Add scripts to package.json

```json
"scripts": {
  "start": "snowpack dev",
  "build": "snowpack build",
  "init": "snowpack init"
},
```

### Initialize snowpack configuration file

See all supported [configuration options](https://www.snowpack.dev/reference/configuration).

```bash
npm run init
```

```javascript
/** @type {import("snowpack").SnowpackUserConfig } */

module.exports = {
  mount: { },
  plugins: [ ],
  packageOptions: { },
  devOptions: { },
  buildOptions: { },
};
```

### Create index.html

```bash
touch index.html
```

```html
<!-- index.html -->

<html>
  <body>
    <h1>ajcwebdev</h1>
  </body>
</html>
```

### Start development server

```bash
npm run start
```

This will result in an error:


>Build Result Error: There was a problem with a file build result.
>
>Error: HTML fragment found!
>
>HTML fragments (files not starting with "`<!doctype html>`") are not transformed like full HTML pages. Add the missing doctype, or set `buildOptions.htmlFragments=true` if HTML fragments are expected.

Okay, let's change that:

```html
<!-- index.html -->

<!doctype html>
<html>
  <body>
    <h1>ajcwebdev</h1>
  </body>
</html>
```

Another error:

>Build Result Error: There was a problem with a file build result.
>
>Error: No `<head>` tag found in HTML (this is needed to optimize your app).

Makes sense, let's give it one more try.

```html
<!-- index.html -->

<!doctype html>
<html>
  <head>
      <title>ajcwebdev</title>
  </head>
  
  <body>
    <h1>ajcwebdev</h1>
  </body>
</html>
```

![01-index-html](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/v92ad450zi0rsqg5cy36.png)

### 🚀

## Add Svelte and Svelte plugin for Snowpack

```bash
npm i svelte
npm i -D @snowpack/plugin-svelte
```

### Add plugin-svelte to snowpack config

```javascript
plugins: [
  '@snowpack/plugin-svelte'
],
```

### Create root component

The root component will be called `App.svelte`.

```bash
touch App.svelte
```

You can tell it's a Svelte thing cause it ends in `.svelte`. Svelte is a JavaScript framework for building components but it does not extend the JavaScript language with something like JSX. Instead, it is a superset of HTML.

```html
<!-- App.svelte -->

<div class="App">
  <header class="App-header">
    <a
      class="App-link"
      href="https://svelte.dev"
      target="_blank"
      rel="noopener noreferrer"
    >
      Learn Svelte
    </a>
  </header>
</div>
```

### Create entry point

The entry point for our app will be `index.js`.

```bash
touch index.js
```

Import our `App.svelte` component into `index.js`.

```javascript
// index.js

import App from "./App.svelte";

let app = new App({
  target: document.body,
});

export default app;
```

### Import `index.js` into `index.html`

```html
<!-- index.html -->

<!doctype html>
<html>
  <head>
    <title>ajcwebdev</title>
  </head>
  
  <body>
    <h1>ajcwebdev</h1>
    <script
      type="module"
      src="/index.js"
    >
    </script>
  </body>
</html>
```

![02-App-svelte](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/61dlu9dr2gnfwcu4y7uz.png)

## Deploy to the world

### Add netlify.toml with build command

```bash
touch netlify.toml
```

```toml
[build]
  command = "npm run build"
  publish = "build"
```

### Create gitignore

```bash
touch .gitignore
```

### Add package-lock.json and node_modules to gitignore

```
package-lock.json
node_modules
.DS_Store
```

Create empty repository on GitHub with the name of your project.

### Initialize git repo

```bash
git init
git add .
git commit -m "Initial commit"
```

### Change master branch to main and set remote origin

```bash
git branch -M main
git remote add origin https://github.com/ajcwebdev/ajcwebdev-snowpack.git
```

### Push to main

```bash
git push -u origin main
```

Connect Netlify to [GitHub repo](https://github.com/ajcwebdev/ajcwebdev-snowpack) and set the [domain name](https://ajcwebdev-snowpack.netlify.app/).

![03-netlify-deployment](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7mdmk9lssadbt2c736ki.png)