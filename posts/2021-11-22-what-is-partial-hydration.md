---
title: what is partial hydration and why is everyone talking about it?
description: Hydration uses client-side JavaScript to convert static HTML pages into dynamic web pages. Partial hydration aims to hydrate only the components of an application that need to be interactive.
date: 2021-11-22
tags:
  - partial
  - hydration
  - islands
  - rendering
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o5ogt1tmk4mwgnty4g8z.jpg
layout: layouts/post.njk
---

In [The Cost Of Client-side Rehydration (February 8, 2019)](https://addyosmani.com/blog/rehydration/), Addy Osmani argued that:

>*Server rendering a page and then rehydrating DOM client-side through a serialized version of a UI's dependencies (rehydration) can come with a real cost. It can heavily delay Time To Interactive by making UI look ready before client-side processing has completed.*

He wondered if the benefits of rehydration were outweighed by the Uncanny Valley that is created for users by painting pixels early. To illustrate this point, he created the following graphic:

![01-rehydration-uncanny-valley](https://addyosmani.com/assets/images/hydrate@2x.png)

## Outline

* [To Hydrate or Not to Hydrate](#to-hydrate-or-not-to-hydrate)
  * [Partial Hydration and Progressive Rehydration](#partial-hydration-and-progressive-rehydration)
  * [Islands of Interactivity](#islands-of-interactivity)
* [Adding Partial Hydration to Existing Frameworks](#adding-partial-hydration-to-existing-frameworks)
  * [React](#react)
  * [Preact](#preact)
  * [Vue](#vue)
  * [Solid](#solid)
  * [Svelte](#svelte)
* [Frameworks Built for Partial Hydration](#frameworks-built-for-partial-hydration)
  * [Marko](#marko)
  * [Elder.js](#elderjs)
  * [Astro](#astro)
  * [Slinkity](#slinkity)
  * [îles](#îles)
  * [Qwik](#qwik)
* [Conclusion](#conclusion)

## To Hydrate or Not to Hydrate

Hydration (or rehydration) is a technique that uses client-side JavaScript to convert static HTML pages into dynamic web pages by attaching event handlers to the HTML elements. The JavaScript can be delivered either through static hosting or server-side rendering.

### Partial Hydration and Progressive Rehydration

Partial rehydration is an extension of the idea of progressive rehydration where individual pieces of a server-rendered application are “booted up” over time. This contrasts with the approach of initializing the entire application at once. In an attempt to systematically define and compare the different commonly used rendering and hydration techniques, Addy Osmani and Jason Miller published [Rendering on the Web (February 6, 2019)](https://developers.google.com/web/updates/2019/02/rendering-on-the-web):

>*Partial rehydration has proven difficult to implement. This approach is an extension of the idea of progressive rehydration, where the individual pieces (components / views / trees) to be progressively rehydrated are analyzed and those with little interactivity or no reactivity are identified.*
>
>*For each of these mostly-static parts, the corresponding JavaScript code is then transformed into inert references and decorative functionality, reducing their client-side footprint to near-zero.*

Sounds great! But like all things in tech, nothing is a silver bullet (especially since silver bullets only work on [werewolf shaped problems](https://twitter.com/threepointone/status/1459574071465762820) anyway).

>*The partial hydration approach comes with its own issues and compromises. It poses some interesting challenges for caching, and client-side navigation means we can't assume server-rendered HTML for inert parts of the application will be available without a full page load.*

To summarize the points of the article, the authors included this behemoth of a comparison table:

![02-comparison-chart-of-different-rendering-techniques](https://developers.google.com/web/updates/images/2019/02/rendering-on-the-web/infographic.png)

But this table isn't the whole story, as Jason Miller noted on [Twitter](https://twitter.com/_developit/status/1093256348488396800):

>*One thing I want to clarify: the diagram in this tweet doesn't tell the full story, and doesn't show SSR+Hydration with possible optimizations like component caching applied.*

To expand on this view point, [Mike Sherov](https://twitter.com/mikesherov/status/1093842567173718016) argued:

>*The chart talks about Time to First Byte and Time to Interactive relative to First Contentful Paint, but not First Contentful Paint relative to Time to First Byte... so it misses out on that positive fact that SSR solutions will render faster. The chart assumes that “SSR with (re)hydration” doesn’t employ a “JavaScript as progressive enhancement”... that is, it assumes the page is only functional after all JavaScript is delivered*
>
>*Next.js and others encourage JavaScript as progressive enhancement so that Time to Interactive equals First Contentful Paint still. As mentioned in the disclaimer tweet, this ignores CDN caching of SSR HTML, but ignoring it significantly changes the value prop of “SSR with JavaScript as progressive enhancement”. CDN caching of HTML is flexible AND has high Time to First Byte.*

### Islands of Interactivity

While continuing to explore this problem space, Jason Miller coined the term [Islands Architecture (August 11, 2020)](https://jasonformat.com/islands-architecture/) in reference to the "Component Islands" pattern advocated by Katie Sylor-Miller. He believed web developers should aim to create "islands of interactivity" with JavaScript carefully selected for inclusion only when necessary.

>*In an "islands" model, server rendering is not a bolt-on optimization aimed at improving SEO or UX. Instead, it is a fundamental part of how pages are delivered to the browser. The HTML returned in response to navigation contains a meaningful and immediately renderable representation of the content the user requested.*

Does this mean that partial hydration and the Island's Architecture are interchangeable, or that partial hydration is an implementation of the Island's Architecture? Not exactly. According to Ryan Carniato in [Is 0kb of JavaScript in your Future? (May 3, 2021)](https://dev.to/this-is-learning/is-0kb-of-javascript-in-your-future-48og), partial hydration is a lot like the Island's architecture because the end result is Islands of interactivity but the difference is the authoring experience.

>*Instead of authoring a static layer and an Island's layer, you write your code like a single app more like a typical frontend framework. Partial hydration automatically can create the islands for you to ship the minimal code to the browser. Marko is a JavaScript library that uses its compiler to automate this partial hydration process to remove Server only rendered components from the browser bundle.*

## Adding Partial Hydration to Existing Frameworks

Despite the growing awareness throughout 2019 and 2020 of the difficulties of hydration and the need for some form of partial hydration, we can find discussions of the topic much earlier. Paul Lewis described three different levels of hydration (which he called "booting models") in his blog post, [When everything's important, nothing is! (December 10, 2016)](https://aerotwist.com/blog/when-everything-is-important-nothing-is/).

![03-three-modes-of-partial-hydration-javascript-based-server-render-plus-hydrate-and-progressive-render-plus-bootstrap](https://miro.medium.com/max/2000/0*KhPIdw8fgMb8uHRb)

Early attempts at this were made by [Angular](https://github.com/angular/angular/issues/13446) and [Ember](https://github.com/ember-fastboot/ember-cli-fastboot#ember-fastboot). These attempts appear to have struggled to gain traction. The relevant Angular issue is currently still open 5 years later and Brian Cardarella argued in [Should you use Ember FastBoot or not? (August 1, 2017)](https://dockyard.com/blog/2017/08/01/should-you-use-ember-fastboot-or-not-part-1) that the costs were too high for DockYard to implement FastBoot.

### React

The release of [React v16.0](https://reactjs.org/blog/2017/09/26/react-v16.0.html) in September 2017 introduced the [`hydrate()`](https://reactjs.org/docs/react-dom.html#hydrate) method as an alternative to the `render()` method. According to Andrew Clark:

>*React 16 is better at hydrating server-rendered HTML once it reaches the client. It no longer requires the initial render to exactly match the result from the server. Instead, it will attempt to reuse as much of the existing DOM as possible. No more checksums!*

`hydrate()` behaves similarly to the `render()` method. However, instead of rendering a React element into the DOM in the supplied container and returning a reference to the component, `hydrate()` can hydrate a container whose HTML contents are rendered by `ReactDOMServer` and attempt to attach event listeners to the existing markup.

To balance this trade off, developers were advised to use `render()` for rendering the content solely on the client side and `hydrate()` for rendering on top of server-side rendered markup. Much like the Google team, the React team urged caution when deciding whether to use hydration:

>*In general, we don’t recommend that you render different content on the client versus the server, but it can be useful in some cases (e.g. timestamps). However, it’s dangerous to have missing nodes on the server render as this might cause sibling nodes to be created with incorrect attributes.*

Two years later, Sebastian Markbåge opened a PR to implement [Partial Hydration (January 28, 2019)](https://github.com/facebook/react/pull/14717) as a native feature in React:

>*Partial hydration adds a mechanism for partially hydrating a server rendered result while other parts of the page are still loading the code or data. This means that you can start interacting with parts of the screen while others are still hydrating. In this model you always have to hydrate the root content first because it is what provides props to the children, which can be of arbitrary complexity.*
>
>*The model assumes that the root of the app is designed to be relatively shallow and then each abstraction gets progressively more complex the deeper it gets. To become interactive faster, components in the tree can themselves use progressive enhancement to add more complexity after initial hydration.*

### Preact

Markus Oberlehner foreshadows Slinkity (discussed more in a later section) by explaining how to combine the static site generator Eleventy with Preact in [Building Partially Hydrated, Progressively Enhanced Static Websites with Isomorphic Preact and Eleventy (March 22, 2020)](https://markus.oberlehner.net/blog/building-partially-hydrated-progressively-enhanced-static-websites-with-isomorphic-preact-and-eleventy/):

>*What if I tell you that we can have it all? We can use a modern JavaScript framework, at least as powerful as React, combine it with an exceptional static site generator, and build our websites in a way that they offer real progressive enhancement and a minimal JavaScript bundle size. Combining Eleventy with Preact makes this possible.*

### Vue

Based on his work porting [vue-lazy-hydration](https://github.com/maoberlehner/vue-lazy-hydration) to Vue 3, Markus Oberlehner compares different forms of partial hydration in [Partial Hydration Concepts: Lazy and Active (November 8, 2020)](https://markus.oberlehner.net/blog/partial-hydration-concepts-lazy-and-active/):

>*Lazy Hydration is a form of Partial Hydration where you can trigger hydration at a later point and not immediately after loading the site. A good example is components outside of the viewport. You don’t need to hydrate them instantly, but you can delay hydration until the component is visible.*

```html
<template>
  <TheNavigation/>

  <LazyHydrate skip>
    <TheBlogArticle/>
  </LazyHydrate>

  <LazyHydrate when-visible>
    <TheFooter/>
  </LazyHydrate>
</template>
```

>*The Lazy Hydration concept you can see above works best when we have a mostly interactive application, but we want to exclude some parts from the hydration. Let’s imagine the other way around: we have a huge application with deeply nested components, it is a static website, but there is this one deeply nested component, which must be interactive.*

```html
<template>
  <LazyHydrate skip>
    <App/>
  </LazyHydrate>
</template>
```

### Solid

Ryan Carniato, taking influence from Marko (discussed more in a later section), proposes using sub-component (or component-level) hydration in [Partial Hydration (November 15, 2020)](https://github.com/solidjs/solid/issues/264).

>*This is the last core feature missing in our SSR story. Truth be told outside of Marko most libraries aren't doing amazing here. We can too consider a more manual approach here at first. I think the key innovation would be to follow Marko's footsteps and recognize there in fact 3 partial hydration modes rather than 2 other libraries are aware of. There is a middle mode that make us considerably more efficient at this. This is uniquely possible given the granular non-component tied approach used here.*

### Svelte

According to Rich Harris in March 2021, partial hydration is [on Svelte's radar but not yet on its roadmap](https://news.ycombinator.com/item?id=26558886). This makes sense since Svelte is already compiling an optimized build of vanilla JavaScript that does not require a runtime. However, Kevin Åberg Kultalahti proposed [Partial Hydration in Svelte (May 9, 2021)](https://github.com/sveltejs/kit/issues/1390) via the `use:action` directive. We will also see in the next section that a Svelte metaframework, Elder.js, has struck out on its own to implement partial hydration.

## Frameworks Built for Partial Hydration

As we can see, essentially every major frontend JavaScript framework has attempted to add some form of partial hydration with varying degrees of success. But there is another category entirely of frontend frameworks that consider partial hydration a key feature to be included from their inception.

### Marko

If there is one framework that can be given credit for first introducing partial hydration as a primary feature (even before the term was invented), my money would be on Marko. Patrick Steele-Idem discussed Marko's internals at length in [Async Fragments: Rediscovering Progressive HTML Rendering with Marko (December 8, 2014)](https://tech.ebayinc.com/engineering/async-fragments-rediscovering-progressive-html-rendering-with-marko/). He also includes a wealth of links to prior art such as:
* [The Lost Art of Progressive HTML Rendering (November 14, 2005)](https://blog.codinghorror.com/the-lost-art-of-progressive-html-rendering/) by Jeff Atwood.
* [Best Practices for Speeding Up Your Web Site (December 12, 2006)](https://developer.yahoo.com/performance/rules.html) by Yahoo's Exceptional Performance Team (epic team name). 
* [Progressive rendering via multiple flushes (December 21, 2009)](https://www.phpied.com/progressive-rendering-via-multiple-flushes/) by Stoyan Stefanov.
* [BigPipe: Pipelining web pages for high performance (June 4, 2010)](https://www.facebook.com/notes/10158791368532200/) by Changhao Jiang.

Michael Rawlings takes a look at Marko's innovations from the perspective of the last few years in [Maybe you don’t need that SPA (May 12, 2020)](https://medium.com/@mlrawlings/maybe-you-dont-need-that-spa-f2c659bc7fec):

>*Marko allows you to build pages by composing components and some of these components can be stateful. Only those components that have state, or other logic targeted at the browser are actually sent to the browser and Marko automatically handles serializing any data from the server needed by these components and mounting them in the browser.*
>
>*This means for most apps, you end up sending down much less code than you would for an equivalent SPA — even with code-splitting. And if no components need to be hydrated? Nothing is hydrated.*

![04-full-page-hydration-versus-component-level-hydration-in-marko](https://miro.medium.com/max/1400/1*5UGQKpjUAJAJMUtOgbJc5A.png)

### Elder.js

Despite Svelte core choosing to hold off on focusing on partial hydration throughout 2020-2021, one of the early entrants into the race for partial hydration leadership was Elder.js by Nick Reese, a static site generator built with Svelte. Focused primarily on SEO, Elder.js lets you hydrate just the parts of the client that need to be interactive.

This lets you reduce your payloads while still having control over component lazy-loading, preloading, and eager-loading. While lesser known than Astro, Elder.js included partial hydration as early as [August](https://github.com/Elderjs/elderjs/commit/19ecd9429284e5358858bbb5fb92e17c41dea33d) [2020](https://github.com/Elderjs/elderjs/commit/b1bffae863228b8721ec78ed3f44822af24bc859), roughly six months before Astro's [initial commit](https://github.com/snowpackjs/astro/commit/af6b029e95e9c98e6fb9c642915d461b8d7f448e).

{% raw %}
```jsx
<div class="right">
  <Clock hydrate-client={{}} />
</div>
```
{% endraw %}

### Astro

Despite the early innovations of Marko and Elder, the framework that deserves the most credit for bringing partial hydration to the mainstream is Astro. Fred K. Schott describes the architecture and goals of Astro in [Introducing Astro: Ship Less JavaScript (June 8, 2021)](https://astro.build/blog/introducing-astro/).

>*Astro works a lot like a static site generator. If you have ever used Eleventy, Hugo, or Jekyll (or even a server-side web framework like Rails, Laravel, or Django) then you should feel right at home with Astro.*
>
>*In Astro, you compose your website using UI components from your favorite JavaScript web framework (React, Svelte, Vue, etc). Astro renders your entire site to static HTML during the build. The result is a fully static website with all JavaScript removed from the final page.*

There are already plenty of frameworks based on [React](https://nextjs.org/docs/advanced-features/static-html-export), [Vue](https://nuxtjs.org/announcements/going-full-static/), and [Svelte](https://sapper.svelte.dev/docs#sapper_export) that include the ability to render your components to static HTML during build time. However, if you want to hydrate these projects on the client then you have to ship an entire bundle of dependencies along with the static HTML. Unlike these previous frameworks, Astro includes the ability to load just a single component and its dependencies where that component is needed.

>*Of course, sometimes client-side JavaScript is inevitable for things like image carousels, shopping carts, and auto-complete search bars. This is where Astro really shines: When a component needs some JavaScript, Astro only loads that one component (and any dependencies). The rest of your site continues to exist as static, lightweight HTML.*
>
>*In Astro, this kind of partial hydration is built into the tool itself. You can even automatically defer components to only load once they become visible on the page with the :visible modifier. This new approach to web architecture is called islands architecture.*

Astro includes five `client:*` directives to hydrate components on the client at runtime. A directive is a component attribute that tells Astro how your component should be rendered.

* `<MyComponent client:load />` - Hydrate the component on page load.
* `<MyComponent client:idle />` - Hydrate the component as soon as main thread is free.
* `<MyComponent client:visible />` - Hydrate the component as soon as the element enters the viewport. Useful for content lower down on the page.
* `<MyComponent client:media={QUERY} />` - Hydrate the component as soon as the browser matches the given media query. Useful for sidebar toggles, or other elements that should only display on mobile or desktop devices.
* `<MyComponent client:only />` - Hydrates the component at page load, similar to `client:load`. The component will be skipped at build time, useful for components that are entirely dependent on client-side APIs. Best avoided unless absolutely needed.

The following example is hydrating a React component (`MyReactComponent`) in the browser with `client:visible`. `client:visible` means the component won't load any client-side JavaScript until it becomes visible in the user's browser.

```jsx
---
import MyReactComponent from '../components/MyReactComponent.jsx';
---

<MyReactComponent client:visible />
```

### Slinkity

[Slinkity](https://slinkity.dev/) is a framework that uses Vite to bring dynamic, client side interactions to your static 11ty sites with [partial hydration](https://slinkity.dev/docs/partial-hydration/). In [Ship JavaScript where it counts with Vite + Partial Hydration (November 12, 2021)](https://www.netlify.com/blog/2021/11/12/ship-javascript-where-it-counts-with-vite-partial-hydration/), Ben Holmes makes the case for turning off partial hydration by default so the developer has to explicitly opt-in:

>*Right now, the Jamstack landscape definitely relies on an opt-out mindset. Too much JavaScript on initial page load? Opt-out with code splitting and lazy ESM loading. Need less JavaScript on your company’s splash page? Opt-out with server-rendered components. The world of partial hydration introduced by Astro, Slinkity + 11ty, or Îles flips that opt-out to an opt-in.*
>
>*Too much JavaScript on initial page load? Well, you’ll need to opt-in to JavaScript hydration for your UI components to create that problem! These frameworks default to no JavaScript shipped for your React, Vue, Svelte, etc, with hydration “modes” to decide how and when those resources should be loaded (if at all).*

To choose how a given component is rendered, you'll need to pass a `render` prop, as in a prop literally named `render`.

```jsx
export const frontMatter = {
  render: 'eager';
}

function Page() {...}
```

Alternatively, you can use a shortcode that includes `render` and the hydration option you want to select. The default for all components is `eager` to mirrow how previous component-based frameworks operate.

{% raw %}
```html
<!-- page-with-shortcode.html -->

<body>
  {% react 'components/Example' 'render' 'eager' %}
</body>
```
{% endraw %}

* A component loaded with `eager` will be rendered to static HTML, and shipped to the client as a JavaScript bundle. In the previous example, visiting `page-with-shortcode.html` imports React and the `components/Example.jsx` JavaScript bundle as soon as the page is done parsing, ensuring our component is interactive as soon as possible.
* `lazy` is similar to `eager` except it only loads your component's JavaScript when your component is scrolled into view by using the [Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API).
* `static` components are rendered to HTML at build time and don't ship any JavaScript to the client, meaning no interactivity or state management. This is useful if you want to use component languages like React as just a templating language.

### îles

[îles](https://iles-docs.netlify.app/) is a static-site generator that provides automatic [partial hydration](https://iles-docs.netlify.app/guide/hydration). The name of the project is French for "islands," as a clever homage to both the islands architecture and it's build tool Vite. Inspired by Astro, you are able to define which components should remain interactive in the production build with `client:` directives in components. Here is an example with MDX:

```markdown
---
audio: /song-for-you.mp3
---

## Play a song

<AudioPlayer {...frontmatter} client:visible/>
```

Components in the `src/components` directory are auto-imported on demand, which is why the example above does not need to import the `AudioPlayer` component. You can also use directives inside Vue components as seen in the following example:

```html
<Audio client:visible :src="audio" :initialDuration="initialDuration"/>
```

### Qwik

[Qwik](https://github.com/BuilderIO/qwik) focuses on resumability of server-side-rendering of HTML and fine-grained lazy-loading of code. It is designed for the best possible time to interactive. According to Miško Hevery in [A First Look at Qwik - The HTML First Framework (June 23, 2021)](https://dev.to/mhevery/a-first-look-at-qwik-the-html-first-framework-af):

>*The basic goal of Qwik is to focus on the time-to-interactive metric by delaying JavaScript as much as possible to take advantage of the browser’s lazy loading capabilities. This is in stark contrast to existing frameworks that treat server-side-rendering and time-to-interactive more as an afterthought rather than a primary goal, which drives all other design decisions.*

With Qwik we now have a framework explicitly aiming to prioritize Time to Interactive which it does through resumability. But what does it mean to be resumable?

>*The basic idea behind Qwik is that it is resumable. It can continue where the server left off. There is but the tiniest amount of code to execute on the client. The `qwikloader`, which takes the static HTML generated from server-side-rendering and resumes it, is less than 1kb and will execute in under 1ms... All the other interactivity of your website is downloaded lazily as you interact with the site in the smallest possible chunks.*

Qwik also [rehydrates components asynchronously](https://dev.to/mhevery/qwik-the-answer-to-optimal-fine-grained-lazy-loading-2hdp) and out of order to ensure that the first interaction does not cause a full application download and bootstrap.

>*Here asynchronously means that the rendering system can pause rendering to asynchronously download a template for a component, and then continue the rendering process. This is in stark contrast to all of the existing frameworks, which have fully synchronous rendering pipelines. And because the rendering is synchronous, there is no place to insert asynchronous lazy-loading. The consequence is that all of the templates need to be present ahead of call to render.*

Ryan Carniato had [this](https://twitter.com/RyanCarniato/status/1458872931962933259) to say about Qwik:

>*It's the only framework that goes from shipping basically only the JavaScript you need on a page (starting maybe even from 0kb), to being able to go full SPA. It can actually with out of order lazy hydration bring in client routing after the fact.*
>
>*It is the only framework that I know of today that basically operates as Islands but can morph into a single application as needed. This is coming to Marko and probably others but we shouldn't get ahead of ourselves. And it's a longer road for traditional SPAs.*

## Conclusion

Frameworks like Qwik bring us full circle from Addy Osmani's warning to developers that slow Time to Interactive scores can cause an uncanny valley effect. Websites ship skeleton HTML that looks like it should be interactive but momentarily is not because the user has to wait for the client to hydrate.

![05-qwik-time-to-interactive-graphic](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dy3wa5ewpz9o9xoa87ys.png)

But as this article demonstrates, as soon as developers are given the ability to configure these parameters they find themselves in a complex decision matrix of trade offs. The next evolution will involve taking these performance advantages and designing conventions to simplify these techniques and create happy paths in the frameworks that developers are already comfortable using.

*Special thanks to Ryan Carniato for early feedback on the post and Ben Holmes for chatting with me for literally months about this stuff.*