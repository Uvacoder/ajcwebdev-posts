---
title: a first look at astro
description: Astro is an open source web framework that is influenced by the Islands Architecture and created by Fred K. Schott, Matthew Phillips, Nate Moore, and Drew Powers.
date: 2021-11-27
tags:
  - astro
  - vite
  - partial
  - hydration
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wqe8cweemy2w34307i42.jpeg
layout: layouts/post.njk
---

[Astro](https://astro.build/) is an open source web framework influenced by the [Islands Architecture](https://jasonformat.com/islands-architecture/). It was created by Fred K. Schott, Matthew Phillips, Nate Moore, and Drew Powers as an outgrowth of the work being done simultaneously on Snowpack and Skypack.

## Outline

* [Introduction](#introduction)
  * [Partial Hydration](#partial-hydration)
  * [Client Directives](#client-directives)
* [Set up](#set-up)
  * [Create project and install Astro dependency](#create-project-and-install-astro-dependency)
  * [Add CLI Commands](#add-cli-commands)
  * [Create a Page](#create-a-page)
  * [Start Development Server](#start-development-server)
* [Add Components](#add-components)
  * [Create a Markdown Component](#create-a-markdown-component)
  * [Create a React Component](#create-a-react-component)
  * [Create a Svelte Component](#create-a-svelte-component)
  * [Create a Vue Component](#create-a-vue-component)
  * [Create a GraphQL Data Fetching Component](#create-a-graphql-data-fetching-component)
* [Deploy to Netlify](#deploy-to-netlify)
  * [Create a GitHub Repository](#create-a-github-repository)
  * [Connect GitHub Repository to Netlify](#connect-github-repository-to-netlify)

## Introduction

The framework deserves a fair amount of credit for bringing partial hydration and the concept of "Islands of Interactivity" to the mainstream web development conversation.

### Partial Hydration

I have an [entirely separate, lengthy article](https://dev.to/ajcwebdev/what-is-partial-hydration-and-why-is-everyone-talking-about-it-3k56) about this, but here's the summary. The conversation had been present but on the fringes for well over a decade. The first framework that fully supported these techniques, Marko, was created in 2014 but remained the odd duck out until around 2019.

However, in the last 2 years there has been an influx of frameworks drawing on similar motivations and prior art including Slinkity, Elder.js, îles, and Qwik. Fred K. Schott describes the architecture and goals of Astro in [Introducing Astro: Ship Less JavaScript (June 8, 2021)](https://astro.build/blog/introducing-astro/):

>*Astro works a lot like a static site generator. If you have ever used Eleventy, Hugo, or Jekyll (or even a server-side web framework like Rails, Laravel, or Django) then you should feel right at home with Astro.*
>
>*In Astro, you compose your website using UI components from your favorite JavaScript web framework (React, Svelte, Vue, etc). Astro renders your entire site to static HTML during the build. The result is a fully static website with all JavaScript removed from the final page.*

While there are plenty of frameworks based on [React](https://nextjs.org/docs/advanced-features/static-html-export), [Vue](https://nuxtjs.org/announcements/going-full-static/), and [Svelte](https://sapper.svelte.dev/docs#sapper_export) that let you render components to static HTML during build time, if you want to hydrate these projects on the client then you have to ship an entire bundle of dependencies along with the static HTML. Astro, on the other hand, includes the ability to load just a single component and its dependencies where that component is needed.

### Client Directives

Astro includes five `client:*` directives to hydrate components on the client at runtime. A directive is a component attribute that tells Astro how your component should be rendered.

|Directive|Description|
|---------|-----------|
|`<Component client:load />`|Hydrates the component on page load.|
|`<Component client:idle />`|Hydrates the component as soon as main thread is free.|
|`<Component client:visible />`|Hydrates the component as soon as the element enters the viewport.|
|`<Component client:media={QUERY} />`|Hydrates the component as soon as the browser matches the given media query.|
|`<Component client:only />`|Hydrates the component at page load, similar to `client:load`. The component will be skipped at build time.|

## Set up

This tutorial will build up an Astro project from scratch instead of using any of the starter templates because I believe that is a better way to learn how a framework works, but the [templates](https://github.com/snowpackjs/astro/tree/main/examples) are really fantastic.

All the code for this tutorial can be found [on my GitHub](https://github.com/ajcwebdev/ajcwebdev-astro).

### Create project and install Astro dependency

Create a new directory for your project, `cd` into the project, initialize a `package.json` file, and install the `astro` dependency.

```bash
mkdir ajcwebdev-astro
cd ajcwebdev-astro
yarn init -y
yarn add -D astro
echo 'node_modules\ndist\n.DS_Store' > .gitignore
```

### Add CLI Commands

Add the following scripts to `package.json`.

```json
"scripts": {
  "dev": "astro dev",
  "start": "astro dev",
  "build": "astro build",
  "preview": "astro preview"
},
```

All commands are run from the root of the project.

* `yarn dev` and `yarn start` both start a local development server on `localhost:3000`.
* `yarn build` builds a production site to `./dist`.
* `yarn preview` previews the build locally before deploying.

### Create a Page

Astro looks for `.astro` or `.md` files in the `src/pages` directory. Each page is exposed as a route based on its file name. Static assets such as images can be placed in the `public` directory.

```bash
mkdir -p src/pages public
touch src/pages/index.astro
```

Inside the `src/pages` directory we created an `index.astro` file.

```js
---
// src/pages/index.astro

let title = 'ajcwebdev-astro'
---

<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width">

    <title>{title}</title>
  </head>
  
  <body>
    <main>
      <header>
        <div>
          <h1>ajcwebdev-astro</h1>
        </div>

        <p>Hello! This is an example Astro project by Anthony Campolo (ajcwebdev).</p>
      </header>
    </main>

    <footer>
      <h3>Find me on the internet:</h3>

      <ul>
        <li><a href="https://github.com/ajcwebdev">GitHub</a></li>
        <li><a href="https://twitter.com/ajcwebdev">Twitter</a></li>
        <li><a href="https://dev.to/ajcwebdev">DEV</a></li>
      </ul>
    </footer>
  </body>
</html>
```

### Start Development Server

```bash
yarn dev
```

Open [localhost:3000](https://localhost:3000) to see the home page.

![01-ajcwebdev-astro-home-page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yi4ctdurfq8vgr0a1rkm.png)

## Add Components

We'll create a directory called `components` inside `src` to hold any Astro/React/Vue/Svelte/Preact components. Then we will create three extra directories that will hold `.astro`, `.jsx`, and `.svelte` files for Markdown, React, and Svelte components respectively.

### Create a Markdown Component

The first example will use Markdown and render completely statically. Astro includes a built in [`Markdown` component](https://docs.astro.build/guides/markdown-content/#astros-markdown-component) that can be imported into any `.md` file.

```bash
mkdir -p src/components/markdown
touch src/components/markdown/HelloMarkdown.astro
```

Import the `Markdown` component from `astro/components` and write some Markdown.

```js
---
// src/components/markdown/HelloMarkdown.astro

import { Markdown } from 'astro/components'
---

<article>
  <section>
    <h2>Markdown</h2>

    <Markdown>
      ### This is an h3 with Markdown

      *Pretty* **cool**, ***right***?
    </Markdown>
  </section>
</article>
```

Return to `index.astro` and import `HelloMarkdown` from `'../components/markdown/HelloMarkdown.astro'`. Place `<HelloMarkdown />` inside the `main` tags.

```js
---
// src/pages/index.astro

import HelloMarkdown from '../components/markdown/HelloMarkdown.astro'

let title = 'ajcwebdev-astro'
---

<html lang="en">
  <head>
    ...
  </head>
  
  <body>
    <main>
      <header>
        ...
      </header>

      <HelloMarkdown />
    </main>

    <footer>
      ...
    </footer>
  </body>
</html>
```

![02-home-page-with-markdown-component](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7fcs6yg5gx3e00ezzsfd.png)

### Create a React Component

To configure Astro, add an `astro.config.mjs` file in the root of your project. All settings are optional and you can view the full configuration API (including information about default configuration) on [GitHub](https://github.com/snowpackjs/astro/blob/main/packages/astro/src/%40types/astro.ts).

```bash
touch astro.config.mjs
```

Add the following code to `astro.config.mjs` to enable the React renderer and provide support for React JSX components.

```javascript
// astro.config.mjs

export default ({
  renderers: [
    '@astrojs/renderer-react'
  ],
})
```

We'll create a `react` directory with a `HelloReact.jsx` component inside.

```bash
mkdir src/components/react
touch src/components/react/HelloReact.jsx
```

It's a React component so you're contractually obligated to make it a counter with `useState` that is triggered with an `onClick` event handler on a `<button>`.

```jsx
// src/components/react/HelloReact.jsx

import { useState } from 'react'

export default function HelloReact({ children, count: initialCount }) {
  const [count, setCount] = useState(initialCount)
  
  const add = () => setCount((i) => i + 1)
  const subtract = () => setCount((i) => i - 1)

  return (
    <>
      <div className="children">
        {children}
      </div>

      <div className="counter">
        <button onClick={subtract}>-</button>

        <pre>{count}</pre>

        <button onClick={add}>+</button>
      </div>
    </>
  )
}
```

Importing the `HelloReact` component is much like the `HelloMarkdown` component. However, this time we're including `someProps` in the front matter to set the initial `count` to `0`.

```js
---
// src/pages/index.astro

import HelloMarkdown from '../components/markdown/HelloMarkdown.astro'
import HelloReact from '../components/react/HelloReact.jsx'

const someProps = {
  count: 0,
}

let title = 'ajcwebdev-astro'
---

<html lang="en">
  <head> ... </head>
  
  <body>
    <main>
      <header> ... </header>

      <HelloMarkdown />
      <HelloReact {...someProps} client:visible />
    </main>

    <footer> ... </footer>
  </body>
</html>
```

We also include `client:visible` to hydrate the component as soon as the element enters the viewport.

![03-home-page-with-react-component](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5atsy2uh62gu4nmjk3g4.png)

### Create a Svelte Component

Add `renderer-svelte` to `renderers` in `astro.config.mjs` to enable the Svelte renderer, provide support for Svelte components, and enable you to poorly rewrite SvelteKit in your spare time. Svelte is so cool. Am I allowed to write Svelte at work yet?

```javascript
// astro.config.mjs

export default ({
  renderers: [
    '@astrojs/renderer-react',
    '@astrojs/renderer-svelte'
  ],
})
```

As with React, create a `svelte` directory and `HelloSvelte.svelte` file.

```bash
mkdir src/components/svelte
touch src/components/svelte/HelloSvelte.svelte
```

Our Svelte component will contain the same functionality as our React component.

```html
<!-- src/components/svelte/HelloSvelte.svelte -->

<script>
  let count = 0

  function add() {
    count += 1
  }

  function subtract() {
    count -= 1
  }
</script>

<div class="children">
  <slot />
</div>

<div class="counter">
  <button on:click={subtract}>-</button>

  <pre>{ count }</pre>

  <button on:click={add}>+</button>
</div>
```

![04-svelte-vs-react-meme](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7zahodmf2b84f6sv8drl.jpeg)

Import `HelloSvelte` and set it to `client:visible`.

```js
---
// src/pages/index.astro

import HelloMarkdown from '../components/markdown/HelloMarkdown.astro'
import HelloReact from '../components/react/HelloReact.jsx'
import HelloSvelte from '../components/svelte/HelloSvelte.svelte'

const someProps = {
  count: 0,
}

let title = 'ajcwebdev-astro'
---

<html lang="en">
  <head> ... </head>
  
  <body>
    <main>
      <header> ... </header>

      <HelloMarkdown />
      <HelloReact {...someProps} client:visible />
      <HelloSvelte client:visible />
    </main>

    <footer> ... </footer>
  </body>
</html>
```

![05-home-page-with-svelte-component](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m4nkmv0nokjllzg7h4vn.png)

### Create a Vue Component

Add `renderer-vue` to `renderers` in `astro.config.mjs` to enable the Vue renderer.

```javascript
// astro.config.mjs

export default ({
  renderers: [
    '@astrojs/renderer-react',
    '@astrojs/renderer-svelte',
    '@astrojs/renderer-vue'
  ],
})
```

You know the drill.

```bash
mkdir src/components/vue
touch src/components/vue/HelloVue.vue
```

Our Vue component will contain the same functionality as our React and Svelte components.

```html
<!-- src/components/vue/HelloVue.vue -->

<template>
	<div class="counter">
		<button @click="subtract()">-</button>

		<pre>{{ count }}</pre>

		<button @click="add()">+</button>
	</div>

	<div class="counter-message">
		<slot />
	</div>
</template>

<script>
  import { ref } from 'vue'

  export default {
    setup() {
      const count = ref(0)
      const add = () => (count.value = count.value + 1)
      const subtract = () => (count.value = count.value - 1)

      return {
        count, add, subtract,
      }
    },
  }
</script>
```

Import `HelloVue` and set it to `client:visible`.

```js
---
// src/pages/index.astro

import HelloMarkdown from '../components/markdown/HelloMarkdown.astro'
import HelloReact from '../components/react/HelloReact.jsx'
import HelloSvelte from '../components/svelte/HelloSvelte.svelte'
import HelloVue from '../components/vue/HelloVue.vue'

const someProps = {
  count: 0,
}

let title = 'ajcwebdev-astro'
---

<html lang="en">
  <head>
    ...
  </head>
  
  <body>
    <main>
      <header>
        ...
      </header>

      <HelloMarkdown />
      <HelloReact {...someProps} client:visible />
      <HelloSvelte client:visible />
      <HelloVue client:visible />
    </main>

    <footer>
      ...
    </footer>
  </body>
</html>
```

![06-home-page-with-vue-component]()

### Create a GraphQL Data Fetching Component

```bash
mkdir src/components/graphql
touch src/components/graphql/HelloGraphQL.astro
touch src/pages/graphql.astro
```

```js
---
// src/components/graphql/HelloGraphQL.astro

const res = await fetch('https://rickandmortyapi.com/graphql', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    query: `{
      characters {
        results {
          id
          name
        }
      }
    }`
  })
})

const data = await res.json()
---

<ul>
  {data.data.characters.results.map(({ name, id }) => (
    <li key={id}>
      {name}
    </li>
  ))}
</ul>
```

Import `HelloGraphQL`.

```js
---
// src/pages/graphql.astro

import HelloGraphQL from '../components/graphql/HelloGraphQL.astro'

let title = 'GraphQL'
---

<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width">

    <title>{title}</title>
  </head>
  
  <body>
    <main>
      <HelloGraphQL />
    </main>
  </body>
</html>
```

![07-graphql-page-with-graphql-data-fetching-component]()

## Deploy to Netlify

The Astro docs contain over [a dozen different deployment options](https://docs.astro.build/guides/deploy/). We will deploy our project to Netlify.

Create a `netlify.toml` file for our build configuration.

```bash
touch netlify.toml
```

Add `yarn build` for the build command and `dist` for the publish directory.

```toml
[build]
  command = "yarn build"
  publish = "dist"
```

Here is our final project structure.

```
/
├── public
├── src
│   ├── components
│   │   ├── graphql
│   │   │   └── HelloGraphQL.astro
│   │   ├── markdown
│   │   │   └── HelloMarkdown.astro
│   │   ├── react
│   │   │   └── HelloReact.jsx
│   │   ├── svelte
│   │   │   └── HelloSvelte.svelte
│   │   └── vue
│   │       └── HelloVue.vue
│   └── pages
│       └── index.astro
├── netlify.toml
└── package.json
```

### Create a GitHub Repository

Initialize a git repo and push to a GitHub repository.

```bash
git init
git add .
git commit -m "svuereactroQL"
gh repo create ajcwebdev-astro
git push -u origin main
```

### Connect GitHub Repository to Netlify

Connect your GitHub repository to Netlify.

![08-connect-github-repository-to-netlify](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/26jrzahq9ctl6pxqwz5a.png)

Go to "Site settings," to give yourself a custom domain name.

![09-add-a-custom-domain](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g4nwpzzcox0zg825hed7.png)

Open [ajcwebdev-astro.netlify.app](https://ajcwebdev-astro.netlify.app/) to see the deployed site.

![10-ajcwebdev-astro-deployed-on-netlify](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6097ceqjovj0d4aicfhu.png)

Congratulations, you're now an expert on microfrontends and are allowed to ask your boss for a 25-30% pay bump.