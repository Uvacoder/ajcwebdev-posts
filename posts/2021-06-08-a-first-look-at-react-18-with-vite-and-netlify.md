---
title: a first look at react 18 with vite and netlify
description: Create a boilerplate React app with Vite, upgrade to React 18, and deploy your app to Netlify.
date: 2021-06-08
tags:
  - react 
  - suspense 
  - vite 
  - netlify
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dkjpx2q7x1065dxb6hl0.png
layout: layouts/post.njk
---

When it’s released, [React 18](https://github.com/reactwg/react-18/discussions/4) will include out-of-the-box improvements including:
* [Automatic batching](https://github.com/reactwg/react-18/discussions/21)
* New APIs like [`startTransition`](https://github.com/reactwg/react-18/discussions/41)
* A [new streaming server renderer](https://github.com/reactwg/react-18/discussions/37) with built-in support for `React.lazy`

The React team has also taken a new step by creating the [React 18 Working Group](https://github.com/reactwg/react-18) to provide feedback, ask questions, and collaborate on the release. The Working Group is hosted on [GitHub Discussions](https://github.com/reactwg/react-18/discussions) and is available for the public to read.

## React 18 Working Group

Members of the working group can leave feedback, ask questions, and share ideas. The core team will also use the discussions repo to share their research findings. As the stable release gets closer, any important information will also be posted on the [React blog](https://reactjs.org/blog/all.html/).

Because an initial surge of interest in the Working Group is expected, only invited members will be allowed to create or comment on threads. However, the threads are fully visible to the public, so everyone has access to the same information. The team believes this is a good compromise between creating a productive environment for working group members, while maintaining transparency with the wider community.

No specific release date is scheduled, but the team expects it will take several months of feedback and iteration before React 18 is ready for most production applications.

* Library Alpha: Available today
* Public Beta: At least several months
* Release Candidate (RC): At least several weeks after Beta
* General Availability: At least several weeks after RC

More details about the projected release timeline are [available in the Working Group](https://github.com/reactwg/react-18/discussions/9).

## Create React App with Vite's React Template

```bash
yarn create @vitejs/app ajcwebdev-react18 --template react
```

### Install dependencies

```bash
cd ajcwebdev-react18
yarn
```

### Start development server

```bash
yarn dev
```

![01-create-vite-app](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fdvsb936oo4qlhq2fw12.png)

## Install `react@beta` and `react-dom@beta`

### package.json

If we look in our `package.json` we'll see the following dependencies included from the Vite template.

```json
"dependencies": {
  "react": "^17.0.0",
  "react-dom": "^17.0.0"
},
```

Install `beta` versions.

```bash
yarn add react@beta react-dom@beta
```

Check your `dependencies` for the new versions.

```json
"dependencies": {
  "react": "^18.0.0-beta-96ca8d915-20211115",
  "react-dom": "^18.0.0-beta-96ca8d915-20211115"
},
```

Use `esbuild.jsxInject` to automatically inject JSX helper imports for every file transformed by ESBuild:

```javascript
// vite.config.js

import { defineConfig } from 'vite'
import reactRefresh from '@vitejs/plugin-react-refresh'

export default defineConfig({
  plugins: [reactRefresh()],
  esbuild: {
    jsxInject: `import React from 'react'`
  }
})
```

### main.jsx

```jsx
// src/main.jsx

import ReactDOM from 'react-dom'
import './index.css'
import App from './App'

const root = ReactDOM.createRoot(
  document.getElementById('root')
)

root.render(<App />)
```

### App.jsx

```jsx
// src/App.jsx

import logo from './logo.svg'
import './App.css'

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img
          src={logo}
          className="App-logo"
          alt="logo"
        />

        <p>
          React 18 Deployed on Netlify with Vite
        </p>
      </header>
    </div>
  )
}

export default App
```

## Deploy to Netlify

```bash
touch netlify.toml
```

```toml
[build]
  publish = "dist"
  command = "yarn build"
```

Create a blank GitHub repository at [github.new](https://github.com/new/).

```bash
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/ajcwebdev/ajcwebdev-react18.git
git push -u origin main
```

Connect your GitHub repository to Netlify.

![02-connect-github-repository](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sud29dkfgb9kxcthbm0a.png)

The build commands are included from your `netlify.toml`.

![03-build-commands-auto-imported](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pfdbymv7infchvybsjgh.png)

```
$ yarn build
yarn run v1.22.4
warning package.json: No license field

$ vite build
vite v2.3.7 building for production...
transforming...
✓ 26 modules transformed.
rendering chunks...

dist/assets/favicon.17e50649.svg   1.49kb
dist/assets/logo.ecc203fb.svg      2.61kb
dist/index.html                    0.51kb
dist/assets/index.7cb030a3.js      0.39kb / brotli: 0.20kb
dist/assets/index.0673ce28.css     0.76kb / brotli: 0.40kb
dist/assets/vendor.9aeda92c.js     134.00kb / brotli: 37.26kb

Done in 4.86s.
​
(build.command completed in 5.1s)
```

Set a custom domain.

![04-custom-domain](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/adsfhpjsem8g8wkcm6ap.png)

Go to your [new domain](https://ajcwebdev-react18.netlify.app/).

![05-deployed-website-on-netlify](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sx49sydxcw7q6hgc2tv7.png)

You can find all the code for this article [on my GitHub](https://github.com/ajcwebdev/ajcwebdev-react18).