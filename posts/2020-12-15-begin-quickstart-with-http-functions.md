---
title: begin quickstart with http functions
description: Begin is a platform for building modern web apps, sites, & APIs. It leverages globally available serverless infrastructure, SSD-backed databases, CDNs, and GitHub powered continuous integration.
date: 2020-12-15
tags:
  - begin
  - serverless
  - aws
  - lambda
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/l5g3q355ch0bbv8ruye9.png
layout: layouts/post.njk
---

Begin is a platform for building modern web apps, sites, & APIs. It leverages globally available serverless infrastructure, SSD-backed databases, CDNs, and GitHub powered continuous integration. It is scientifically proven to be "ridiculously quick."

## Prerequisites

* Begin will provision your [GitHub](https://github.com/join) repo pre-wired with the integrations it needs such as webhooks to Begin's CI.
* Begin manages AWS infrastructure solely on [Node 12](https://nodejs.org/en/download/) with additional runtimes coming soon.
* Recent releases of Node bundle npm 5.x but [npm 6.x](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) is required for local development. Run `npm install npm@latest -g` to upgrade.
* Click `Login` on the Begin home page to create a [Begin account](https://begin.com). Authorize it with GitHub and pick a username.

## Creating and deploying an app

Login to your Begin account and click the ["Create new app"](https://begin.com/apps/new) button. You will first choose either a Node.js or Deno runtime.

![01-begin-new-app-selector](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m0lhkmvj6gqz6k1q92au.png)

By default you will see a list of Node.js framework-specific starter apps. Scroll down to see a list of additional example apps.

![02-example-apps](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/flhic3jxt4cm51591r4t.png)

Select an app from the list of starters. You will then be asked to give your project a name.

![03-create-hello-world-app](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/oe1dx6t8k2n9tisc24ix.png)

Click "Create Hello World app" to spin up a new project under `github.com/{your GH username}/{your repo name}`.

![04-begin-activity](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ifr2tz8dfzludftax3pw.png)

After Begin sets up your repo it will kick off its first deploy to your `staging` environment.

![05-verify-install](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7f60g44zc9pfsiigf6bn.png)

By default, each commit to `main` initiates a build.

![06-built-and-deployed](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/x0fxpekzb6z46t896eud.png)

If the build is green that build is immediately deployed to your app's `staging` environment. To access your `staging` environment, click the `Staging` link in the build status module.

![07-staging](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/91odb5lwt0u13llgbgcx.png)

Click `Production` to see your production environment.

![08-production](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ap89kywfxwu7exutu5ve.png)

## Project structure

Begin applications are comprised of many small, individually executing cloud functions.

```
.
├── .gitignore
├── LICENSE
├── index.js
├── package.json
└── readme.md
```

If you try to create your application as a single, giant, monolithic Lambda function then Brian LeRoux will personally come to your house, slap you in the face, and kick your dog.

## package.json

Begin example apps written for the `node.js` runtime use `JSON` added to the apps `package.json` file.

```json
"name": "begin-app",
"version": "0.0.0",
"description": "Begin basic Hello World! app for Node",
"arc": {
  "app": "hello-world",
  "http": [
    {
      "/": {
        "method": "any",
        "src": "."
      }
    }
  ]
},
```

### Start script

```json
"scripts": {
  "start": "cross-env NODE_ENV=testing npx sandbox"
},
```

### Development dependencies

```json
"devDependencies": {
  "@architect/functions": "latest",
  "@architect/sandbox": "^3.3.6",
  "@begin/data": "latest",
  "cross-env": "^7.0.3"
}
```

## index.js

Function directories must contain an `index.js` file that services the cloud function handler and any dependencies required for it to operate.

### HTTP Functions

Each HTTP Function directory services a handler for a publicly available HTTP route.

```javascript
exports.handler = async function http(req) {
  let html = `
    <!doctype html>
    <html lang=en>
      <head>
        <meta charset=utf-8>
        <title>This is a title</title>
        <link
          rel="stylesheet"
          href="https://static.begin.app/starter/default.css"
        >
        <link
          href="data:image/x-icon;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mNkYAAAAAYAAjCB0C8AAAAASUVORK5CYII=" 
          rel="icon"
          type="image/x-icon"
        >
      </head>
      <body>
        <h1 class="center-text">
          I hath been shipped
        </h1>

        <p class="center-text">
          Nailed it
        </p>
      </body>
    </html>
  `

  return {
    headers: {
      'content-type': 'text/html; charset=utf8',
      'cache-control': 'no-cache, no-store, must-revalidate, max-age=0, s-maxage=0'
    },
    statusCode: 200,
    body: html
  }
}
```

To deploy your changes, click "Deploy to Production"

![09-deploy-to-production](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/z9bwy8gvgb6lqlosxrh3.png)

Click `Ship it` to `ship` that sucker.

![10-i-hath-been-shipped](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dp37sf9jg9fjbrwlqeml.png)

You can see this thoroughly shipped site at the [following link](https://play-8q9.begin.app/).