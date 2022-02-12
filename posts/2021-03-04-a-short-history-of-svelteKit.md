---
title: a short history of svelteKit
description: An oral history of SvelteKit.
date: 2021-03-04
tags:
  - svelte
  - sveltekit
  - sapper
  - javascript
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/p3nn57r52krvpdieblta.png
layout: layouts/post.njk
---

## Table of Contents

### October

* [@sveltejs/kit on NPM - 0.0.8](https://www.npmjs.com/package/@sveltejs/kit/v/0.0.8) - October 13, 2020
* [Futuristic Web Development](https://www.youtube.com/watch?v=qSfdtmcZ4d0) - October 19, 2020
* [Svelte, a bright future with Snowpack](https://carlosvin.github.io/posts/svelte-next-bright-future-with-snowpack/en/) - October 24, 2020
* [The Next Svelte May Be Serverless-First](https://www.infoq.com/news/2020/10/svelte-next-serverless-first/) - October 28, 2020

### November - December

* [Twitter Announcement](https://twitter.com/Rich_Harris/status/1324360796223262726) - November 5, 2020
* [What's the deal with SvelteKit?](https://svelte.dev/blog/whats-the-deal-with-sveltekit) - November 5, 2020
* [a first look at svelteKit](https://dev.to/ajcwebdev/a-first-look-at-svelte-kit-372h) - November 6, 2020
* [SvelteKit and the future of web development](https://www.svelteradio.com/episodes/rich-harris-talks-sveltekit-and-the-future-of-web-development) - November 12, 2020
* [@sveltejs/kit on NPM - 1.0.0-next.1](https://www.npmjs.com/package/@sveltejs/kit) - December 3, 2020

### January - February

* [Svelte adapter for Netlify (@1.0.0-next.0)](https://github.com/jbmoelker/test-sveltejs-kit/blob/master/node_modules/%40sveltejs/adapter-netlify/index.js) - January 24, 2021
* [Ben McCann on Sapper and SvelteKit](https://www.svelteradio.com/episodes/ben-mccann-on-sapper-and-sveltekit) - January 25, 2021
* [SvelteKit, the first ‘serverless-first’ framework?](https://www.voorhoede.nl/en/blog/svelte-kit-the-first-serverless-first-framework/) - February 2, 2021

## [@sveltejs/kit on NPM - 0.0.8](https://www.npmjs.com/package/@sveltejs/kit/v/0.0.8)

**Rich Harris, Conduitry - October 13, 2020**

>*Here be dragons, etc etc.*
>
>*This is a more fleshed-out version of [snowpack-svelte-ssr](https://github.com/Rich-Harris/snowpack-svelte-ssr) that aims to replicate Sapper's functionality in its entirety, minus building for deployment (which can be handled by 'adapters' that do various opinionated things with the output of `snowpack build`).*
>
>*It's currently missing a ton of stuff but I figured I'd throw it up on GitHub anyway partly for the sake of 'working in the open' but mostly because I need an issue tracker to organise my thoughts.*
>
>*There are no tests yet or anything like that. Some of the code has just been straight copied over from the existing Sapper repo, but a pleasing amount of it can safely be left behind.*
>
>*This [Snowpack] thing is really neat. It has some bugs and missing features, but identifying those is the whole purpose of this experiment, and so far I've been able to MacGyver my way around them.*
>
>*Clone this repo, `npm install`, and `npm link`. That will create a global link to the `svelte` bin. You can then either `npm run build` or `npm run dev`, if you intend to make changes and see them immediately reflected.*
>
>*Then, clone the corresponding [svelte-app-demo](https://github.com/sveltejs/svelte-app-demo) repo and follow the instructions therein.*

## [Futuristic Web Development](https://www.youtube.com/watch?v=qSfdtmcZ4d0)

**Rich Harris - October 19, 2020**

[Futuristic Web Development](https://youtube.com/watch?v=qSfdtmcZ4d0).

>>*Q: Do I have to use this workflow?*
>
>*No, absolutely not. There's this split in framework land between things like Angular and Ember which kind of really want to own the entire experience, even if it's technically possible to use the constituent parts by themselves.*
>
>*Then on the other side you have things like React and Vue which are really just component frameworks and you're expected to build an app yourself, which is why things like Next and Gatsby and Remix exist. I want Svelte to be both of those.*
>
>*You can be widely productive with this app template in a matter of seconds, but if you're a power user with specific needs that aren't met by this template for whatever reason, then those needs will continue to be met by the Svelte project.*
>
>*It is critically important that people are able to build their own custom integrations, plugins for things like Eleventy, or ever Svelte based frameworks like Routify and ElderJS and that is not gonna change.*

## [Svelte, a bright future with Snowpack](https://carlosvin.github.io/posts/svelte-next-bright-future-with-snowpack/en/)

**Carlosvin - October 24, 2020**

>*As you might know, this blog is powered by Sapper. I am already quite happy with it, so when I hear this announcement, I felt like when they cancel a TV Series that I am enjoying.*
>
>*Happily, there is a good reason, there is going to be a better Sapper, I think they will call it SvelteKit, it is solving some issues and improving some aspects of Sapper, but the main benefit, in my opinion, is that you won’t have to choose between Sapper or Svelte when you start a new application, everything will be supported by the Svelte ecosystem.*

## [The Next Svelte May Be Serverless-First](https://www.infoq.com/news/2020/10/svelte-next-serverless-first/)

**Bruno Couriol - October 28, 2020**

>*Harris then demoed `svelte@next`, the Svelte version that is being experimented upon — we will refer to that from now on as Svelte Next. Like Sapper (and Next.js), Svelte Next adopts a file-based routing. Harris created in the `src/routes` directory the components implementing the routes for the demoed application.*
>
>*An `index.svelte` file implements the home page while an `about/index.svelte` implements the `/about` route. Layout components (`$layout.svelte`) may implement commonly occurring parts of pages (e.g., a header or a footer) and keep the pages’ implementation DRY.*

## [Twitter Announcement](https://twitter.com/Rich_Harris/status/1324360796223262726)

**Rich Harris - November 5, 2020**

## [What's the deal with SvelteKit?](https://svelte.dev/blog/whats-the-deal-with-sveltekit)

**Rich Harris - November 5, 2020**

### Snowpack

>*Snowpack is at the vanguard of this movement, and it's what powers SvelteKit. It's astonishingly fast, and has a beautiful development experience (hot module reloading, error overlays and so on), and we've been working closely with the Snowpack team on features like SSR.*
>
>*The hot module reloading is particularly revelatory if you're used to using Sapper with Rollup (which has never had first-class HMR support owing to its architecture, which prioritises the most efficient output).*

### Rollup

>*That's not to say we're abandoning bundlers altogether. It's still essential to optimise your app for production, and SvelteKit uses Rollup to make your apps as fast and lean as they possibly can be (which includes things like extracting styles into static `.css` files).*

### Server rendering

>*The other foundational assumption is that a server-rendered app needs, well, a server. Sapper effectively has two modes:*
>
>* *`sapper build`, which creates a standalone app that has to run on a Node server*
>* *`sapper export` which bakes your app out as a collection of static files suitable for hosting on services like GitHub Pages.*

### Static files and serverless platforms

>*Static files can go pretty much anywhere, but running a Node server (and monitoring/scaling it etc) is less straightforward.*
>
*Nowadays we're witnessing a shift towards serverless platforms, in which you as the app author don't need to think about the server your code is running on, with all the attendant complexity. You can get Sapper apps running on serverless platforms, thanks to things like [vercel-sapper](https://github.com/thgh/vercel-sapper), but it's certainly not what you'd call idiomatic.*
>
>*SvelteKit fully embraces the serverless paradigm, and will launch with support for all the major serverless providers, with an 'adapter' API for targeting any platforms that we don't officially cater to. In addition, we'll be able to do partial pre-rendering, which means that static pages can be generated at build time but dynamic ones get rendered on-demand.*

## [a first look at svelteKit](https://dev.to/ajcwebdev/a-first-look-at-svelte-kit-372h)

**Anthony Campolo - November 6, 2020**

>*The Svelte core team is shifting its efforts to a project known as SvelteKit. It's important to emphasize that the people building Svelte, Sapper, and SvelteKit are all basically the same people. So really nothing drastic is changing here and it's more of a rebrand and namespace migration. Or is it?*
>
>*In addition to the new name, there is also going to be a larger focus on serverless technology with Svelte now being referred to as a "serverless-first" framework. But to me the most significant change by far is the removal of Rollup and its replacement with Snowpack.*
>
>*SvelteKit is very new, so new it currently exists mostly in the form of the blog post linked at the beginning of this article. But you can download it and start using it, albeit with many, many caveats attached to it.*

## [Rich Harris talks SvelteKit and the future of web development](https://www.svelteradio.com/episodes/rich-harris-talks-sveltekit-and-the-future-of-web-development)

**Svelte Radio - November 12, 2020**

>*SvelteKit is, in one way it's a successor to Sapper. And you could even think of it as Sapper 1.0, if you like. But in another larger sense, it's our kind of vision for the way that you should build Svelte apps in future.*
>
>*It’s something that we've been kind of talking about in a peripheral sense for a long time, we've been talking about how we can evolve Sapper to take advantage of some of the recent trends in front end development, particularly the rise of serverless and more recently, the rise of unbundled workflows.*
>
>*But it all sort of came to a head recently, you know, the pace of development on Sapper had hit a bit of a low, at least until Ben McCann really picked up the baton and started churning through issues. And people were getting a little bit frustrated, I think with the progress. And Anthony is one of those people because he uses Sapper very heavily in his job.*
>
>*At a certain point, we're like, “What if we just started from scratch?” Like the big rewrite, as opposed to trying to get all of these ideas into what was honestly kind of a watery codebase. I sort of proposed this very hesitantly in the discord thinking everyone was going to yell at me. And instead everyone was like, “Oh, yeah, let's do that.”*

## [@sveltejs/kit on NPM - 1.0.0-next.1](https://www.npmjs.com/package/@sveltejs/kit)

**Rich Harris, Conduitry - December 3, 2020**

>*Here be dragons, etc etc.*
>
>*This project aims to replicate Sapper's functionality in its entirety, minus building for deployment (which can be handled by 'adapters' that do various opinionated things with the output of `snowpack build`).*

## [Svelte adapter for Netlify (@1.0.0-next.0)](https://github.com/jbmoelker/test-sveltejs-kit/blob/master/node_modules/%40sveltejs/adapter-netlify/index.js)

**Jasper Moelker - January 24, 2021**

### Get netlify configuration, defined by user in netlify.toml

```javascript
module.exports = async function adapter(builder) {
  let netlify_config

  if (fs.existsSync('netlify.toml')) {
    try {
      netlify_config = toml.parse(fs.readFileSync('netlify.toml', 'utf-8'))
    } catch (err) {
      err.message = `Error parsing netlify.toml: ${err.message}`
      throw err
    }
  }

  else {
    throw new Error(
      'Missing a netlify.toml file. Consult https://github.com/sveltejs/kit/tree/master/packages/adapter-netlify#configuration'
    )
  }

  if (!netlify_config.build || !netlify_config.build.publish || !netlify_config.build.functions) {
    throw new Error(
      'You must specify build.publish and build.functions in netlify.toml. Consult https://github.com/sveltejs/adapter-netlify#configuration'
    )
  }

  // ...code blocks listed below

}
```

### Publish directory for static hosting

```javascript
const publish = path.resolve(netlify_config.build.publish)
```

### Functions directory for cloud functions

```javascript
const functions = path.resolve(netlify_config.build.functions)
```

### Copy static and client files to static hosting directory

```javascript
builder.copy_static_files(
  publish
)

builder.copy_client_files(
  publish
)
```

### Copy server files to cloud functions directory

```javascript
builder.copy_server_files(
  `${functions}/render`
)
```

### Copy renderer to cloud functions directory

```javascript
fs.copyFileSync(
  path.resolve(
    __dirname,
    'files/render.js'
  ),
  `${functions}/render/index.js`
)
```

### Catch-all route to serverless render function in _redirects file

```javascript
fs.writeFileSync(
  `${publish}/_redirects`,
  '/* /.netlify/functions/render 200'
)
```

### Prerender

```javascript
builder.log.info(
  'Prerendering static pages...'
)

await builder.prerender({
  dest: publish
})
```

## [Ben McCann on Sapper and SvelteKit](https://www.svelteradio.com/episodes/ben-mccann-on-sapper-and-sveltekit)

**Svelte Radio - January 25, 2021**

>>*Q: On a high level, what is the difference between Sapper and SvelteKit?*

>*The biggest change in my mind is the developer experience. SvelteKit is built on top of Snowpack and esbuild and so you're gonna see compile times be a lot faster. And that's something that's a problem in larger Sapper projects.*
>
>*You don't notice it necessarily when you start out as a new user with Sapper. But when you start to grow your projects, some of the compile times with Sapper can get to be a bit longer. Our hope is to fix all of those issues and make it a really, really smooth experience with SvelteKit.*

## [SvelteKit, the first ‘serverless-first’ framework?](https://www.voorhoede.nl/en/blog/svelte-kit-the-first-serverless-first-framework/)

**Jasper Moelker - February 2, 2021**

>*To ensure Svelte runs seamlessly on all [serverless platforms], Svelte will provide an adapter for each. Each adapter does three things:*
>
>1. *Copy to the serverless hosting directory Svelte’s:*
>  * ***compiled client-side JS bundles***
>  * ***other static files***
>2. *Copy to the serverless functions directory Svelte’s:*
>  * ***server-side JS bundles***
>  * ***platform specific render function***
>3. *Create the redirects configuration with a catch-all route to the serverless render function.*
>
>*The adapter takes platform specific configuration (files) into account.*