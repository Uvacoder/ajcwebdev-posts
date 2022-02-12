---
title: a first look at svelteKit part 2 - layout, routes
description: There are two underlying concepts of SvelteKit. Each page of your app is a Svelte component. Pages are created by adding files to the src/routes directory of your project.
date: 2021-04-04
tags:
  - svelte
  - sveltekit
  - vite
  - javascript
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2wpntsr7i17is759nm5h.png
layout: layouts/post.njk
---

There are two underlying concepts of SvelteKit:
* Each page of your app is a Svelte component.
* Pages are created by adding files to the **`src/routes`** directory of your project. Pages are server-rendered so a user's first visit is as fast as possible. A client-side app then takes over.

## about.svelte

We don't need any JavaScript on this page but we'll load it in development for hot module replacement.

### $app/env

`browser` is `true` or `false` depending on whether the app is running in the browser or on the server. `dev` is `true` in development mode, `false` in production.

```html
<script context="module">
  import { browser, dev } from '$app/env';

  export const hydrate = dev;

  export const router = browser;

  export const prerender = true;
</script>
```

If the client-side router is already loaded because we came here from elsewhere in the app then it is used automatically. There's no dynamic data here, so we can prerender and serve it as a static asset in production.

### svelte:head

The `<svelte:head>` element allows you to insert elements inside the `<head>` of your document.

```html
<svelte:head>
  <title>About</title>
</svelte:head>
```

```html
<div class="content">
  <h1>About this app</h1>

  <p>
    This is a <a href="https://kit.svelte.dev">SvelteKit</a> app.
    You can make your own by typing the following
    into your command line and following the prompts:
  </p>

  <pre>npm init svelte@next</pre>

  <p>
    The page you're looking at is purely static HTML,
    with no client-side interactivity needed.
    Because of that, we don't need to load any JavaScript.
    Try viewing the page's source, or opening
    the devtools network panel and reloading.
  </p>

  <p>
    The <a href="/todos">TODOs</a> page illustrates SvelteKit's data loading and form handling.
    Try using it with JavaScript disabled!
  </p>
</div>

<style>
  .content {
    width: 100%;
    max-width: var(--column-width);
    margin: var(--column-margin-top) auto 0 auto;
  }
</style>
```

Enter `http://localhost:3000/about` to see this page.

![01-about-page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ojsvge1m94r2gmmz7s9t.png)

## __layout.svelte

Inside `src/routes` is a layout component that applies to every page.

```html
<script>
  import Header from '$lib/Header/index.svelte';
  import '../app.css';
</script>
```

Links between pages are contained in the `<Header>` component.

```html
<Header />

<main>
  <slot />
</main>

<footer>
  <p>visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to learn SvelteKit</p>
</footer>
```

A `<slot />` tag indicates where a component places its children, so our layout component is the parent of our pages.

## Header

Inside `src/lib/Header/` is an `index.svelte` file.

### $app/stores

Stores are are added to the [context](https://svelte.dev/tutorial/context-api) of your root component and are unique to each request on the server. It is safe to include user-specific data in `page` it is not shared between multiple requests handled by the same server simultaneously.

Because of that, the stores are not free-floating objects: they must be accessed during component initialization. The stores themselves attach to the correct context at the point of subscription. This means you can import and use them directly in components without boilerplate.

```html
<script>
  import { page } from '$app/stores';
  import logo from './svelte-logo.svg';
</script>
```

`page` is a readable store whose value reflects the object passed to `load` functions. It contains:
* `host`
* `path`
* `params`
* `query`

### sveltekit:prefetch

For _dynamic_ routes, such as `src/routes/blog/[slug].svelte`, code splitting is not not enough to ensure fast startup times. To render the blog post we need to fetch the data for it and we can't do that until we know what `slug` is.

Lag caused by the browser waiting for the data can be mitigate by adding a `sveltekit:prefetch` attribute for _prefetching_ the data.

```html
<header>
  <div class="corner">
    <a href="https://kit.svelte.dev">
      <img src={logo} alt="SvelteKit" />
    </a>
  </div>

  <nav>
    <svg viewBox="0 0 2 3" aria-hidden="true">
      <path d="M0,0 L1,2 C1.5,3 1.5,3 2,3 L2,0 Z" />
    </svg>

    <ul>
      <li class:active={$page.path === '/'}>
        <a sveltekit:prefetch href="/">
          Home
        </a>
      </li>

      <li class:active={$page.path === '/about'}>
        <a sveltekit:prefetch href="/about">
          About
        </a>
      </li>

      <li class:active={$page.path === '/todos'}>
        <a sveltekit:prefetch href="/todos">
          Todos
        </a>
      </li>
    </ul>

    <svg viewBox="0 0 2 3" aria-hidden="true">
        <path d="M0,0 L0,3 C0.5,3 0.5,3 1,2 L2,0 Z" />
    </svg>
  </nav>
</header>
```

SvelteKit will run the page's `load` function as soon as the user hovers over the link (on desktop) or touches it (on mobile). This makes the app feel snappier since you aren't waiting for the `click` event to trigger navigation.