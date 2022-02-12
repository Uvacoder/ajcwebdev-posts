---
title: a history of hosting redwoodJS - how the universal deployment machine was built
date: 2021-06-02
tags:
  - redwoodjs
  - lambda
  - pm2
  - serverless
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hk00mtlotvtfrkmsrcfg.jpg
layout: layouts/post.njk
---

>*We want to deploy to all the things. Write once, learn once, run everywhere.*
>
>***[Peter Pistorius](https://github.com/peterp)*** - *[Deployment Targets tracking issue (August 8, 2020)](https://github.com/redwoodjs/redwood/issues/947)*

## Phase 0 - Netlify

[RedwoodJS](https://redwoodjs.com/) was originally architected to run as a static frontend deployed on a CDN that sends requests to a serverless GraphQL handler. The GraphQL handler was first deployed on AWS Lambda via Netlify Functions. Despite all the jargon in the previous description, you can forget about most of those terms with the notable exceptions of frontend and GraphQL.

You can think about your Redwood application as consisting of a React frontend speaking to a Node backend that exposes a GraphQL API. For the initial v0.1 release of Redwood, the React frontend was hosted on Netlify's CDN and the GraphQL API was hosted on a serverless Lambda handler via Netlify Functions.

However, there's no inherent reason a Redwood app has to be deployed this way. This lead many to wonder and others to experiment with different methods of deploying the frontend and API.

>*What does it take to deploy the redwood `api` part to AWS Lambda directly? I would like to take a shot at it.*
>
>***[Jaikant](https://github.com/Jaikant)*** - *[Deploying to AWS Lambda (April 6, 2020)](https://community.redwoodjs.com/t/deploying-to-aws-lambda/316)*

>*We currently rely on Netlify to do the heavy lifting of deployment for us, but we intend to make it easy to write deployment providers that will allow deployment to a wide variety of services.*
>
>***[Tom Preston-Werner](http://github.com/mojombo)*** - *[Deploying to AWS Lambda (April 8, 2020)](https://community.redwoodjs.com/t/deploying-to-aws-lambda/316/3)*

## Phase 1 - AWS Lambda cause I don't want a server

The first adventurer sought to bypass Netlify Functions and deploy the API directly to AWS Lambda.

>*So I got the api part deployed and working on AWS lambda directly using serverless. I had to move some files around in the `dist` directory and also set the environment variables in AWS. I shall create an issue to discuss the changes.*
>
>***[Jaikant](https://github.com/Jaikant)*** - *[Deploying to AWS Lambda (April 13, 2020)](https://community.redwoodjs.com/t/deploying-to-aws-lambda/316/11)*

### Serverless Framework

>*On deploying to AWS lambda, we would get an api endpoint. This `api` endpoint would be needed within the `web` part in apollo client for the graphql queries.*
>
>*The build step would need the `api` to be built first, before the `web`, so the endpoint is known. Using [serverless](https://github.com/serverless/serverless) to deploy, the config `serverless.yml`, which is at the root of the project.*
>
>***[Jaikant](https://github.com/Jaikant)*** - *[Support Deployments to AWS Lambda (April 14, 2020)](https://github.com/redwoodjs/redwood/issues/431)*

```yaml
service: jai-redwood-1
plugins:
  - serverless-dotenv-plugin

provider:
  name: aws
  runtime: nodejs12.x
  region: ap-south-1

package:
  artifact: zipball/graphql.zip

functions:
  graphql:
    description: redwood on aws lambda
    memorySize: 1024
    timeout: 30
    handler: dist/functions/graphql.handler
    events:
      - http:
          path: graphql
          method: GET
          cors: true
      - http:
          path: graphql
          method: POST
          cors: true
      - http:
          path: playground
          method: ANY
          cors: true
```

>*I tried this with the latest version of `zip-it-and-ship-it` for AWS Lambda and it seems to have worked out of the box. I did not have to make any changes to the zip folder and I used `handler: graphql.handler` in `serverless.yml`.*
>
>*I changed the one line in `serverless.yml` from `handler: dist/functions/graphql.handler` to `handler: graphql.handler` since `graphql.js` now lives in the root of the zipball.*
>
>***[Hemil Desai](https://github.com/hemildesai)*** - *[Support Deployments to AWS Lambda (August 8, 2020)](https://github.com/redwoodjs/redwood/issues/431#issuecomment-670853186)*

### yarn rw setup deploy aws-serverless

>*I've just chatted with @hemildesai and we've come up with some concrete steps for adding AWS deployments using the serverless framework.*
>
>*Introduce a deploy command:*
>
>* *`yarn rw deploy api aws-serverless`*
>* *runs `yarn build api`, built files are in `./api/dist`*
>* *runs zip it and ship it, placing the files in `./api/dist/`*
>* *runs serverless deploy*
>
>***[Peter Pistorius](https://github.com/peterp)*** - *[Support Deployments to AWS Lambda (August 19, 2020)](https://github.com/redwoodjs/redwood/issues/431#issuecomment-676357234)*

```bash
yarn rw setup deploy aws-serverless
```

```yaml
service: app

plugins:
  - serverless-dotenv-plugin

provider:
  name: aws
  runtime: nodejs12.x
  region: us-east-2
  httpApi:
    cors: true
    payload: '1.0'
  stackTags:
    source: serverless
    name: Redwood Lambda API with HTTP API Gateway
  tags:
    name: Redwood Lambda API with HTTP API Gateway

package:
  individually: true

functions:
  graphql:
    description: graphql function deployed on AWS Lambda
    package:
      artifact: api/dist/zipball/graphql.zip
    memorySize: 1024
    timeout: 25
    tags:
      endpoint: /.redwood/functions/graphql
    handler: graphql.handler
    events:
      - httpApi:
          path: /.redwood/functions/graphql
          method: GET
      - httpApi:
          path: /.redwood/functions/graphql
          method: POST
```

## Phase 2 - PM2 cause I want a server

>*Do you prefer hosting Redwood on your own server, the traditional serverful way, instead of all this serverless magic? Well, you can! In this recipe we configure a Redwood app with PM2 and Nginx on a Linux server.*
>
>***[Nick Geerts](https://github.com/njjkgeerts)*** - *[Add serverfull hosting recipe to cookbook (October 24, 2020)](https://github.com/redwoodjs/redwoodjs.com/pull/431)*

With the `deploy aws-serverless` command, developers now had the ability to drop down to the underlying AWS Lambdas and were no longer reliant on Netlify Functions. This may have looked like it was presaging a stronger push towards a wider array of serverless function providers.

But the exact opposite happened, the next investigation involved leaving the serverless world behind entirely. Despite their appeal from a DX perspective, larger Redwood applications hosted on AWS Lambda experienced noticeable cold starts or exceeded the maximum allotted memory for their handler.

And for some developers running a server is not challenging or foreign, it is familiar and simple.

>*I’m a freelancer and have a lot of smaller clients and host my own Linux server with node.js instances in PM2 with a reverse proxy in Nginx. I was looking into deploying an example Redwood app to my own Linux server. When I build the project, the `web/dist` output is simple enough - just files to host in Nginx (yay JAMstack!).*
>
>*However, how can I host the function artifacts in `api/dist`? Do these conform to a spec like the serverless framework and how can I deploy them to PM2? Or do I need to use something like OpenFAAS or the Fn Project?*
>
>***[Nick Geerts](https://github.com/njjkgeerts)*** - *[Serverfull hosting (September 13, 2020)](https://community.redwoodjs.com/t/serverfull-hosting/1182)*

### pm2.config.js

>*I have a working example!* :sunglasses:
>
>*Live site: http://redwood-pm2.nickgeerts.com*
>*Repo: https://github.com/njjkgeerts/redwood-pm2*
>
>***[Nick Geerts](https://github.com/njjkgeerts)*** - *[Serverfull hosting (September 26, 2020)](https://community.redwoodjs.com/t/serverfull-hosting/1182/5)*

```javascript
const name = 'redwood-pm2'
const repo = 'git@github.com:njjkgeerts/redwood-pm2.git'
const user = 'deploy'
const path = `/home/${user}/${name}`
const host = 'example.com'
const port = 8911
const build = `yarn install && yarn rw build && yarn rw prisma deploy`

module.exports = {
  apps: [
    {
      name,
      node_args: '-r dotenv/config',
      cwd: `${path}/current/`,
      script: 'yarn rw serve api',
      args: '--port ${port}',
      env: {
        NODE_ENV: 'development',
      },
      env_production: {
        NODE_ENV: 'production',
      },
    },
  ],

  deploy: {
    production: {
      user,
      host,
      ref: 'origin/master',
      repo,
      path,
      ssh_options: 'ForwardAgent=yes',
      'post-deploy': `${build} && pm2 reload pm2.config.js --env production && pm2 save`,
    },
  },
}
```

### Server-less or Server-more?

>*"Serverfull" makes sense in the context of Serverless, but to the best of my knowledge, it's not a well-known term (yet). So when people are browsing the left column navigation docs, or using the site search, is there anything we can add to help others find this document and identify immediately with the topic. Some ideas:*
>
>* *Server hosting*
>* *Traditional or classic server architecture*
>* *Self-hosting Redwood*
>* *Hosting Redwood without serverless or docker containers*
>
>***[David Price](https://github.com/thedavidprice)*** - *[Add serverfull hosting recipe to cookbook - (October 26, 2020)](https://github.com/redwoodjs/redwoodjs.com/pull/431#issuecomment-716780610)*

## Phase 3 - Render cause I want only one dashboard

>*render.com looks the same like netlify to me but comes with a way of having postgresql database, is that not a better choice in the tutorial section as deployment target?*
>
>***[Guled](https://github.com/guledali)*** - *[Using render.com instead of Netlify and Heroku (June 10, 2020)](https://community.redwoodjs.com/t/using-render-com-instead-of-netlify-and-heroku/728)*

As more Redwood apps have been deployed to production, we've seen developers place a great emphasis on having their:
* Redwood API running on a server
* Build process and infrastructure provisioning defined in code
* Frontend, API, and database all hosted on the same provider

### api-server

>*RedwoodJS is architected as monorepo with two separate sides, an “api side” and a “web side.” The api side is a NodeJS application and the web side is a static SPA (single page app).*
>
>*This works out great when hosting on Render because hosting a static site is free, and static content is served via super-fast CDN. So we’re going to setup two sides on Render.com, but first you need to add some things to your RedwoodJS project.*
>
>*Install the `api-server` in your api side.*

```bash
cd api
yarn add @redwoodjs/api-server
```

>*Create a “health” function used by Render to check if your api side is running correctly.*

```javascript
// api/src/functions/healthz.js

export const handler = async () => {
    return {
        statusCode: 200
    }
}
```

>*On Render, create a new "web service":*
>
>* *Connect to your repository*
>* *Environment is `Node`*
>* *Build: `yarn && yarn rw prisma migrate deploy && yarn rw build api`*
>* *Start: `cd api && yarn rw-api-server --port 80`*
>* *Use custom domain: “api.raccoon.trade”*
>
>*On Render create a new "static site":*
>
>* *Build: `yarn && yarn rw build web`*
>* *Publish: `./web/dist`*
>* *Add rewrite and redirect rules*
>
>![redirect-and-rewrite-rules](https://rwjs-discourse.nyc3.cdn.digitaloceanspaces.com/original/2X/6/6727b61158eb4f0ee4ebee84248d0c22e9332673.png)
>
>***[Peter Pistorius](https://github.com/peterp)*** - *[Using render.com instead of Netlify and Heroku (March 15, 2021)](https://community.redwoodjs.com/t/using-render-com-instead-of-netlify-and-heroku/728/4)*

### yarn rw setup deploy render

>*Perfect. We’ll set up a render.yaml and make this easier. Stay tuned!*
>
>***[Anurag Goel](https://github.com/anurag)*** - *[Using render.com instead of Netlify and Heroku (March 15, 2021)](https://community.redwoodjs.com/t/using-render-com-instead-of-netlify-and-heroku/728/5)*

>*The PR adds a command to the Redwood CLI to automatically configure project for deployment to Render.*
>
>***[Sean Doughty](https://github.com/SEANDOUGHTY)*** - *[Adding Setup Deploy Render Command (March 25, 2021)](https://github.com/redwoodjs/redwood/pull/2099)*

```bash
yarn rw setup deploy render
```

```yaml
services:
- name: history-hosting-test-web
  type: web
  env: static
  buildCommand: yarn rw deploy render web
  staticPublishPath: ./web/dist
  envVars:
  - key: NODE_VERSION
    value: 14
  routes:
  - type: rewrite
    source: /.redwood/functions/*
    destination: replace_with_api_url/*
  - type: rewrite
    source: /*
    destination: /index.html

- name: history-hosting-test-api
  type: web
  env: node
  region: oregon
  buildCommand: yarn && yarn rw build api
  startCommand: yarn rw deploy render api
  envVars:
  - key: NODE_VERSION
    value: 14
  - key: DATABASE_URL
    fromDatabase:
      name: history-hosting-test-db
      property: connectionString

databases:
  - name: history-hosting-test-db
    region: oregon
```

## Phase 4 - Still in progress

Is this deployment machine truly universal? That depends of course on your definition of universality. If you want to universally deploy to AWS, you're covered. However, other major cloud providers have proved more difficult.

### Google Cloud Run

>*Adding setup and deploy functionality for firebase hosting for web, with Cloud Run for API.*
>
>***[Benjamin Coe](https://github.com/bcoe)*** - *[Adding support for GCP project generation (September 26, 2020)](https://github.com/redwoodjs/redwood/pull/1225)*

### Azure

>*I'm not sure which direction this will take ([The Serverless Framework](https://www.serverless.com/) vs maintaining deployment providers in Redwood) but here is some Azure specific on the topic.*
>
>*Azures equivalent to AWS's Lambda and S3 is [Azure Functions](https://azure.microsoft.com/en-us/services/functions/) and [Azure Storage](https://azure.microsoft.com/en-us/services/storage/). If the deployment would utilize Git deployment to the fullest, would there be any need for a storage account/bucket?*
>
>***[Johan Eliasson](https://github.com/jeliasson)*** - *[Support Deployments to AWS Lambda (July 7, 2020)](https://github.com/redwoodjs/redwood/issues/431#issuecomment-654931795)*

#### Azure Static Web Apps

>*I've been tinkering Azure Static Web Apps, which would not only host the API side (via Azure Functions), but also the front-end. I don't think function-only deployment would be too far off though.*
>
>***[Kim-Adeline Miguel](https://github.com/kimadeline)*** - *[Deployment Targets tracking issue (September 7, 2020)](https://github.com/redwoodjs/redwood/issues/947#issuecomment-688554760)*

#### Azure Functions

>*I am working on hacking together a solution to run Redwood on Azure using Azure Functions.*
>
>*My approach so far is to vendor the `API` package into a Redwood project: [`redwood-azure`](https://github.com/t-eckert/redwood-azure) and replace the code that generates Lambdas with code that generates Azure Functions. I have started this process by unpacking and attempting to grok some of the `API` package in [my own fork](https://github.com/t-eckert/redwood/tree/t-eckert/bluesky) of this repo. The branch is called "bluesky" because "Azure" ha ha.*
>
>*If I can get my hacked and vendored API package working in the Redwood solution, then I can approach the core maintainers with questions of architecture and implementation. AFAIK, there will have to be some flag to tell the API package whether it should compile to Azure Functions or Lambdas.*
>
>*With the final implementation, I want to limit the variance between the Lambda producing code and the Functions producing code. This will require me to gain a better understanding of the `API` package.*
>
>*Still early days.*
>
>*[Here is a link](https://github.com/t-eckert/redwood-azure/tree/t-eckert/hack-api/api/hack-api) to the beginning of my vendored "hack-api".*
>
>***[Thomas Eckert](https://github.com/t-eckert)*** - *[Deployment Targets tracking issue (April 6, 2021)](https://github.com/redwoodjs/redwood/issues/947#issuecomment-814581390)*

### Cloudflare

>*Can I deploy the redwood app to the cloudflare page?*
>
>***[Muhammad Rusdi](https://github.com/muhrusdi)*** - *[Deploy Redwoodjs with Cloudflare Page (March 3, 2021)](https://github.com/redwoodjs/redwood/issues/1899/)*

Such a simple question with such a long and complicated answer.

A [Cloudflare Worker](https://blog.cloudflare.com/introducing-cloudflare-workers/) is JavaScript you write that runs on Cloudflare's edge. A [Cloudflare Service Worker](https://blog.cloudflare.com/cloudflare-workers-unleashed/) is specifically a worker which handles HTTP traffic and is written against the Service Worker API. [Cloudflare Pages](https://pages.cloudflare.com/) is a deployment platform for frontend developers modeled on Netlify and Vercel.

#### Sounds cool but remember that part about Redwood being a Node app?

>*Hi [@muhrusdi](https://github.com/muhrusdi). To the best of my knowledge, no one has tried to deploy Redwood on Cloudflare. Personally, I am a big fan of their products. However, their alternative to AWS Lambdas, which is Cloudflare Workers, is a [V8 runtime](https://developers.google.com/apps-script/guides/v8-runtime), which is *not* the same as a Node.js runtime.*
>
>*This means the Redwood API will most likely not run on Cloudflare Workers. In the case of a very simple app, it might be fine. But once you add things like Prisma Client it's almost sure to fail.*
>
>*If you wanted to run an experiment, you could try to deploy Redwood's Web side to Cloudflare's CDN and the API site directly to AWS Lambdas using [these instructions](https://redwoodjs.com/docs/deploy#aws-serverless-deploy) for a Redwood AWS deploy target.*
>
>***[David Price](https://github.com/thedavidprice)*** - *[Deploy Redwoodjs with Cloudflare Page (March 4, 2021)](https://github.com/redwoodjs/redwood/issues/1899/#issuecomment-791183928)*

#### Prisma is currently out of the question

>*The potential gotchas that keep me from trying:*
>- *Prisma binary hates v8 runtime*
>- *Some random dependency in `@redwoodjs/api` hates v8 runtime*
>- *Auth provider packages hate v8 runtime*
>- *Misc API package dependencies used by random Redwood Project X hate v8 runtime*
>
>*In general, I foresee it being frustrating if my local dev environment spins up the API in a Node runtime so it's not until I get to deployment that things break, which is then difficult to diagnose.*
>
>*So I'd suggest starting with local dev for experimentation --> I have ZERO knowledge here:*
>
>- *Is it possible to spin up a local v8 runtime for dev?*
>- *Does babel/esbuild support it?*
>
>*If we can't do something like this, then I'm not optimistic about creating a workable, um, workflow __[Editor's note: lol]__ regardless of being able to spin up a test app on Cloudflare using Workers.*
>
>![Alt Text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4jx0j0u7uai6542fu19t.png)
>
>*[Workers do NOT support Node.js.](https://developers.cloudflare.com/workers/learning/how-workers-works)*
>
>*I believe we can transpile code to the JavaScript version we need. But we can't control the other dependencies (and their dependencies), which may or may not have Node.js specific requirements + dependencies.*
>
>*[I'll be cheering you on. But here's your first roadblock.](https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference#binarytargets-options)*
>
>***[David Price](https://github.com/thedavidprice)*** - *[Running RedwoodJS on Cloudflare Workers (April 7, 2021)](https://community.redwoodjs.com/t/running-redwoodjs-on-cloudflare-workers/2013/)*

#### WASM, it practically cures cancer!

>*You can specify a web worker output target. But you're not wrong about various dependencies using non-web worker-friendly methods. An immediate example I can give you there is validation libraries on workers.*
>
>*Wrangler won't currently support a WASM build target, and workers don't currently support TCP - sooooooooo, [this is a hard blocker for anything Prisma related.](https://github.com/prisma/prisma/issues/85)*
>
>***[Matt Sutkowski](https://github.com/msutkowski)*** - *[Running RedwoodJS on Cloudflare Workers (April 7, 2021)](https://community.redwoodjs.com/t/running-redwoodjs-on-cloudflare-workers/2013/)*

#### Deploy just Redwood-Worker-Functions to Cloudflare?

>*So maybe we have limitations. BUT it would still be rad to know what Workers are good at and what we __could__ use them for. Imagine 🌈 a Redwood App that takes advantage of a hybrid cloud infrastructure:*
>
>- *Cloudflare CDN + Networking (DNS)*
>- *Specific Redwood-Worker-Functions deployed to Cloudflare*
>- *Other parts of API deployed to AWS Fargate container for persistence*
>
>*I don't know __why__ we'd do this. But it's possible. `¯\_(ツ)_/¯`*
>
>*I do think there's a lot of value we could unlock in Cloudflare Key/Value and Durable Objects when it comes to performance + caching. Just don't know what the ROI would be on trying to figure out how to piece it all together.*
>
>***[David Price](https://github.com/thedavidprice)*** - *[Running RedwoodJS on Cloudflare Workers (April 7, 2021)](https://community.redwoodjs.com/t/running-redwoodjs-on-cloudflare-workers/2013/)*

#### Cache rules everything around me

>*I too very much doubt that Prisma+Cloudflare will happen. Those two products just seem so out of sync, why globally distribute your workers, but then connect to a single database.*
>
>*I think caching is probably the immediate answer, probably on the API-level. Maybe you have an API that's exposed via GraphQL, but that's cached globally. Then... you don't really need global workers, just global varnish.*
>
>***[Peter Pistorius](https://github.com/peterp)*** - *[Running RedwoodJS on Cloudflare Workers (April 7, 2021)](https://community.redwoodjs.com/t/running-redwoodjs-on-cloudflare-workers/2013/)*