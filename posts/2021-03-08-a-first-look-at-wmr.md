---
title: a first look at wmr
description: wmr is an all-in-one development tool for modern web apps.
date: 2021-03-08
tags:
  - wmr
  - preact
  - esm
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/peios1fpzh1xz9vqxlxm.jpg
layout: layouts/post.njk
---

[wmr](https://github.com/preactjs/wmr) is an all-in-one development tool for modern web apps. Since it leverages ESM modules, it only requires an HTML files with `<script type=module>`. It stands for Waldo's My Roommate and anyone who tells you otherwise is wrong.

## Setup project with blank package.json

```bash
mkdir ajcwebdev-wmr
cd ajcwebdev-wmr
touch package.json
```

### Add wmr to devDependencies

```json
{
  "devDependencies": {
    "wmr": "^1.2.0"
  }
}
```

### Install preact dependencies

```bash
yarn add preact preact-iso
```

[preact-iso](https://www.npmjs.com/package/preact-iso) includes Isomorphic async tools for Preact including:
* Lazy-load components using `lazy()` and `<ErrorBoundary>`
* Generate static HTML for your app using `prerender()`, waiting for `lazy()` components and data dependencies.
* Implement async-aware client and server-side routing using `<Router>`

### Add scripts

```json
"scripts": {
  "start": "wmr",
  "build": "wmr build --prerender",
  "serve": "wmr serve"
},
```

## Create a public folder containing index.js and index.html

```bash
mkdir public
touch public/index.js public/index.html 
```

### index.js

```javascript
// public/index.js

import hydrate from 'preact-iso/hydrate'
import Home from './pages/HomePage.js'

export function App() {
  return (
    <>
      <Home />
    </>
  )
}

hydrate(
  <App />
)
```

### `hydrate.js`

`hydrate()` is a thin wrapper around Preact's hydrate() method. It performs hydration when the HTML for the current page includes pre-rendered output from `prerender()`. It falls back to plain rendering in any other cases and checks if it's running in a browser context before attempting any rendering.

### index.html

```html
<!-- public/index.html -->

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />

    <title>ajcwebdev-wmr</title>

    <meta
      name="description"
      content="what's more random"
    />
    <meta
      name="viewport"
      content="width=device-width,initial-scale=1"
    />
	  
    <link
      rel="preload"
      as="script"
      href="/index.js"
      crossorigin
    />
  </head>

  <body>
    <script
      type="module"
      src="/index.js"
    >
    </script>
  </body>
</html>
```

### Create pages directory and HomePage.js

```bash
mkdir public/pages
touch public/pages/HomePage.js
```

### Create Home component

```javascript
// public/pages/HomePage.js

export default function Home() {
  return (
    <>
      <h1>ajcwebdev</h1>
    </>
  )
}
```

```bash
yarn start
```

![01-homepage](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ihw8d5f7sl890l4734du.png)

### Add prerendering

`prerender()` renders a Virtual DOM tree to an HTML string using [preact-render-to-string](https://github.com/preactjs/preact-render-to-string). The difference is that it is asynchronous, and waits for any Promises thrown by components during rendering (Suspense-style) to resolve before returning the HTML.

The Promise returned from `prerender()` resolves to an Object with `html` and `links[]` properties.
* `html` contains pre-rendered static HTML markup
* `links` is an Array of any non-external URL strings found in links on generated page

```js
// public/index.js

export async function prerender(data) {
  const {
    default: prerender
  } = await import('preact-iso/prerender')

  return await prerender(
    <App {...data} />
  )
}
```