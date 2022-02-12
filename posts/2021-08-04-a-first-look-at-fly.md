---
title: a first look at fly
description: Fly is a platform for full stack applications and databases that need to run globally.
date: 2021-08-04
tags:
  - fly
  - docker
  - containers
  - devops
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m5vtt1wa4ey8ips4mzrh.jpeg
layout: layouts/post.njk
---

[Fly](https://fly.io/) is a platform for full stack applications and databases that need to run globally. Fly executes your code close to users and scales compute in cities where your app is busiest. You can run arbitrary Docker containers and host popular databases like Postgres.

## Outline

* [Fly Setup](#fly-setup)
  * [Install flyctl](#install-flyctl)
  * [Create Fly Account](#create-fly-account)
  * [Login to Fly Account](#login-to-fly-account)
* [Create Project](#create-project)
  * [Create Server](#create-server)
  * [Run Server](#run-server)
  * [Create Dockerfile](#create-dockerfile)
* [Launch Application on Fly](#launch-application-on-fly)
  * [Deploy Application](#deploy-application)
  * [Show Current Application Status](#show-current-application-status)

## Fly Setup

The code for this example can be found [on my GitHub](https://github.com/ajcwebdev/ajcwebdev-fly).

### Install flyctl

You can download the CLI on [Mac, Linux, or Windows](https://fly.io/docs/getting-started/installing-flyctl/).

```bash
brew install superfly/tap/flyctl
```

### Create Fly Account

If you are a new user you can create an account with `flyctl auth signup`.

```bash
flyctl auth signup
```

After your browser opens you can either:

* Sign-up with your name, email and password.
* Sign-up with GitHub and check your email for link to set a password for verification.

You will also be prompted for credit card payment information, required for charges outside the free plan on Fly. See [Pricing](https://fly.io/docs/about/pricing) for more details.

### Login to Fly Account

If you already have an account you can login with `flyctl auth login`.

```bash
flyctl auth login
```

After your browser opens, sign in with your username and password. If you signed up with Github, use the Sign-in with Github button to sign in.

## Create Project

```bash
mkdir ajcwebdev-fly
cd ajcwebdev-fly
npm init -y
npm i express --save
touch index.js Dockerfile .dockerignore
```

### Create Server

```javascript
const express = require("express")
const app = express()

const port = process.env.PORT || 3000

app.get(
  "/", (req, res) => {
    greeting = "<h1>ajcwebdev-fly</h1>"
    res.send(greeting)
  }
)

app.listen(
  port,
  () => console.log(`Hello from port ${port}!`)
)
```

### Run Server

```bash
node index.js
```

```
Hello from port 3000!
```

![01-hello-world-localhost](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/z6dt6up0b24u2gu45fsc.png)

### Create Dockerfile

Add the following code to `Dockerfile` in the root directory of your project.

```dockerfile
FROM node:14-alpine
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm i
COPY . ./
EXPOSE 8080
CMD [ "node", "index.js" ]
```

Add the following code to `.dockerignore` in the same directory as your `Dockerfile`.

```
node_modules
Dockerfile
.dockerignore
.git
.gitignore
npm-debug.log
```

## Launch Application on Fly

Run `flyctl launch` in the directory with your source code to configure your app for deployment. This will create and configure a fly app by inspecting your source code and prompting you to deploy.

```bash
flyctl launch --name ajcwebdev-fly --region sjc
```

```
Creating app in /Users/ajcwebdev/ajcwebdev-fly
Scanning source code
Detected Dockerfile app
Automatically selected personal organization: Anthony Campolo
Created app ajcwebdev-fly in organization personal
Wrote config file fly.toml
Your app is ready. Deploy with `flyctl deploy`
? Would you like to deploy now? No
```

This creates a `fly.toml` file.

```toml
app = "ajcwebdev-fly"

kill_signal = "SIGINT"
kill_timeout = 5

[env]

[experimental]
  allowed_public_ports = []
  auto_rollback = true

[[services]]
  http_checks = []
  internal_port = 8080
  protocol = "tcp"
  script_checks = []

[services.concurrency]
  hard_limit = 25
  soft_limit = 20
  type = "connections"

[[services.ports]]
  handlers = ["http"]
  port = 80

[[services.ports]]
  handlers = ["tls", "http"]
  port = 443

[[services.tcp_checks]]
  grace_period = "1s"
  interval = "15s"
  restart_limit = 6
  timeout = "2s"
```

Add the following `PORT` number under `env`.

```toml
[env]
  PORT = 8080
```

### Deploy Application

```bash
flyctl deploy
```

```
Deploying ajcwebdev-fly
==> Validating app configuration
--> Validating app configuration done

Services
TCP 80/443 ⇢ 8080
Waiting for remote builder fly-builder-old-sun-2915... connecting
Creating WireGuard peer "interactive-Anthonys-MacBook-Pro-anthony-stepzen-com-783" in region "iad" for organization personal

Remote builder fly-builder-old-sun-2915 ready
==> Creating build context
--> Creating build context done
==> Building image with Docker
Sending build context to Docker daemon  6.034kB

Step 1/7 : FROM node:14-alpine
14-alpine: Pulling from library/node
Digest: sha256:0c80f9449d2690eef49aad35eeb42ed9f9bbe2742cd4e9766a7be3a1aae2a310
Status: Downloaded newer image for node:14-alpine
 ---> d93b35a67404

Step 2/7 : WORKDIR /usr/src/app
 ---> Running in 81218d388a5e
 ---> 2ae716ba5525

Step 3/7 : COPY package*.json ./
 ---> ad19515d5299

Step 4/7 : RUN npm i
 ---> Running in 41102030018c
npm WARN ajcwebdev-fly@1.0.0 No description
npm WARN ajcwebdev-fly@1.0.0 No repository field.

added 50 packages from 37 contributors and audited 50 packages in 1.923s

found 0 vulnerabilities
 ---> c9f0004b1f7f

Step 5/7 : COPY . ./
 ---> 05927a4fb537

Step 6/7 : EXPOSE 8080
 ---> Running in 5a3069e1bc22
 ---> a3d733fda688

Step 7/7 : CMD [ "node", "index.js" ]
 ---> Running in 5e1c8548aed1
 ---> 3dff2152628b

Successfully built 3dff2152628b
Successfully tagged registry.fly.io/ajcwebdev-fly:deployment-1628129385
--> Building image done

==> Pushing image to fly
The push refers to repository [registry.fly.io/ajcwebdev-fly]
deployment-1628129385: digest: sha256:467d26b125b95ad397ff1258a0cf94d7cccd72e63cfd153f8632092164e0f9d7 size: 1992
--> Pushing image done

Image: registry.fly.io/ajcwebdev-fly:deployment-1628129385
Image size: 120 MB

==> Creating release
Release v0 created

You can detach the terminal anytime without stopping the deployment

Monitoring Deployment

1 desired, 1 placed, 1 healthy, 0 unhealthy
[health checks: 1 total, 1 passing]

--> v0 deployed successfully
```

### Show Current Application Status

Status includes application details, tasks, most recent deployment details and in which regions it is currently allocated.

```bash
flyctl status
```

```
App
  Name     = ajcwebdev-fly       
  Owner    = personal                    
  Version  = 0                           
  Status   = running                     
  Hostname = ajcwebdev-fly.fly.dev  

Deployment Status
  ID          = d94d886a-f338-28cf-4078-1ed838eea224         
  Version     = v0                                           
  Status      = successful                                   
  Description = Deployment completed successfully            
  Instances   = 1 desired, 1 placed, 1 healthy, 0 unhealthy  

Instances
ID       VERSION REGION DESIRED STATUS  HEALTH CHECKS      RESTARTS CREATED 
40591407 0       sjc    run     running 1 total, 1 passing 0        37s ago
```

Open your browser to current deployed application with `flyctl open`.

```bash
flyctl open
```

```
Opening http://ajcwebdev-fly.fly.dev/
```

Visit [ajcwebdev-fly.fly.dev](http://ajcwebdev-fly.fly.dev/) to see the site.

![02-ajcwebdev-fly-dev](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/c8z7js20jh2r8ld9degx.png)