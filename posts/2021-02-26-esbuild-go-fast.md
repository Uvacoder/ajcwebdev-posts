---
title: esbuild go fast
description: really fast
date: 2021-02-26
tags:
  - esbuild
  - go
  - fast
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/l1710470ujd3bcd7rf0h.png
layout: layouts/post.njk
---

If you've heard of [esbuild](https://github.com/evanw/esbuild) you've probably seen this picture.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/a7r97v41f7dd3zu3zlmi.png)

Okay, sounds good.

## Setup

### Create root directory and install esbuild

```bash
mkdir ajcwebdev-esbuild
cd ajcwebdev-esbuild
npm i esbuild
```

```
added 7 packages, and audited 8 packages in 1s

found 0 vulnerabilities
```

Rocket emoji

## Create a React app

### Install React, ReactDOM, and create app.jsx

```bash
npm i react react-dom
touch app.jsx
```

### Create a greeting

```jsx
// app.jsx

import * as React from 'react'
import * as Server from 'react-dom/server'

let Greet = () => <h1>Hello, world but really really fast!</h1>

console.log(
  Server.renderToString(
    <Greet />
  )
)
```

### Bundle the file


```bash
./node_modules/.bin/esbuild app.jsx --bundle --outfile=out.js
```

This creates a file called `out.js` containing your code and the React library bundled together. It is no longer dependent on `node_modules`. Think about that for a second.

### Run that sucker

```bash
node out.js
```

```html
<h1 data-reactroot="">
  Hello, world but really really fast!
</h1>
```