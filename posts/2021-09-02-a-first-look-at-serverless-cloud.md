---
title: a first look at serverless cloud
description: Serverless Cloud is a new serverless app platform from Serverless, Inc. Unlike the Serverless Framework, it lives on a new hosting service in the cloud instead of deploying your application directly to AWS.
date: 2021-09-02
tags:
  - serverless
  - cloud
  - hosting
  - deployment
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fgek5skr4ix79m6hbwao.png
layout: layouts/post.njk
---

[Serverless Cloud](https://www.serverless.com/cloud) is a new serverless app platform from [Serverless, Inc.](https://serverless.com/) Unlike the company's initial product, the [Serverless Framework](https://www.serverless.com/framework), it does not deploy your application directly to AWS. Instead, your apps are instantly deployed and live on a new hosting service in the cloud with a dashboard and real-time logs.

All the code for this project can be found [on my GitHub](https://github.com/ajcwebdev/ajcwebdev-serverless-cloud).

## Install Cloud CLI

Install `@serverless/cloud` from `npm`.

```bash
npm i -g @serverless/cloud
```

## Initialize Service

Create a blank folder on your local machine for your service's code and initialize your Serverless Cloud service with the `cloud` command.

```bash
mkdir ajcwebdev-serverless-cloud
cd ajcwebdev-serverless-cloud
cloud
```

Your browser will open automatically and log you in via the CLI or provide a login link in the terminal. Once you are connected you will be given an activation code to enter when prompted.

## Deploy to staging environment

Give you service a name and deploy it with `deploy dev` in the interactive terminal.

```bash
deploy dev
```

You can also use `cloud deploy dev` if you want to clone one of these projects from a repo and immediately deploy it.

```bash
cloud deploy dev
```

You will be given a deployed [endpoint](https://fabulous-mvp-00zr8.cloud.serverless.com/) with a sample todo app.

![01-todo-app-starter-template](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7ctt2zo38ugw1wqdixwh.png)

Add a few todos.

![02-todo-items](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/uypkzz7mg6kd6kwg4ewd.png)

## Project Code

### `package.json`

The `@serverless/cloud` package is included by default in the cloud runtime, so it does not need to be included as a dependency in `package.json`.

```json
{
  "name": "ajcwebdev-serverless-cloud",
  "version": "1.0.0",
  "description": "Serverless Cloud todo api",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "start": "cloud",
    "test": "cloud test"
  },
  "devDependencies": {
    "@jest/globals": "^27.1.0",
    "@serverless/cloud": "^0.0.22"
  },
  "serverless": {
    "org": "ajcwebdev",
    "service": "ajcwebdev-serverless-cloud"
  }
}
```

### `index.js`

We import a handful of modules from `@serverless/cloud` at the top of our `index.js` file.

```js
// index.js

import {
  api,
  data,
  schedule,
  params
} from '@serverless/cloud'
```

* `api` is used to build REST APIs.
  * `api.get` - `GET` method
  * `api.post` - `POST` method
  * `api.delete` - `DELETE` method
  * `api.use` - Middleware
* `data` is used to access Serverless Data.
  * `data.get` - Reads the data
  * `data.getByLabel` - Reads the data with a specified label
  * `data.set` - Writes the data to storage
  * `data.remove` - Deletes the data
* `schedule` is used to create scheduled tasks.
  * `schedule.every` - Runs on a specified interval of time such as every 60 minutes

### `getTodos` function

This function can be reused in different API paths to get all the todos or to get a specific todo based on its label.

```js
// index.js

const getTodos = async (status, meta) => {
  let result

  if (status === 'all') {
    result = await data.get('todo:*', meta)
  } else if (status === 'complete') {
    result =  await data.getByLabel('label1','complete:*', meta)
  } else {
    result = await data.getByLabel('label1','incomplete:*', meta)
  }

  return {
    items: result.items.map(
      item => item.value
    )
  }
}
```

### `GET` todo items - `/todos`

This function calls our `getTodos` function with the status and returns the results.

```js
// index.js

api.get('/todos', async (req, res) => {
  let result = await getTodos(
    req.query.status,
    req.query.meta ? true : {}
  )
  
  console.log(params.CLOUD_URL)

  res.send({
    items: result.items
  })
})
```

### `POST` updates to a todo item - `'/todos/:id'`

This function takes the `body` of the request and sets it to `data`. The `body` can include a `duedate`. It also includes an `id`, `createdAt` date, and `status` that can be `complete` or `incomplete`. After setting the todo, the `getTodos` query is run again on all the todos and the updated list is returned.

```js
// index.js

api.post('/todos/:id', async (req, res) => {
  console.log(new Date().toISOString())

  let body = req.body
  if (body.duedate) {
    body.duedate = new Date(body.duedate).toISOString()
  }

  await data.set(
    `todo:${req.params.id}`,
    {
      ...body,
      createdAt: Date.now()
    },
    Object.assign({},
      req.body.status ? 
        { 
          label1: body.status === 'complete'
            ? `complete:${new Date().toISOString()}` 
            : `incomplete:${body.duedate ? body.duedate : '9999' }` }
        : null
    )
  )
  
  let result = await getTodos(
    req.query.status
  )

  res.send({
    items: result.items
  })
})
```

Pay no attention to the [ternary soup](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) in `Object.assign` if it doesn't make any sense. Just try to leave it alone and don't touch it.

### `DELETE` a todo item - `'/todos/:id'`

This function deletes the todo with `data.remove` and then queries and returns the remaining todos in the list.

```js
// index.js

api.delete('/todos/:id', async (req, res) => {
  await data.remove(`todo:${req.params.id}`)
  
  let result = await getTodos(req.query.status)

  res.send({
    items: result.items
  })
})
```

### Custom error handler middleware

This function provides middleware for error handling. Errors are also streamed live to your terminal in dev mode.

```js
// index.js

api.use((err, req, res, next) => {
  console.error(err.stack)

  if (!err.statusCode) {
    err.statusCode = 500
  }

  const error = {
    name: err.name,
    statusCode: err.statusCode,
    message: err.message,
  }

  res.status(err.statusCode).json(error)
})
```

### Check for overdue todos hourly with `schedule.every`

Sometimes you might want to run code on a schedule, like if you want to send alerts when items are overdue. This function looks for items that are overdue, loops through the overdue items, and sends an alert if necessary.

```js
// index.js

schedule.every("60 minutes", async () => {
  console.log(`Checking for overdue TODOs...`)

  let overdueItems = await data.getByLabel(
    'label1',
    `incomplete:<${new Date().toISOString()}`
  )

  if (overdueItems.items.length === 0) {
    console.log(`Nothing overdue!`)
  }

  for (let item of overdueItems.items) {
    console.log(
      `ALERT: '${item.value.name}' is overdue!!!`
    )
  }
})
```

### Sample todos in `data.json`

```json
{
  "key": "todo:1",

  "value": { 
    "id": "1", 
    "name": "Deploy an amazing Serverless Cloud app",
    "status": "complete",
    "completed": "2021-07-01T12:00:00.000Z",
    "createdAt": 1627316142196
  },

  "label1": "complete:2021-07-01T012:00:00.000Z"
},
```

### Tests

There are tests. One day I'll write a test, I promise.

### Static assets

You can serve up static assets from the `static` folder. The folder currently contains:
* `assets` folder for images
* `index.html` to serve the main page
* `styles.css` for styling
* `todos.js` for all the React code so you can scare the backend developers on your team

## Modify `index.html` and deploy to production

Change the `<header>` in `index.html`.

```html
<header>
  <div>
    <h1 class="text-center">
      ajcwebdev serverless cloud
    </h1>
    
    <h3 class="grey text-center">
      Seriously, there are so few servers you wouldn't believe it
    </h3>
  </div>
</header>
```

Deploy to production with `cloud deploy prod` or `deploy prod` in the interactive terminal session.

```bash
cloud deploy prod
```

The [link](https://novel-app-fpp5w.cloud.serverless.com) will be automatically pasted to your clipboard cause having to copy links is for noobs.

![03-todo-template-edit](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/636icv33j0sgw3qtapv2.png)

## Dashboard

Since this is a cloud that means it has to have a dashboard, right? How else can I perform my ClickOps duties?

### Services

Serverless Cloud allows you to build services within your team's organization.

![04-services](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5ig6z59w4ytn0yt6v583.png)

You can create as many services as you want for different use cases or applications.

### Instances

Each instance is completely separate from all the other instances in a service and store their own copy of the data.

![05-instances](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n5y8jh2sk9e52iog95sz.png)

The environments within instances are identical, so you can ensure that your application will behave exactly the same across all of them.

### Metrics

Numbers that tell you information about stuff.

![06-dev-stage-metrics](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rtdb7om647724tbn44hl.png)

## Summary

Pretty cool. Nothing blew up, it worked as expected, and I had a deployed application in under 10 seconds.

### Resources

* [Documentation](https://www.serverless.com/cloud/docs/)
* [GitHub repository](https://github.com/serverless/cloud)