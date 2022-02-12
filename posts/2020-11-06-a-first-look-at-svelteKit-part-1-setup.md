---
title: a first look at svelteKit part 1
description: SvelteKit is a serverless first Svelte metaframework for building web applications with filesystem-based routing.
date: 2020-11-06
tags:
  - svelte
  - sveltekit
  - vite
  - sapper
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/silyhpk9so0xweyu4dvc.png
layout: layouts/post.njk
---

Back in July I was [considering building](https://twitter.com/ajcwebdev/status/1278830337930326016) a project with Sapper, which thankfully I never actually started. I say thankfully because, as Rich Harris announced at the recent [Svelte Summit](https://sveltesummit.com/), Sapper 1.0 will be released... never.

Instead the Svelte core team is shifting its efforts to a project known as SvelteKit, as detailed in [What's the deal with SvelteKit?](https://svelte.dev/blog/whats-the-deal-with-sveltekit). It's important to emphasize here that the people building Svelte, Sapper, and SvelteKit are all basically the same people. So really nothing drastic is changing here, it's more of a rebrand and namespace migration. Or is it?

There is also going to be a larger focus on serverless technology, with Svelte now being referred to as a ["serverless-first"](https://www.infoq.com/news/2020/10/svelte-next-serverless-first/) framework. But to me the most significant change by far is the removal of Rollup as a development dependency and its replacement with ~~[Snowpack](https://www.snowpack.dev/)~~ [Vite](https://vitejs.dev/).

SvelteKit is very new, so new it currently exists mostly in the form of the ~~blog post linked at the beginning of this article~~ [a monorepo inside the SvelteJS GitHub organization](https://github.com/sveltejs/kit) and a [website](kit.svelte.dev). But you can download it and start using it, albeit with many, many caveats attached to it.

## Initialize Demo App

```bash
mkdir ajcwebdev-sveltekit
cd ajcwebdev-sveltekit
npm init svelte@next
```

After initializing the project you'll be given a disclaimer.

>*Welcome to SvelteKit!*
>
>*This is beta software; expect bugs and missing features. If you encounter a problem, open an issue on https://github.com/sveltejs/kit/issues if none exists already.*
>
>*Stuck? Visit us at https://svelte.dev/chat*

You'll then be asked a series of questions to configure your application. Feel free to answer based on your own personal use case.

```
✔ Which Svelte app template? › SvelteKit demo app
✔ Use TypeScript? … No
✔ Add ESLint for code linting? … No
✔ Add Prettier for code formatting? … No
```

>*Want to add other parts to your code base? Visit https://github.com/svelte-add/svelte-adders, a community project of commands to add particular functionality to Svelte projects*

For the sake of simplicity in this blog series I answered no for TypeScript, ESLint, and Prettier and I will not include any additional CSS libraries.

### Install dependencies and start development server

```bash
npm i
npm run dev -- --open
```

Open `localhost:3000` to see the project.

![01-hello-world-localhost-3000](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ufn76oho7hr6jd27ih3l.png)

Something might happen if you click the `+` or `-` buttons around `0`, although what that could be no one can possibly know.

If we look at our terminal we'll see the following message:

```
[vite] page reload .svelte/dev/generated/root.svelte
```

I used to have a Snowpack joke here, unfortunately that joke is now deprecated.

### Project Structure

Now we'll look at the code.

```
├── src
│   ├── lib
│   │   ├── header
│   │   │   ├── Header.svelte
│   │   │   └── svelte-logo.svg
│   │   ├── Counter.svelte
│   │   └── form.js
│   ├── routes
│   │   ├── todos
│   │   │   ├── _api.js
│   │   │   ├── [uid].json.js
│   │   │   ├── index.json.js
│   │   │   └── index.svelte
│   │   ├── __layout.svelte
│   │   ├── about.svelte
│   │   └── index.svelte
│   ├── app.css
│   ├── app.html
│   ├── global.d.ts
│   └── hooks.js
├── static
│   ├── favicon.png
│   ├── robots.txt
│   ├── svelte-welcome.png
│   └── svelte-welcome.webp
├── .gitignore
├── .npmrc
├── jsconfig.json
├── package-lock.json
├── package.json
├── README.md
└── svelte.config.js
```

### HTML Entry Point

```html
<!-- src/app.html -->

<!DOCTYPE html>

<html lang="en">
  <head>
    <meta charset="utf-8" />

    <link
      rel="icon"
      href="/favicon.png"
    />

    <meta
      name="viewport"
      content="width=device-width, initial-scale=1"
    />

    %svelte.head%
  </head>

  <body>
    <div id="svelte">
      %svelte.body%
    </div>
  </body>
</html>
```

### Client App Entry Point

A `.svelte` file contains three parts:
* `<script>` for JavaScript
* `<style>` for CSS
* Any markup you want to include with HTML.

```html
<!-- src/routes/index.svelte -->

<script>
  import Counter from '$components/Counter.svelte';
</script>
```

```html
<!-- src/routes/index.svelte -->

<svelte:head>
  <title>Home</title>
</svelte:head>

<section>
  <h1>
    <div class="welcome">
      <picture>
        <source
          srcset="svelte-welcome.webp"
          type="image/webp"
        />
        <img
          src="svelte-welcome.png"
          alt="Welcome"
        />
      </picture>
    </div>

    to your new<br />SvelteKit app
  </h1>

  <h2>
    try editing <strong>src/routes/index.svelte</strong>
  </h2>

  <Counter />
</section>
```

```html
<!-- src/routes/index.svelte -->

<style>
  section {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    flex: 1;
  }

  h1 {
    width: 100%;
  }

  .welcome {
    position: relative;
    width: 100%;
    height: 0;
    padding: 0 0 calc(100% * 495 / 2048) 0;
  }

  .welcome img {
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    display: block;
  }
</style>
```

### Counter Component

Unlike React, Svelte does not have a virtual DOM. Instead, Svelte includes its own system of reactivity to keep the DOM in sync with your application state. This includes responding to events, such as a mouse click. First we initialize a variable `count` to a value of `0`.

```html
<!-- src/lib/Counter.svelte -->

<script>
  import { spring } from 'svelte/motion';

  let count = 0;

  const displayed_count = spring();
  $: displayed_count.set(count);
  $: offset = modulo($displayed_count, 1);

  function modulo(n, m) {
    return ((n % m) + m) % m;
  }
</script>
```

We can listen to any event on an element with the `on:` directive.

```html
<!-- src/lib/Counter.svelte -->

<div class="counter">
  <button
    on:click={() => (count -= 1)}
    aria-label="Decrease the counter by one"
  >
    <svg aria-hidden="true" viewBox="0 0 1 1">
      <path d="M0,0.5 L1,0.5" />
    </svg>
  </button>

  <div class="counter-viewport">
    <div
      class="counter-digits"
      style="transform: translate(0, {100 * offset}%)"
    >
      <strong style="top: -100%" aria-hidden="true">
        {Math.floor($displayed_count + 1)}
      </strong>

      <strong>
        {Math.floor($displayed_count)}
      </strong>
    </div>
  </div>

  <button
    on:click={() => (count += 1)}
    aria-label="Increase the counter by one"
  >
    <svg aria-hidden="true" viewBox="0 0 1 1">
      <path d="M0,0.5 L1,0.5 M0.5,0 L0.5,1" />
    </svg>
  </button>
</div>
```

To test that everything is working make a change to `index.svelte`.

```html
<!-- src/routes/index.svelte -->

<script>
  import Counter from '$components/Counter.svelte';
</script>

<section>
  <h1>ajcwebdev</h1>

  <Counter/>
  <p>
    <a href="https://github.com/ajcwebdev">GitHub</a>
  </p>
  <p>
    <a href="https://twitter.com/ajcwebdev">Twitter</a>
  </p>
  <p>
    <a href="https://dev.to/ajcwebdev">Dev.to</a>
  </p>
</section>
```

![03-home-page-ajcwebdev](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n7pfnihhcz8uz2kxdgq3.png)

### Config

SvelteKit is a rare framework containing configuration that's actually interesting. You may have heard in the past that SvelteKit was using Snowpack. While SvelteKit was heavily developed in tandem with Snowpack, it migrated to Vite in February 2021 about a month before its beta launch.

You can read Rich Harris's reasoning for the switch [here](https://github.com/sveltejs/kit/pull/409):

>*While we had misgivings about Vite 1 (which gave Vue apps preferential treatment, and didn't really support SSR), Vite 2 does a really great job of solving some tricky problems that were previously in SvelteKit's domain, like CSS code-splitting or fixing stack traces during SSR. Since it's Rollup-based, it also means that SvelteKit apps can benefit from the very large ecosystem of Rollup plugins.*

### svelte.config.js

`target` will hydrate the `<div id="svelte">` element in `src/app.html`.

```javascript
// svelte.config.js

/** @type {import('@sveltejs/kit').Config} */

const config = {
  kit: {
    target: '#svelte'
  }
};

export default config;
```

### Official adapters for deployment

Svelte apps are built with [adapters](https://kit.svelte.dev/docs#adapters) for optimizing your project to deploy with different environments.

- [`adapter-begin`](https://github.com/sveltejs/kit/tree/master/packages/adapter-begin) — for [Begin](https://begin.com)
- [`adapter-cloudflare-workers`](https://github.com/sveltejs/kit/tree/master/packages/adapter-cloudflare-workers) — for [Cloudflare Workers](https://developers.cloudflare.com/workers/)
- [`adapter-netlify`](https://github.com/sveltejs/kit/tree/master/packages/adapter-netlify) — for [Netlify](https://netlify.com)
- [`adapter-vercel`](https://github.com/sveltejs/kit/tree/master/packages/adapter-vercel) — for [Vercel](https://vercel.com)

There are also adapters for running your app on a Node server or deployed on a static hosting provider.

- [`adapter-node`](https://github.com/sveltejs/kit/tree/master/packages/adapter-node) — for creating self-contained Node apps
- [`adapter-static`](https://github.com/sveltejs/kit/tree/master/packages/adapter-static) — for prerendering your entire site as a collection of static files

By default, `npm run build` will generate a Node app that you can run with `node build`. To use a different adapter, install it and update `svelte.config.js` accordingly.