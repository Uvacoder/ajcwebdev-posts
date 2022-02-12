---
title: a first look at redwoodJS part 2 - routes, layouts
description: Explore Redwood’s router and create links for our pages
date: 2020-06-21
tags:
  - redwoodjs
  - react
  - jamstack
  - javascript
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/7x38o4nssnlf0d9hdo5h.png
layout: layouts/post.njk
---

>*We basically wrote the tutorial the way that we wanted the code to work and then we made the tutorial work by writing the code. As opposed to [README-Driven Development](https://tom.preston-werner.com/2010/08/23/readme-driven-development.html), this is Tutorial-Driven Development.*
>
>***Tom Preston-Werner*** - *[Full Stack Radio](https://www.fullstackradio.com/episodes/138)*

# Part 2 - Routes, Layouts

In [part 1](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-1017) we installed and created our first RedwoodJS application with the Redwood CLI. We used:
* `yarn create redwood-app` to generate the initial app
* `redwood generate page`
  * `HomePage` folder containing a `HomePage` file containing a `HomePage` component
  * `AboutPage` folder containing an `AboutPage` file containing an `AboutPage` component

We were able to navigate between the different pages in our browser by entering `/about` for the About page or a slash (`/`) for the Home page. Depending on your experience with React this may have been surprising to you.

If you've worked on routing in React before you know that to achieve this there needs to be an entirely different package imported containing the router and then your routes need to be wrapped in a `<BrowserRouter>` or `<Router>` component to give the router access to your pages. Well guess what.....

## 2.1 `Routes.js`

```jsx
// web/src/Routes.js

import { Router, Route } from '@redwoodjs/router'

const Routes = () => {
  return (
    <Router>
      <Route path="/about" page={AboutPage} name="about" />
      <Route path="/" page={HomePage} name="home" />
      <Route notfound page={NotFoundPage} />
    </Router>
  )
}

export default Routes
```

In our `web/src` folder we have a file called `Routes.js`. When we used the CLI to generate `HomePage` and `AboutPage` we also created these routes.

Components from `web/src/pages` are auto-imported. Nested directories are supported, and should be uppercase. Each sub-directory will be prepended onto the component name. For example:

`web/src/pages/HomePage/HomePage.js` -> `HomePage`
`web/src/pages/Admin/YoPage/YoPage.js` -> `AdminYoPage`

## 2.2 `App.js`

The other file in our `web/src` folder is `index.js` which is our root component that `ReactDOM` renders to the screen.

All [components are composable](https://reactjs.org/blog/2013/06/05/why-react.html) in React. This encourages the creation of reusable UI components that present data that changes over time. Web application UIs were traditionally built using templates or HTML directives.

But in React you always have a root component that contains other components and those components can contain other components.

```jsx
// web/src/App.js

import { FatalErrorBoundary, RedwoodProvider } from '@redwoodjs/web'
import { RedwoodApolloProvider } from '@redwoodjs/web/apollo'

import FatalErrorPage from 'src/pages/FatalErrorPage'
import Routes from 'src/Routes'

import './index.css'

const App = () => (
  <FatalErrorBoundary page={FatalErrorPage}>
    <RedwoodProvider titleTemplate="%PageTitle | %AppTitle">
      <RedwoodApolloProvider>
        <Routes />
      </RedwoodApolloProvider>
    </RedwoodProvider>
  </FatalErrorBoundary>
)

export default App
```

If that's confusing just give it some time and it'll start to click as this series goes on. The important take away is that we have three nested layers:

### `RedwoodApolloProvider`

Redwood Apps come ready-to-query with `RedwoodApolloProvider`, which is a Provider that wraps [ApolloProvider](https://www.apollographql.com/docs/react/api/react/hooks/#the-apolloprovider-component). But with a bit of configuring, you can swap out [RedwoodApolloProvider](https://github.com/redwoodjs/redwood/blob/main/packages/web/src/apollo/index.tsx) for your client of choice.

For an example of configuring your own GraphQL Client, see the [redwoodjs-react-query-provider](https://www.npmjs.com/package/redwoodjs-react-query-provider). You can also install [react-query](https://react-query.tanstack.com/) directly into your project and fire away with the [`fetch()` API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API).

Why would someone do this? If you don't import `RedwoodApolloProvider` it won't be included in your bundle size. This will reduce one of your heaviest client dependencies. There are also a variety of different default caching strategies across different clients.

## 2.3 `Link`

Matching URLs to Pages is the first half of the equation when it comes to routing. The other half is generating links to your pages. We want to add a link to the page for navigating between our `home` and `about` pages. We'll need three things in our `HomePage.js`:

1. Import `Link` and `routes` from `@redwoodjs/router`
2. `<Link to={}>About</Link>`
3. Pass `routes.about()` into `<Link to={}>`

```jsx
// web/src/pages/HomePage/HomePage.js

import { Link, routes } from '@redwoodjs/router'
import { MetaTags } from '@redwoodjs/web'

const HomePage = () => {
  return (
    <>
      <MetaTags
        title="Home"
        description="The home page of the website"
      />

      <header>
        <h1>ajcwebdev</h1>

        <nav>
          <ul>
            <li>Home</li>
          </ul>
          <ul>
            <li><Link to={routes.about()}>About</Link></li>
          </ul>
        </nav>
      </header>

      <main>This page is the home!</main>

      <footer>
        <h3>Find me online:</h3>

        <ul>
          <li><a href="https://dev.to/ajcwebdev">Blog</a></li>
          <li><a href="https://twitter.com/ajcwebdev">Twitter</a></li>
          <li><a href="https://github.com/ajcwebdev">GitHub</a></li>
        </ul>
      </footer>
    </>
  )
}

export default HomePage
```

`Link` generates a link to a route. The `routes` object can access URL generators for any of the routes. These are called ***named route functions*** because they are named after anything you include in the `name` prop of the `Route`.

Return to the browser to see the changes to your Home page.

![01-HomePage-with-about-link](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9qywdczg56jvhk71et47.png)

Click the link to be brought to the About page.

![02-AboutPage](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jqwkvxxutrr7v8rv6x6b.png)

Now we want to be able to navigate back to `HomePage` once we are on `AboutPage`. We'll do the exact same three steps except we pass in `routes.home()` instead of `routes.about()` because we want to navigate to `HomePage`. 

```jsx
// web/src/pages/AboutPage/AboutPage.js

import { Link, routes } from '@redwoodjs/router'
import { MetaTags } from '@redwoodjs/web'

const AboutPage = () => {
  return (
    <>
      <MetaTags
        title="About"
        description="The page that tells you about stuff"
      />
      
      <header>
        <h1>ajcwebdev</h1>

        <nav>
          <ul>
            <li><Link to={routes.home()}>Home</Link></li>
          </ul>
          <ul>
            <li><Link to={routes.about()}>About</Link></li>
          </ul>
        </nav>
      </header>

      <main>
        <h1>About</h1>
        <p>This page tells you about stuff!</p>
      </main>
    </>
  )
}

export default AboutPage
```

### Return to your About page

![03-AboutPage-with-return-home-link](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fgr71ywsmhc4es25r36c.png)

There is now a link that will take you back to the Home page when clicked.

## 2.4 `redwood generate layout`

You've probably been on a website before. Usually there's some kind of navigation bar at the top and footer at the bottom that stays consistent as you travel around the website. Redwood's folder structure is designed to make this really easy.

![04-layouts](https://dev-to-uploads.s3.amazonaws.com/i/wm80gmj2hyf3mehmr1ox.jpg)

Remember how I said everything is a component containing other components? Pages contain components and your layout will contain all of your pages. This is natural to do in React because pages and layouts are themselves components.

We can generate a file for our layout with `yarn redwood generate layout` or `yarn rw g layout`. This will work a lot like the `generate page` commands from earlier.

```bash
yarn rw g layout blog
```

```
✔ Generating layout files...
  ✔ Successfully wrote file `./web/src/layouts/BlogLayout/BlogLayout.test.js`
  ✔ Successfully wrote file `./web/src/layouts/BlogLayout/BlogLayout.stories.js`
  ✔ Successfully wrote file `./web/src/layouts/BlogLayout/BlogLayout.js`
```

This will create a folder inside `web/src/layouts` instead of `web/src/pages` because we are generating a layout and not a page.

```
└── layouts
    └── BlogLayout
        │── BlogLayout.js
        │── BlogLayout.stories.js
        └── BlogLayout.test.js
```

## 2.5 `BlogLayout`

```jsx
// web/src/layouts/BlogLayout/BlogLayout.js

const BlogLayout = ({ children }) => {
  return <>{children}</>
}

export default BlogLayout
```

`children` is where the magic will happen. Any page content given to the layout will be rendered here.

```jsx
// web/src/layouts/BlogLayout/BlogLayout.js

import { Link, routes } from '@redwoodjs/router'

const BlogLayout = ({ children }) => {
  return (
    <>
      <header>
        <h1>ajcwebdev</h1>
        
        <nav>
          <ul>
            <li><Link to={routes.home()}>Home</Link></li>
          </ul>
          <ul>
            <li><Link to={routes.about()}>About</Link></li>
          </ul>
        </nav>
      </header>
      
      <main>{children}</main>

      <footer>
        <h3>Find me online:</h3>

        <ul>
          <li><a href="https://dev.to/ajcwebdev">Blog</a></li>
          <li><a href="https://twitter.com/ajcwebdev">Twitter</a></li>
          <li><a href="https://github.com/ajcwebdev">GitHub</a></li>
        </ul>
      </footer>
    </>
  )
}

export default BlogLayout
```

Now we'll add a `<BlogLayout>` wrapper around the content returned in our `HomePage` and `AboutPage` components.

```jsx
// web/src/pages/HomePage/HomePage.js

import { MetaTags } from '@redwoodjs/web'
import BlogLayout from 'src/layouts/BlogLayout'

const HomePage = () => {
  return (
    <>
      <BlogLayout>
        <MetaTags
          title="Home"
          description="The home page of the website"
        />

        <p>This page is the home!</p>
      </BlogLayout>
    </>
  )
}

export default HomePage
```

We can remove the `import` for `Link` and `routes` from `HomePage` since those are now inside the `BlogLayout`.

```jsx
// web/src/pages/AboutPage/AboutPage.js

import { MetaTags } from '@redwoodjs/web'
import BlogLayout from 'src/layouts/BlogLayout'

const AboutPage = () => {
  return (
    <BlogLayout>
      <MetaTags
        title="About"
        description="The page that tells you about stuff"
      />

      <h1>About</h1>
      <p>This page tells you about stuff!</p>
    </BlogLayout>
  )
}

export default AboutPage
```

The import statement uses `src/layouts/BlogLayout` and not `../src/layouts/BlogLayout` or `./src/layouts/BlogLayout`. This is a convenience feature so you don't need to worry about the nesting of your folders.

`src` is an alias to the `src` path in the current workspace. 

* When working in `web`, `src` points to `web/src`
* When working in `api`, `src` points to `api/src`

If we look at `HomePage` it should look and behave the same way as before except we have turned our title in the `<h1>` tag into a `HomePage` link.

![05-HomePage-with-BlogLayout](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1neavm10x588eal0sixm.png)

`AboutPage` is the same except we have removed the link to return Home since we can now click the title.

![06-AboutPage-with-BlogLayout](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/htk6uv4mj0zhh12vftsf.png)

In the [next part](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-part-3-5ao5) we'll start working with a database and learn how to create, retrieve, update, and destroy blog posts.