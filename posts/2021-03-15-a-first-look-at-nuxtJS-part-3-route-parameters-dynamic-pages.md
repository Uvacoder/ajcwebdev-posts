---
title: a first look at nuxtJS part 3 - route parameters, dynamic pages
description: How to query the Dad jokes API with dynamic routes
date: 2021-03-15
tags:
  - nuxt
  - vue
  - javascript
  - axios
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5vw93p3zlkliyqcqdthf.jpg
layout: layouts/post.njk
---

In the last part we created a layout for navigation with the `<NuxtLink>` component. We also created a `Mountain` component to fetch data with Nuxt's built in `fetch` hook. In this part, we will use the [`@nuxtjs/axios`](https://axios.nuxtjs.org/) module to query the [Dad Jokes API](https://icanhazdadjoke.com/) and create [dynamic pages](https://nuxtjs.org/docs/2.x/directory-structure/pages#dynamic-pages) with the Joke `id`.

### Install `@nuxtjs/axios` dependency

```bash
yarn add @nuxtjs/axios
```

### Add `@nuxtjs/axios` to `modules` in config file

```javascript
// nuxt.config.js

export default {
  target: 'static',
  components: true,
  modules: [
    '@nuxtjs/axios',
  ]
}
```

### Jokes page

Create a `Jokes` directory with an `index.vue` file.

```bash
mkdir pages/Jokes
touch pages/Jokes/index.vue
```

Include an `<h2>` tag as a placeholder.

```html
// pages/Jokes/index.vue

<template>
  <div>
    <h2>Jokes</h2>
  </div>
</template>
```

Enter `localhost:3000/jokes` to navigate to the new page.

![01-jokes-page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9bw6i69wr665twbcl85u.png)

Import `axios` and make a `get` request to `https://icanhazdadjoke.com/search`.

```html
// pages/Jokes/index.vue

<template>
  ...
</template>

<script>
import axios from "axios"

export default {
  async created() {
    const config = {
      headers: {
        Accept: "application/json"
      }
    }

    try {
      const res = await axios.get(
        "https://icanhazdadjoke.com/search",
        config
      )
      console.log(res)
    } catch (err) {
      console.log(err)
    }
  }
}
</script>
```

If we check the network tab in our browser we can see the response.

```json
{
  "current_page":1,
  "limit":20,"next_page":2,
  "previous_page":1,
  "results":
  [
    {
      "id":"0189hNRf2g",
      "joke":"I'm tired of following my dreams. I'm just going to ask them where they are going and meet up with them later."
    },
    { ... }
  ],
  "search_term":"",
  "status":200,
  "total_jokes":649,
  "total_pages":33
}
```

To map over the list of jokes we will need to set `res.data.results` to `this.jokes`.

```html
// pages/Jokes/index.vue

<template>
  ...
</template>

<script>
import axios from "axios"

export default {
  data() {
    return {
      jokes: []
    }
  },

  async created() {
    const config = {
      headers: {
        Accept: "application/json"
      }
    }

    try {
      const res = await axios.get(
        "https://icanhazdadjoke.com/search",
        config
      )

      this.jokes = res.data.results
      console.log(this.jokes)
    } catch (err) {
      console.log(err)
    }
  }
}
</script>
```

If we `console.log(this.jokes)` we get the array of jokes.

![02-this-jokes](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o00q93genehz2s0jr73y.png)

### Joke component

Create `Joke` component.

```bash
touch components/Joke.vue
```

```html
// components/Joke.vue

<template>
  <div class="joke">
    <p>{{ joke }}</p>
  </div>
</template>

<script>
export default {
  name: "Joke",
  props: [
    "joke",
    "id"
  ]
}
</script>

<style>
.joke {
  padding: .75rem;
  border: 1px solid black;
  border-radius: .5rem;
  margin: .75rem 0;
  font-size: 1.25rem;
}
</style>
```

### Import `Joke` component into Joke page

```html
// pages/Jokes/index.vue

<template>
  <div>
    <h2>Jokes</h2>

    <Joke
      v-for="joke in jokes"
      :key="joke.id"
      :id="joke.id"
      :joke="joke.joke"
    />
  </div>
</template>

<script>
  ...
</script>
```

![03-joke-component](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dl6zpzuvw9hrz7j1ynxr.png)

### Joke page

Dynamic pages can be created when you don't know the name of the page due to it coming from an API or you don't want to create the same page over and over. A dynamic page can be created with an underscore before:
* The `.vue` file name
* The name of the directory

Create a dynamic directory inside `pages/Jokes` called `_id`.

```bash
mkdir pages/Jokes/_id
touch pages/Jokes/_id/index.vue
```

After defining a file named `_slug.vue` in your pages folder, you can access the value using the context with `params.slug`, which in our case is `params.id`.

{% raw %}
```html
// pages/Jokes/_id/index.vue

<template>
  <div>
    <NuxtLink to="/jokes">
      Back To Jokes
    </NuxtLink>

    <h2>
      {{ joke }}
    </h2>

    <small>
      ID: {{ $route.params.id }}
    </small>
  </div>
</template>

<script>
import axios from "axios"

export default {
  data() {
    return {
      joke: {}
    }
  },
  async created() {
    const config = {
      headers: {
        Accept: "application/json"
      }
    }

    try {
      const res = await axios.get(
        `https://icanhazdadjoke.com/j/${this.$route.params.id}`,
        config
      )

      this.joke = res.data.joke
    } catch (err) {
      console.log(err)
    }
  }
}
</script>
```
{% endraw %}

If we use an `id` from earlier in the article (`0189hNRf2g`) we can visit a dynamic page.

![04-dynamic-page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bgko6u6r77kj4txkurvy.png)

### Add `<NuxtLink>` to link to dynamic pages

We can link to pages for any joke by including `<NuxtLink :to="'jokes/' + id">` in our `Joke` component.

```html
// components/Joke.vue

<template>
  <NuxtLink :to="'jokes/' + id">
    <div class="joke">
      <p>{{ joke }}</p>
    </div>
  </NuxtLink>
</template>

<script>
  ...
</script>

<style>
  ...
</style>
```

![05-link-to-dynamic-page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/j9be0jc04oe8zc4fgfb7.png)

### Add Jokes route to layout

Lastly, add a link to the `/jokes` route in the navigation bar.

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
          <li>
            <NuxtLink to="/jokes">Jokes</NuxtLink>
          </li>
        </ul>
      </nav>
    </header>

    <main>
      <Nuxt />
    </main>

    <footer>
      ...
    </footer>
  </div>
</template>

<style>
  ...
</style>
```

Now we can navigate to the `/jokes` route from any page.

![06-jokes-link-in-navbar](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5vqdc764cc195m6aity3.png)