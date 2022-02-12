---
title: a first look at nuxtJS part 1 - setup, pages
description: How to setup a NuxtJS project and create pages.
date: 2020-09-04
tags:
  - nuxt
  - vue
  - javascript
  - netlify
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/i2478wmn6ye4sivjwxgw.jpg
layout: layouts/post.njk
---

NuxtJS is a Vue meta-framework created by [Sébastien Chopin in October 2016](https://github.com/nuxt/nuxt.js/commit/0072ed31da6ce39d21046e05898f956cff190390). It is a progressive framework designed for creating modern web applications. It is based on official Vue libraries including Vue Core, Vue Router, and Vuex.

It was inspired by NextJS and much like that React framework it started as a solution for server-side rendering. In recent years though it has expanded to include [static site generation](https://nuxtjs.org/blog/nuxt-static-improvements/) inspired by projects like Gatsby and Gridsome.

NuxtJS automatically generates the vue-router configuration based on your file tree of Vue files inside the pages directory. When you create a .vue file in your pages directory you will have basic routing working with no extra configuration needed.

All the code for this series is available at the [following repo](https://github.com/ajcwebdev/ajcwebdev-nuxt). Each part of the series will have its own branch corresponding to the state of the project at the end of that article.

## Installation

To start we will create a directory, initialize a `package.json` file.

```bash
mkdir ajcwebdev-nuxt
cd ajcwebdev-nuxt
yarn init -y
```

Create the following files:

* `.gitignore`
* `.env`
* `README.md`

```bash
touch .gitignore .env README.md
```

Add the following scripts to `package.json`.

```json
"scripts": {
  "dev": "nuxt",
  "build": "nuxt build",
  "generate": "nuxt generate",
  "start": "nuxt start"
}
```

Add the following to `.gitignore` so we do not commit our dependencies or environment variables.

```
node_modules
.env
.nuxt
.DS_Store
```

To install Nuxt we will enter the following command:

```bash
yarn add nuxt
```

This will install a ton of dependencies. Trust the dependencies. Believe in the dependencies.

If we check our `package.json` file we'll see `nuxt` has been installed with the current version, which as of this writing is `2.15.3`.

```json
"dependencies": {
  "nuxt": "^2.15.3"
}
```

## Create home page

To create our first page we will enter the following command:

```bash
mkdir pages
touch pages/index.vue
```

This creates our pages directory and a file inside the directory called `index.vue`. Open `index.vue` and enter the following Vue code:

```html
// pages/index.vue

<template>
  <h1>ajcwebdev</h1>
</template>
```

## Start development server

```bash
yarn dev
```

Open up `localhost:3000` in a browser.

![01-hello-world](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pfjs6jynwqg971ebat1p.png)

### Fill out Home page

Include an `<h1>` for the title of the page and links to some of your social media accounts.

```html
// pages/index.vue

<template>
  <div class="container">
    <header>
      <h1>ajcwebdev</h1>
    </header>

    <p>This is the home page</p>
    
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
```

![02-home-page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dalh5ys0w9pq0mtx49jm.png)

### Add CSS

```html
// pages/index.vue

<template>
  ...
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

![03-home-page-with-css](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gwfbu5emfia1nf6hinxn.png)

### Create About page

Create another file called `about.vue` for our About page.

```bash
touch pages/about.vue
```

The component includes a `<p>` tag containing a brief description of the page.

```html
// pages/about.vue

<template>
  <div class="container">
    <p>This page tells you about stuff</p>
  </div>
</template>
```

![04-about-page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pl9t9j9or57oh1bd6adn.png)

### Add CSS

```html
// pages/about.vue

<template>
  ...
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

![05-about-page-with-css](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cmuyi1ca27b450pa0c7r.png)

## Deploy site

Create `netlify.toml` for build configuration.

```bash
touch netlify.toml
```

```toml
[build]
  publish = "dist/"
  command = "nuxt generate"
```

### nuxt.config.js

The `nuxt.config.js` file is the single point of configuration for Nuxt.js. If you want to add modules or override default settings, this is the place to apply the changes.

```bash
touch nuxt.config.js
```

Set `target` to `static` to let Nuxt.js know we are deploying our site to a static hosting provider.

```javascript
// nuxt.config.js

export default {
    target: 'static'
}
```

### Initialize git repo

To finish off we will initialize a git repo.

```bash
git init
```

Add all files to staging area, make initial commit, and set origin branch to main.

```bash
git add .
git commit -m "Initial commit"
git branch -M main
```

Create a blank repo on [GitHub](http://github.com/) and add remote.

```bash
git remote add origin https://github.com/ajcwebdev/ajcwebdev-nuxt.git
```

Push to main.

```bash
git push -u origin main
```

Connect the repo to your Netlify account and deploy using the predefined build commands. You can set a custom domain to match your project and repo name. [Your site is now live](https://ajcwebdev-nuxt.netlify.app/).

![06-deployed-website](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/62rj73ogeux3531z32ud.png)

In the next part we will be creating a layout and our first component.