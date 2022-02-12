---
title: a first look at aleph.js
description: Aleph.js is the React Framework in Deno.
date: 2020-11-09
tags:
  - aleph
  - deno
  - nextjs
  - react
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/9jiexmie5rc4gth2f6tw.jpg
layout: layouts/post.njk
---

[Aleph.js](https://alephjs.org/) is a React framework for Deno with support for ES module imports, file-system routing, SSR, SSG, and hot module reloading with Fast Refresh. It takes inspiration from the popular React framework for Node, Next.js. Thankfully I wasn't there to name it or else we would all be calling it Dext.js.

## Install Aleph

```bash
deno install -A -f -n aleph https://deno.land/x/aleph@v0.3.0-alpha.1/cli.ts
export PATH="/Users/<YOUR_USERNAME>/.deno/bin:$PATH"
aleph init ajcwebdev-aleph
```

```
Download https://deno.land/x/aleph@v0.3.0-alpha.1/cli/init.ts
Check https://deno.land/x/aleph@v0.3.0-alpha.1/cli/init.ts

INFO Downloading template...
INFO Saving template...
INFO Done

INFO Aleph.js is ready to Go.

INFO $ cd ajcwebdev-aleph
INFO $ aleph dev    # start the app in `development` mode
INFO $ aleph start  # start the app in `production` mode
INFO $ aleph build  # build the app to a static site (SSG)
```

## Start development server

```bash
cd ajcwebdev-aleph
aleph dev
```

```
INFO Start watching code changes...
INFO Aleph server ready on http://localhost:8080
```

## Open browser to `localhost:8080`

![03](https://dev-to-uploads.s3.amazonaws.com/i/1mymv9lqg2rvibtb6k55.jpg)

## Directory structure

```
├── api
│   └── counter
│       ├── [action].ts
│       └── index.ts
├── components
│   └── logo.tsx
├── lib
│   └── useCounter.ts
├── pages
│   └── index.tsx
├── public
│   └── logo.svg
├── style
│   └── index.css
├── .gitignore
├── app.tsx
└── import_map.json
```

## `index.tsx`

```jsx
import { Import, useDeno } from 'https://deno.land/x/aleph/mod.ts'
import React, { useState } from 'https://esm.sh/react'
import Logo from '../components/logo.tsx'

export default function Home() {
  const [count, setCount] = useState(0)
  const version = useDeno(() => {
    return Deno.version
  })

  return (
    <div className="page">
      <Import from="../style/index.less" />

      <p className="logo">
        <Logo />
      </p>

      <h1>Welcome to use <strong>Aleph.js</strong>!</h1>

      <p className="links">
        <a
          href="https://alephjs.org"
          target="_blank"
        >
          Website
        </a>

        <span>&middot;</span>

        <a
          href="https://alephjs.org/docs/get-started"
          target="_blank"
        >
          Get Started
        </a>

        <span>&middot;</span>

        <a
          href="https://alephjs.org/docs"
          target="_blank"
        >
          Docs
        </a>

        <span>&middot;</span>

        <a
          href="https://github.com/postui/alephjs"
          target="_blank"
        >
          Github
        </a>
      </p>

      <p className="counter">
        <span>Counter:</span>

        <strong>{count}</strong>

        <button onClick={() => setCount(n => n - 1)}>
          -
        </button>

        <button onClick={() => setCount(n => n + 1)}>
          +
        </button>
      </p>

      <p className="copyinfo">
        Built by Aleph.js in Deno v{version.deno}
      </p>
    </div>
  )
}
```

## Edit the home page

```jsx
return (
  <div className="page">
    <Import from="../style/index.less" />

    <p className="logo">
      <Logo />
    </p>

    <h1>ajcwebdev</h1>

    <p className="links">
      <a
        href="https://dev.to/ajcwebdev"
        target="_blank"
      >
        Blog
      </a>

      <span>&middot;</span>

      <a
        href="https://github.com/ajcwebdev"
        target="_blank"
      >
        Github
      </a>
    </p>

    <p className="copyinfo">
      Built by Aleph.js in Deno v{version.deno}
    </p>
  </div>
)
```

![05](https://dev-to-uploads.s3.amazonaws.com/i/zo90fz7qct09k3stprgh.jpg)