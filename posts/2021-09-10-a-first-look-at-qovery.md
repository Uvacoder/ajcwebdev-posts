---
title: a first look at qovery
description: Qovery is a CaaS (Container as a Service) platform for deploying fullstack applications to the Cloud.
date: 2021-09-10
tags:
  - qovery
  - docker
  - aws
  - cloud
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t8328tvmjb6wtopbmzh1.png
layout: layouts/post.njk
---

[Qovery](https://www.qovery.com/) is a CaaS (Container as a Service) platform for deploying fullstack applications to the Cloud with your own account on AWS, GCP, Azure, and Digital Ocean. It syncs to your git repository, detects your Dockerfile, and can integrate with a variety of [open source web frameworks](https://hub.qovery.com/guides/tutorial).

The code for this article can be found [on my GitHub](https://github.com/ajcwebdev/ajcwebdev-qovery).

## Setup Qovery Account

Create a Qovery account at the following [link](https://start.qovery.com/).

### Install Qovery CLI

Install instructions will vary depending on whether you are using MacOS, Linux, or Windows. I will be using the MacOS instructions with [Homebrew](https://brew.sh/), see [this](https://hub.qovery.com/docs/using-qovery/interface/cli/) link for other operating systems.

```bash
brew tap Qovery/qovery-cli
brew install qovery-cli
```

### Login to Qovery account with `qovery auth`

```bash
qovery auth
```

This will open your browser and ask for authentication through GitHub or GitLab.

## Create project

Start with a blank application and initialize a `package.json`.

```bash
mkdir ajcwebdev-qovery
cd ajcwebdev-qovery
npm init -y
npm i express
touch index.js Dockerfile .dockerignore
echo 'node_modules\n.DS_Store' > .gitignore
```

### `index.js`

Include the following code inside `index.js` to return an HTML snippet.

```js
// index.js

const express = require("express")
const app = express()

const PORT = 8080
const HOST = '0.0.0.0'

app.get('/', (req, res) => {
  res.send('<h2>ajcwebdev-qovery</h2>')
})

app.listen(PORT, HOST)
console.log(`Running on http://${HOST}:${PORT}`)
```

### Start up server

Run `node index.js` to start the server and open [localhost:8080](http://localhost:8080/).

```bash
node index.js
```

![01-ajcwebdev-qovery-localhost-8080](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/34pe0hedrqeertf5zcc4.png)

### `Dockerfile`

```dockerfile
FROM node:14-alpine
LABEL org.opencontainers.image.source https://github.com/ajcwebdev/ajcwebdev-docker
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm i
COPY . ./
EXPOSE 8080
CMD [ "node", "index.js" ]
```

### `.dockerignore`

```
node_modules
Dockerfile
.dockerignore
.git
.gitignore
npm-debug.log
```

### Connect to GitHub repository

```bash
git init
git add .
git commit -m "I'd make a qovery joke but I have no idea what the name means"
gh repo create ajcwebdev-qovery
git push -u origin main
```

## Setup Qovery Project

### Create a project

![02-qovery-projects](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n7xrq7j5fk4okcim0isl.png)

Create a new project.

![03-create-new-project](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/x6fvfqp5bqdb6j1ut7pc.png)

### Create an environment

![04-qovery-environments](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zel2phopee02e805wklx.png)

Create a new environment called `dev`.

![05-create-new-environment](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ewpwwe8f4se2alyy7z32.png)

### Create an app

![06-qovery-apps](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/esigyhgyhbg2dnzmw6ii.png)

Connect your GitHub repo and set the branch to `main`.

![07-create-application](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g1313k0pkbwwjs5ivlpt.png)

### Application Dashboard

![08-express-server-dashboard](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/om6ywsrhnocg10ixgcs5.png)

Set the port to 8080.

![09-set-port-to-8080](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/uqku0sfp6o9o4di2jslc.png)

Deploy your application with the action button.

![10-deploy-action](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/j6la440rm6i4xvgnkdrg.png)

Click actions and select open to see your [running application](https://z2622bffa-zb0c2091c-gtw.qovery.io/).

![11-deployed-express-server](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wsq0kvnep48exlq1947r.png)

### Check application context with `qovery context`

The `context` command lets you configure the CLI to work with your chosen application. Before executing other commands, you need first to set up the context. The context is then remembered and used by the CLI.

```bash
qovery context
```

```
qovery context
Qovery: Current context:
Context not yet configured. 

Qovery: You can set a new context using 'qovery context set'.
```

### Configure a new context with `qovery context set`

```bash
qovery context set
```

```
Qovery: Current context:
Context not yet configured. 

Qovery: Select new context
Organization:
✔ ajcwebdev
Project:
✔ ajcwebdev-first-project
Environment:
✔ dev
Application:
✔ express-server

Qovery: New context:
Organization | ajcwebdev              
Project      | ajcwebdev-first-project
Environment  | dev                    
Application  | express-server
```

### Check your application logs with `qovery log`

```bash
qovery log
```

```
TIME                    MESSAGE                         
Sep  9 23:22:26.246670  Running on http://0.0.0.0:8080  
```

### Check the status of your application with `qovery status`

```bash
qovery status
```

```
Application    | Status 
express-server | RUNNING
```