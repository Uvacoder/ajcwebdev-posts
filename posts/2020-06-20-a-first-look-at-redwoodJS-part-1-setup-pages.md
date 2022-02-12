---
title: a first look at redwoodJS part 1 - setup, pages
description: Install and create our first Redwood application
date: 2020-06-20
tags:
  - redwoodjs
  - react
  - jamstack
  - javascript
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/7x38o4nssnlf0d9hdo5h.png
layout: layouts/post.njk
---

>*I like to think that most things can be achieved. Whatever you have in your head you can probably pull off with code as long as it's possible within the constraints of the universe.*
>
>*It's just a matter of time... and money... and attention.*  
>
>***Tom Preston-Werner*** - *[Full Stack Radio](https://www.fullstackradio.com/episodes/138)*

*Note: Redwood has not yet reached v1.0 and this material is subject to change. All code samples and commands will be for the current version (v0.36.4)*

# Part 1 - Setup, Pages

[RedwoodJS](https://dev.to/ajcwebdev/redwood-1d46) is a fullstack, serverless framework for the Jamstack. I will start at the very beginning and assume no prior knowledge of Redwood although I do assume a basic knowledge of React. But I'm talking really basic, you'll be fine if you:
* Know what a component is
* Have written at least a dozen lines of JSX
* Have generated at least one project with [create-react-app](https://reactjs.org/docs/create-a-new-react-app.html)

If none of that made sense you should click the link to the `create-react-app` docs and work through those before reading this. This series is geared towards someone who has at least a few months experience, around the point where they start getting comfortable with the workflows of git, npm/yarn, and the terminal.

You will need `yarn` for this tutorial which has slight differences from `npm`. You can find installation instructions [here](https://yarnpkg.com/getting-started/install) or just enter `npm install -g yarn`.

## 1.1 `yarn create redwood-app`

The first step is to create our Redwood project. You can call your project anything you want, just make sure to keep using your name anytime I use `ajcwebdev-redwood` in a terminal command.

```bash
yarn create redwood-app ajcwebdev-redwood
```

Output:

```
success Installed "create-redwood-app@0.36.4" with binaries:
      - create-redwood-app
  ✔ Creating Redwood app
    ✔ Checking node and yarn compatibility
    ✔ Creating directory '/Users/ajcwebdev/ajcwebdev-redwood'
  ✔ Installing packages
    ✔ Running 'yarn install'... (This could take a while)
  ✔ Convert TypeScript files to JavaScript
  ✔ Generating types

Thanks for trying out Redwood!
```

This creates a folder called `ajcwebdev-redwood` holding all the generated code. It also provides a handy-dandy guide to a list of community resources.

>### Join the Community
>
>* **[Join our Forums](https://community.redwoodjs.com)**
>* **[Join our Chat](https://discord.gg/redwoodjs)**
>
>### Get some help
>
>* **[Get started with the Tutorial](https://redwoodjs.com/tutorial)**
>* **[Read the Documentation](https://redwoodjs.com/docs)**
>
>### Stay updated
>
>* **[Sign up for our Newsletter](https://www.redwoodjs.com/newsletter)**
>* **[Follow us on Twitter](https://twitter.com/redwoodjs)**
>
>### Become a Contributor
>
>* **[Learn how to get started](https://redwoodjs.com/docs/contributing)**
>* **[Find a Good First Issue](https://redwoodjs.com/good-first-issue)**

Come hang out with us, we're super fun!

`yarn rw` is the same as `yarn redwood` and can be used to save a few keystrokes. Before entering the next commands create an empty repository on [GitHub](https://repo.new). All the code for this series can be found [on my GitHub](https://github.com/ajcwebdev/ajcwebdev-redwood).

### Initialize git repo

Enter your new project directory and change the GitHub URL in the last command to the repo you just created in the previous step.

```bash
cd ajcwebdev-redwood
git init
git add .
git commit -m "Nailed it"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME_HERE/YOUR_PROJECT_HERE.git
```

### Push to main

```bash
git push -u origin main
```

>*#pushtomain club; feels like github should give me an achievement for that.*
>
>***Dominic Saadi after pushing to main***

## 1.2 `yarn redwood dev`

### Start the development server

```bash
yarn rw dev
```

```
✔ Generating the Prisma client...

api | [nodemon] 2.0.12
api | [nodemon] to restart at any time, enter `rs`
api | [nodemon] watching path(s): redwood.toml
api | [nodemon] watching extensions: js,mjs,json
api | [nodemon] starting `yarn rw-api-server-watch`

gen | Generating TypeScript definitions and GraphQL schemas...
gen | 10 files generated

api | Building... Took 625 ms
api | Starting API Server... Took 5 ms
api | Listening on http://localhost:8911/
api | Importing Server Functions... 
api | /graphql 1374 ms
api | ... Imported in 1374 ms

web | assets by path static/js/*.js 2.55 MiB
web |   asset static/js/app.bundle.js 2.5 MiB [emitted] (name: app) 1 related asset
web |   asset static/js/runtime-app.bundle.js 48.8 KiB [emitted] (name: runtime-app) 1 related asset
web |   asset static/js/src_pages_NotFoundPage_NotFoundPage_js.chunk.js 3.37 KiB [emitted] 1 related asset
web | asset README.md 1.9 KiB [emitted] [from: public/README.md] [copied]
web | asset favicon.png 1.83 KiB [emitted] [from: public/favicon.png] [copied]
web | asset index.html 483 bytes [emitted]
web | asset robots.txt 24 bytes [emitted] [from: public/robots.txt] [copied]

web | Entrypoint app 2.55 MiB (2.56 MiB) = static/js/runtime-app.bundle.js 48.8 KiB static/js/app.bundle.js 2.5 MiB 2 auxiliary assets
web | orphan modules 432 KiB [orphan] 115 modules
web | runtime modules 32.7 KiB 17 modules
web | modules by path ../node_modules/ 2.08 MiB 532 modules
web | modules by path ./src/ 10.8 KiB
web |   modules by path ./src/*.js 3.46 KiB
web |     ./src/App.js 1.59 KiB [built] [code generated]
web |     ./src/Routes.js 1.88 KiB [built] [code generated]
web |   modules by path ./src/pages/ 5.18 KiB
web |     ./src/pages/FatalErrorPage/FatalErrorPage.js 2.81 KiB [built] [code generated]
web |     ./src/pages/NotFoundPage/NotFoundPage.js 2.37 KiB [built] [code generated]
web |   modules by path ./src/*.css 2.19 KiB
web |     ./src/index.css 1.89 KiB [built] [code generated]
web |     ../node_modules/css-loader/dist/cjs.js??ruleSet[1].rules[0].oneOf[4].use[1]!./src/index.css 305 bytes [built] [code generated]
web | webpack 5.51.1 compiled successfully in 4921 ms
```

Our server is now running on `localhost:8910` (to remember just count 8-9-10). Open a browser and enter `localhost:8910` into the address bar. If you have done everything correctly up to this point you will see the Redwood starter page.

![01-redwood-starter-page](https://dev-to-uploads.s3.amazonaws.com/i/mc6fc28g9grmsqz2vzwy.jpg)

WHOOPS, it worked, we're up and running. Don't worry too much about what it says about custom routes, we'll talk about that in the next article. Here is the file structure that has been created for us.

```
├── api
│   ├── db
│   │   ├── schema.prisma
│   │   └── seeds.js
│   ├── src
│   │   ├── functions
│   │   │   └── graphql.js
│   │   ├── graphql
│   │   ├── lib
│   │   │   ├── auth.js
│   │   │   ├── db.js
│   │   │   └── logger.js
│   │   └── services
│   └── package.json
│
├── web
│   ├── public
│   │   ├── favicon.png
│   │   ├── README.md
│   │   └── robots.txt
│   ├── src
│   │   ├── components
│   │   ├── layouts
│   │   ├── pages
│   │   │   ├── FatalErrorPage
│   │   │   │   └── FatalErrorPage.js
│   │   │   └── NotFoundPage
│   │   │       └── NotFoundPage.js
│   │   ├── App.js
│   │   ├── index.css
│   │   ├── index.html
│   │   └── Routes.js
│   └── package.json
│
├── .env
├── .env.defaults
├── .env.example
├── .gitignore
├── README.md
├── package.json
├── redwood.toml
└── yarn.lock
```

In Redwood, our frontend code is contained in the `web` folder and our backend code is contained in the `api` folder. We'll look at the `web` folder first. Redwood structures the `web` folder a bit like `create-react-app` projects with a `public` and `src` folder.

## 1.3 `redwood generate page`

With our application now set up we can start creating pages with the `generate page` command

### Generate home page

The `generate page` command accepts two arguments for setting the name of the page and its path.

```bash
yarn rw g page home /
```

The `g page home /` command creates a home page and a folder to hold that page. It also creates a couple of extra files that will be useful later in the series. These include a Storybook file along with testing and mocking files.

```
✔ Generating page files...
  ✔ Successfully wrote file `./web/src/pages/HomePage/HomePage.stories.js`
  ✔ Successfully wrote file `./web/src/pages/HomePage/HomePage.test.js`
  ✔ Successfully wrote file `./web/src/pages/HomePage/HomePage.js`
✔ Updating routes file...
```

Since I only entered `home` it will use that to name both the folder and the component file but you can specify each if necessary.

```
└── pages
    ├── FatalErrorPage
    │   └── FatalErrorPage.js
    ├── HomePage
    │   │── HomePage.js
    │   │── HomePage.stories.js
    │   └── HomePage.test.js
    └── NotFoundPage
        └── NotFoundPage.js
```

Return to your browser and you will now see a new page instead of the landing page.

![02-HomePage-on-localhost-8910](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9d873zyjhx1tsbnapk1e.png)

Let's look at the code that was generated for this page. It's a component called `HomePage` that returns a `<div>` with a header `<h1>` and a paragraph tag `<p>`. The [`MetaTags`](https://redwoodjs.com/docs/seo-head#setting-meta-tags-open-graph-directives) component can be used to set relevant SEO tags such as `title`, `description`, and `og:image`.

```jsx
// web/src/pages/HomePage/HomePage.js

import { Link, routes } from '@redwoodjs/router'
import { MetaTags } from '@redwoodjs/web'

const HomePage = () => {
  return (
    <>
      <MetaTags
        title="Home"
        // description="Home description"
        /* you should un-comment description and add a unique description, 155 characters or less
      You can look at this documentation for best practices : https://developers.google.com/search/docs/advanced/appearance/good-titles-snippets */
      />

      <h1>HomePage</h1>
      <p>
        Find me in <code>./web/src/pages/HomePage/HomePage.js</code>
      </p>
      <p>
        My default route is named <code>home</code>, link to me with `
        <Link to={routes.home()}>Home</Link>`
      </p>
    </>
  )
}

export default HomePage
```

This should be pretty self-explanatory if you have experience with React. If this doesn't look familiar it would be helpful to spend a little time studying React by itself before jumping into Redwood.

Now we'll edit the page and see what happens.

```jsx
// web/src/pages/HomePage/HomePage.js

import { MetaTags } from '@redwoodjs/web'

const HomePage = () => {
  return (
    <>
      <MetaTags
        title="Home"
        description="The home page of the website"
      />

      <h1>ajcwebdev</h1>
      <p>This page is the home!</p>

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

Feel free to include links to your own social accounts. With those changes made return to your browser.

![03-HomePage-on-localhost-8910-edited](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wwwqjpczz5w7tchw6ibv.png)

### Generate about page

Now we are going to generate our `about` page.

```bash
yarn rw g page about
```

```
✔ Generating page files...
  ✔ Successfully wrote file `./web/src/pages/AboutPage/AboutPage.stories.js`
  ✔ Successfully wrote file `./web/src/pages/AboutPage/AboutPage.test.js`
  ✔ Successfully wrote file `./web/src/pages/AboutPage/AboutPage.js`
✔ Updating routes file...
```

Like before, this creates an `AboutPage` component inside of an `AboutPage` folder along with files for Storybook and testing.

```
└── pages
    ├── AboutPage
    │   │── AboutPage.js
    │   │── AboutPage.stories.js
    │   └── AboutPage.test.js
    ├── FatalErrorPage
    │   └── FatalErrorPage.js
    ├── HomePage
    │   │── HomePage.js
    │   │── HomePage.stories.js
    │   └── HomePage.test.js
    └── NotFoundPage
        └── NotFoundPage.js
```

We don't have a link to the about page, but we can enter the route manually into our browser by adding `/about` after `localhost:8910`.

![04-AboutPage-on-localhost-8910](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tl7icn9ggfidv1olo6xg.png)

Open up the code and it's another React component much like the last! Components are kind of a big deal in React.

```jsx
// web/src/pages/AboutPage/AboutPage.js

import { Link, routes } from '@redwoodjs/router'
import { MetaTags } from '@redwoodjs/web'

const AboutPage = () => {
  return (
    <>
      <MetaTags
        title="About"
        // description="About description"
        /* you should un-comment description and add a unique description, 155 characters or less
      You can look at this documentation for best practices : https://developers.google.com/search/docs/advanced/appearance/good-titles-snippets */
      />

      <h1>AboutPage</h1>
      <p>
        Find me in <code>./web/src/pages/AboutPage/AboutPage.js</code>
      </p>
      <p>
        My default route is named <code>about</code>, link to me with `
        <Link to={routes.about()}>About</Link>`
      </p>
    </>
  )
}

export default AboutPage
```

We can also edit this page just like the `home` page.

```jsx
// web/src/pages/AboutPage/AboutPage.js

import { MetaTags } from '@redwoodjs/web'

const AboutPage = () => {
  return (
    <>
      <MetaTags
        title="About"
        description="The page that tells you about stuff"
      />

      <h1>About</h1>
      <p>This page tells you about stuff!</p>
    </>
  )
}

export default AboutPage
```

With those changes return to your browser.

![05-AboutPage-on-localhost-8910-edited](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hpkol8jps90rbiel9hdy.png)

## 1.4 `yarn redwood --help`

If at any point you're having trouble remembering commands you can get a quick reminder of all the commands with the `--help` command.

```bash
yarn rw --help
```

|Command                    |Description         |Alias|
|---------------------------|--------------------|-----|
|`rw build` [side..]        |Build for production||
|`rw check`                 |Structural diagnostics for a Redwood project (experimental)|`diagnostics`|
|`rw console`               |Launch an interactive Redwood shell (experimental)|`c`|
|`rw data-migrate` <command>|Migrate the data in your database|`dm`, `dataMigrate`|
|`rw deploy` <target>       |Deploy your Redwood project||
|`rw destroy` <type>        |Rollback changes made by the generate command|`d`|
|`rw dev` [side..]          |Start development servers for api, db, and web||
|`rw exec` <name>           |Run scripts generated with yarn generate script||
|`rw generate` <type>       |Generate boilerplate code and type definitions|`g`|
|`rw info`                  |Print your system environment information||
|`rw lint`                  |Lint your files||
|`rw open`                  |Open your project in your browser||
|`rw prerender`             |Prerender pages of your Redwood app at build time|`render`|
|`rw prisma` [commands..]   |Run Prisma CLI with experimental features||
|`rw serve` [side]          |Run server for api or web in production||
|`rw setup` <commmand>      |Initialize project config and install packages||
|`rw storybook`             |Launch Storybook: An isolated component development environment|`sb`|
|`rw test` [filter..]       |Run Jest tests. Defaults to watch mode||
|`rw ts-to-js`              |Convert a TypeScript project to JavaScript||
|`rw type-check` [sides..]  |Run a TypeScript compiler check on your project|`tsc`, `tc`|
|`rw upgrade`               |Upgrade all @redwoodjs packages via interactive CLI||

### Options

* `--help` - Show help
* `--version` - Show version number
* `--cwd` - Working directory to use (where `redwood.toml` is located)

## 1.5 `redwood.toml`

`redwood.toml` contains the configuration settings for your Redwood app and is what makes your Redwood app a Redwood app. If you remove it and try to run `yarn rw dev`, you'll get an error. You can see the full list of options on the [App Configuration](https://redwoodjs.com/docs/app-configuration-redwood-toml) doc.

```toml
[web]
  title = "Redwood App"
  port = 8910
  apiProxyPath = "/.redwood/functions"
  includeEnvironmentVariables = []
[api]
  port = 8911
```

In the [next part](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-part-2-44ph) we'll take a look at Redwood's router and create links for the pages we created.