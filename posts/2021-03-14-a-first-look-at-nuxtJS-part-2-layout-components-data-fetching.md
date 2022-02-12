---
title: a first look at nuxtJS part 2 - layout, components, data fetching
description: How to create a layout component, auto import components, and fetch data in NuxtJS
date: 2021-03-14
tags:
  - nuxtjs
  - vue
  - javascript
  - components
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/s8vmguv31cxf4ufk0j5d.jpg
layout: layouts/post.njk
---

In the last part we learned how to setup out projects and create pages. In this part we will create a layout for navigation with the `<NuxtLink>` component. We will also configure our application to automatically import components and fetch data with Nuxt's built in `fetch` hook.

## Create layout

```bash
mkdir layouts
touch layouts/default.vue
```

Include the title and footer from `index.vue`. The content of the page will be inserted where the `<Nuxt />` component is placed.

```html
// layouts/default.vue

<template>
  <div class="container">
    <header>
      <h1>ajcwebdev</h1>
    </header>

    <main>
      <Nuxt />
    </main>

    <footer>
      <a
        href="https://dev.to/ajcwebdev"
        target="_blank"
      >
        Blog
      </a>
      <a
        href="https://github.com/ajcwebdev"
        target="_blank"
      >
        GitHub
      </a>
    </footer>
  </div>
</template>

<style>
  .container {
    margin: 0 auto;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-align: center;
  }
</style>
```

Take out the header and footer from the home page.

```html
// pages/index.vue

<template>
  <div class="container">
    <p>This is the home page</p>
  </div>
</template>

<style>
  .container {
    margin: 0 auto;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-align: center;
  }
</style>
```

![01-](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9dg4qfq66db5tjn8q6jm.png)

### Add NuxtLink for home and about pages

Create a navigation bar with links to the home and about pages.

```html
// layouts/default.vue

<template>
  <div class="container">
    <header>
      <h1>ajcwebdev</h1>

      <nav>
        <ul>
          <li>
            <NuxtLink to="/">Home</NuxtLink>
          </li>
          <li>
            <NuxtLink to="/about">About</NuxtLink>
          </li>
        </ul>
      </nav>
    </header>

    <main>
      ...
    </main>

    <footer>
      ...
    </footer>
  </div>
</template>
```

![02-](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/anjqp2loaq0z2ltv898l.png)

## Create Mountains component

The `components` directory is where you put all your components which are then imported into your pages. With Nuxt.js you can create your components and auto import them into your .vue files. This means there is no need to manually import them in the script section.

```bash
mkdir components
touch components/Mountains.vue
```

```html
// components/Mountains.vue

<template>
  <div class="container">
    <h2>Mountains</h2>
  </div>
</template>
```

### Auto import components

Set `components` to `true` in `nuxt.config.js`. Nuxt.js will now scan and auto import your components.

```javascript
// nuxt.config.js

export default {
  target: 'static',
  components: true
}
```

Now any components contained inside the `components` directory can be used on pages without needing to explicitly import the components.

```html
// pages/index.vue

<template>
  <div class="container">
    <p>This is the home page</p>
    <Mountains />
  </div>
</template>

<style>
  ...
</style>
```

![03-](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y6idyerdbmyna7ybxaw1.png)

### Fetch mountains

We will use Nuxt's built in [fetch hook](https://nuxtjs.org/docs/2.x/features/data-fetching/#the-fetch-hook).

```html
// components/Mountains.vue

<template>
  <div class="container">
    <h2>Mountains</h2>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        mountains: []
      }
    },
    async fetch() {
      this.mountains = await fetch(
        'https://api.nuxtjs.dev/mountains'
      )
      .then(res => res.json())
      console.log(this.mountains)
    }
  }
</script>
```

In our console we can see the data we are receiving is an array of seven mountains.

![04-](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gjin8qn9v58jkjn78s3p.png)

### Display mountains

To render the list of mountains, use [`v-for`](https://vuejs.org/v2/guide/list.html) to map over the array and set each list item to `mountain`. Display the `title` with `mountain.title` and set the `key` to `mountains.items` with [`v-bind`](https://vuejs.org/v2/guide/class-and-style.html).

```html
// components/Mountains.vue

<template>
  <div class="container">
    <h2>Mountains</h2>

    <ul>
      <li
        v-for="mountain of mountains"
        v-bind:key="mountain.items"
      >
        {{ mountain.title }}
      </li>
    </ul>
  </div>
</template>

<script>
  ...
</script>
```

![05-](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/x56kcexozzt37ohbbry7.png)

Add error and loading state.

```html
// components/Mountains.vue

<template>
  <div class="container">
    <h2>Mountains</h2>

    <p v-if="$fetchState.pending">
      Almost there...
    </p>

    <p v-else-if="$fetchState.error">
      Error
    </p>

    <div v-else>
      <ul>
        <li
            v-for="mountain of mountains"
            v-bind:key="mountain.items"
          >
          {{ mountain.title }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
  ...
</script>
```

In the next part we will learn how to use route parameters to create dynamic pages.