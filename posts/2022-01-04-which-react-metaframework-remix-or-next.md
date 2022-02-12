---
title: Which React Meta-framework Should You Use, Remix or Next.js?
description: Either framework is well-suited to a wide range of applications. Your choice of Remix vs. Next will depend largely on which specific features are required for your application. Read on to find out which framework is right for your project.
date: 2022-01-04
tags:
  - remix
  - nextjs
  - react
  - framework
cover_image: https://images.prismic.io/wroom/4b16dab9-0030-439f-8955-be012842a615_RemixvsNext+%281%29.png
layout: layouts/post.njk
---

There are many frameworks built on top of React today, including Gatsby, Next.js, Remix, Redwood, Blitz, and others. Next.js has become one of the most popular frameworks in the world thanks to its focus on performance, developer experience, and tight integration with the deployment platform from the same creators, Vercel.

But recently Remix has become heavily discussed and compared to Next.js as an alternative. Many developers are already evaluating Next.js as a potential tool to use for building their applications. Now that Remix is being presented as another option, they will need to know why they would want to pick one over the other. This leads to a laundry list of questions.

What features do they have? What is the developer experience like when using the frameworks? Is one more performant than the other? How do they integrate with other tools? What deployment platforms are they compatible with? In this article, we will explore the similarities and differences between the two frameworks.

## Background for comparing Remix and Next.js

Next.js is an open-source framework created in 2016 and maintained by the team at Vercel, a popular deployment and hosting provider. The company has been on a steep, upward trajectory over the last few years and recently raised a [$150 million](https://vercel.com/blog/vercel-funding-series-d-and-valuation) Series D.

Remix was originally released by Ryan Florence and Michael Jackson under a paid license. They are well known in the React ecosystem as maintainers of React Router and the educators behind React Training. They announced Remix in April 2020, released a paid "supporter preview" in October 2020, and then fully open-sourced the project in November 2021.

Remix received a [$3 million seed round](https://remix.run/blog/seed-funding-for-remix) shortly before fully open-sourcing the project. Notable React developer and teacher, Kent C. Dodds was hired as an early employee of the newly formed company. Kent has been [rebuilding his own platform with Remix](https://kentcdodds.com/blog/why-i-love-remix) while documenting the process.

## Meta-frameworks built on React + Router

To begin, we'll discuss the similarities between the two frameworks. Fundamentally, they are both based on React, although Remix currently plans to [add support for other frameworks](https://twitter.com/remix_run/status/1471116657867333638), such as Vue and Svelte. But as of now, you will be writing React code if you use Remix or Next.js since they take the core React library and add their own routers.

They each use the file system to define routes, commonly referred to as "file-system-based routing." Both allow the ability to declare dynamic routes and have their own specific syntax to accomplish them. Remix comes from the creators of React Router and thus builds heavily on that library, specifically React Router v6. Next.js has implemented its own router.

The main advantage React Router (and thus Remix) has over the Next.js router is that it enables nested routing with nested layouts. With Next.js, if you want to add nested layouts you need to render the layout on each page manually and add it from the `_app` page with custom logic. However, this technique is a documented workaround that has certain limitations when nesting is deeper than two levels.

## Web standard APIs vs. Node.js APIs

Remix is built on top of standard Web APIs while Next.js is built on Node APIs. With Remix you won't have to learn as many additional abstractions over the web platform. You will likely learn concepts that will be useful no matter what tool you decide to use later.

In addition to the APIs, the core of Remix doesn't rely on Node dependencies. Since Remix doesn’t rely on Node, it is more portable to non-Node environments such as Cloudflare Workers and Deno Deploy (discussed more in the deployment section).

This will allow you to run Remix on the edge more easily. Running on the edge means the servers hosting your applications are distributed around the world instead of being centralized in a single physical location. When a user visits the site they are routed to the data center closest to them for extremely fast response times.

## Static site generation and server-side rendering

Next.js supports both static site generation (SSG) and server-side rendering (SSR). With Next.js you can achieve SSG [using the `getStaticProps` method to fetch data at build time](https://prismic.io/blog/making-fetch-happen-different-methods-for-data-fetching) or the `getStaticPaths` method to specify dynamic routes to pre-render pages based on data. `getServerSideProps` can be used to fetch data on each request for server-side rendering.

Remix only supports SSR and doesn't support SSG. This may be surprising to developers raised on the Jamstack. Technically you could run your Remix app through Puppeteer to pre-render the HTML or use the Cache-Control HTTP header to cache the HTML in a CDN.

You may have also heard of Next.js's incremental static regeneration (ISR) which allows you to achieve [stale while revalidating](https://web.dev/stale-while-revalidate/). Responses can be cached and served, while the cache is fresh and requests for updated data will receive a stale response while the server attempts to revalidate. The cache is refreshed after revalidating.

Remix doesn't support ISR because you can use the [Cache-Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) HTTP header to approximate similar behavior, which requires more manual effort. If you are concerned with very fast Time to First Byte and want the framework to take care of the basic rules of caching automatically, you will want to consider the static option with Next.js.

## Data fetching

Both Remix and Next.js provide built-in mechanisms for data fetching. Next.js includes the previously discussed methods (`getStaticProps`, `getStaticPaths`, `getServerSideProps`) to perform [a variety of different kinds of data fetching](https://prismic.io/blog/making-fetch-happen-different-methods-for-data-fetching) depending on whether you want to fetch at runtime, build time, on the client, or on the server. Objects returned from `getStaticProps` or `getServerSideProps` are injected as props on the page component. With Remix you can only access loader data by using the proper hook.

Remix has a built-in `loader` convention and a `useLoaderData` hook that returns the JSON parsed data from your route loader function. Objects you return from a loader will automatically be converted to a Fetch Response object that can be returned as well. This can be used to modify headers, which is especially useful for caching.

Remix adds some syntactic sugar compared to Next.js, but they achieve similar functionality for data fetching with slightly different implementations. Preference for one versus the other will mostly come down to a matter of taste.

## Form handling

Next.js uses JavaScript to communicate with the server from the client. An `onSubmit` handler is used to `POST` form data to an API route on the server. This is a common approach with other single-page applications and requires the developer to write boilerplate JavaScript code to achieve the base functionality. You can also use a library such as [react-hook-form](https://react-hook-form.com/).

Remix, on the other hand, relies on the browser's native HTML form element. Since Remix comes with a notion of a server by default (which could be an actual persistent server or a serverless function), it also includes a PHP-style, server-side `POST` handler. This means that your Remix form will function without the need for any JavaScript. A user could even have JavaScript turned off and they will still be able to use the website.

## Database access

Both Remix and Next.js provide options for including libraries for connecting to databases but do not have native solutions. Remix and Next.js have each leaned heavily on [Prisma](https://www.prisma.io/) as their ORM of choice in their documentation and example applications:

- [How to Build a Fullstack App with Next.js, Prisma, and PostgreSQL](https://vercel.com/guides/nextjs-prisma-postgres)
- [Remix Jokes App](https://remix.run/docs/en/v1/tutorials/jokes#database)

## Auth

Both provide options for including libraries for authentication and authorization but do not have native solutions. Third-party solutions have been created for each including [`next-auth`](https://github.com/nextauthjs/next-auth) and [`remix-auth`](https://github.com/sergiodxa/remix-auth). In their documentation you can also find examples to get started if you want to implement your own authentication:

- [Next.js Authentication](https://nextjs.org/docs/authentication)
- [Remix Authentication](https://remix.run/docs/en/v1/tutorials/jokes#authentication)

If you don't want to go with a token-based approach, Remix also comes with built-in support for cookies, which can make it easier to get sessions.

## Deployment

Both Remix and Next.js provide a wide range of deployment options with various trade-offs, including serverless function-based deployment and traditional server-based deployment. The available deployment options are a testament to the increasingly powerful partnership between open-source frameworks and deployment providers.

A few years ago, most frameworks only included built-in support for one or two providers, if they provided built-in support at all. I think it is very likely that the deployment options for both will increase as time goes on since more companies are currently being created to focus on different hosting niches around serverless, containers, virtual machines, and edge computing.

Remix currently supports Architect, Cloudflare Workers, [Fly.io](http://fly.io/), Netlify, and Vercel. This allows you to run Remix with serverless functions, containers, and VMs. It does not depend on a Node environment because of its use of web standard APIs.

Next.js supports Vercel, Netlify, and any hosting provider that supports a Node or Docker environment.

Regardless of your application, your users, or your requirements, you will almost certainly find a good deployment fit for either framework.

## Miscellaneous extra features

Since Remix is new it falls short on a few extra features that Next.js includes, such as an image component, automatic font CSS inlining, and internationalized routing. Both include conventions for adding links and meta information to your page's head. They also both include the ability to add error handling into your application.

## Should you use Remix or Next?

Remix includes many improvements in developer experience through their new abstractions and user experience by shipping less JavaScript. However, Next.js has been in development significantly longer, has a bigger community of users, and has more resources dedicated to its development from the team at Vercel.

As with most new technology, Remix is being used for lots of personal projects and toy applications, with the notable exception of Kent's website. Next.js, on the other hand, is being used in a large number of [production applications](https://nextjs.org/showcase).

Here is a recap of the different categories that we have discussed. For some categories, one framework comes out on top, while in others it is essentially equal. These comparisons are not an exact science and represent only their writer’s humble opinion based on the points previously presented.

|Category|Next.js|Remix|
|--------|-------|-----|
|Meta-frameworks built on React + Router|❌|✅|
|Web standard APIs vs. Node.js APIs|❌|✅|
|Static site generation and server-side rendering|✅|❌|
|Data fetching|=|=|
|Form handling|❌|✅|
|Database access|=|=|
|Auth|=|=|
|Deployment|=|=|
|Miscellaneous extra features|✅|❌|
|Production use|✅|❌|

Either framework is well-suited to a wide range of applications that fall along similar use cases including blogs, e-commerce, and data-intensive dashboards. Your choice of one versus the other will depend largely on which specific features are required for your application, which style of development is preferred by the developers on your team, and how much tolerance you have for newly created technology.

*Special thanks to [Martin Bavio](https://twitter.com/marbiano3) for providing feedback and technical consultation on this article.*