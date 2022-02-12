---
title: deploying node.js on digital ocean with pm2
description: In this tutorial we will deploy a Node.js application on Digital Ocean with PM2.
date: 2021-06-12
tags:
  - javascript 
  - node
  - pm2
  - linux
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qzw1l3ib8si1cjmbj2qp.jpeg
layout: layouts/post.njk
---

Serverless deployment is becoming easier and easier every year, but there will always be a subset of use cases that require a persistent, running server. Projects with stricter requirements around performance, computation, storage, concurrency, and isolation may opt for a more traditional deployment strategy and host a Linux server.

In this tutorial we will deploy a Node.js application on Digital Ocean with [PM2](https://pm2.keymetrics.io/). PM2 is a production process manager for Node.js applications. It contains a built-in load balancer that allows you to keep applications alive indefinitely and it can also reload applications without downtime.

## Outline

* [Create Node App with PM2](#create-node-app-with-pm2)
  * [Create HTTP Server](#create-http-server)
  * [Start Server on Localhost](#start-server-on-localhost)
  * [Configure Node App for PM2](#configure-node-app-for-pm2)
  * [Create GitHub Repository](#create-github-repository)
* [Deploy Linux Server on Digital Ocean Droplet](#deploy-linux-server-on-digital-ocean-droplet)
  * [Setup SSH Keys](#setup-ssh-keys)
  * [Generate an RSA Key Pair](#generate-an-rsa-key-pair)
  * [Create a Password](#create-a-password)
  * [Copy Key to the Clipboard](#copy-key-to-the-clipboard)
  * [Choose a Hostname](#choose-a-hostname)
  * [Login to Server from Terminal](#login-to-server-from-terminal)
  * [Enter Password](#enter-password)
* [Install Server Dependencies and Start Server](#install-server-dependencies-and-start-server)
  * [Install Node](#install-node)
  * [Clone GitHub Repository and Install Node Modules](#clone-github-repository-and-install-node-modules)
  * [Start App as a Process with PM2](#start-app-as-a-process-with-pm2)

## Create Node App with PM2

We will generate a minimal Node.js application. The only dependency we will install is `pm2`.

```bash
mkdir ajcwebdev-pm2
cd ajcwebdev-pm2
yarn init -y
yarn add pm2
touch index.js
echo 'node_modules\n.DS_Store' > .gitignore
```

If we look at our `package.json` file in the root of our project we will see:

```json
{
  "name": "ajcwebdev-pm2",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "dependencies": {
    "pm2": "^5.1.2"
  }
}
```

### Create HTTP Server

`index.js` will return a header and paragraph tag.

```javascript
// index.js

const http = require('http')

const port = process.env.PORT || 8080

const server = http.createServer((req, res) => {
  res.statusCode = 200
  res.setHeader('Content-Type', 'text/html')
  res.write('<title>ajcwebdev-pm2</title>')
  res.write('<h1>ajcwebdev-pm2</h1>')
  res.end('<p>PM2 is a daemon process manager</p>')
})

server.listen(port, () => {
  console.log(`Server running on Port ${port}`)
})
```

### Start Server on Localhost

Enter the following command to start your development server and see your project.

```bash
node index.js
```

The file is served to `localhost:8080`. You should see the following message in your terminal.

```
Server running on Port 8080
```

Open [localhost:8080](http://localhost:8080) to see your application.

![01-node-app-running-on-localhost-8080](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8t6inu8sews6r7g5f1g9.png)

### Configure Node App for PM2

Create a PM2 ecosystem configuration file.

```bash
yarn pm2 init
```

Terminal output:

```
File /Users/ajcwebdev/ajcwebdev-pm2/ecosystem.config.js generated
```

Open the newly created `ecosystem.config.js` file.

```javascript
// ecosystem.config.js

module.exports = {
  apps : [{
    script: 'index.js',
    watch: '.'
  }, {
    script: './service-worker/',
    watch: ['./service-worker']
  }],

  deploy : {
    production : {
      user : 'SSH_USERNAME',
      host : 'SSH_HOSTMACHINE',
      ref  : 'origin/master',
      repo : 'GIT_REPOSITORY',
      path : 'DESTINATION_PATH',
      'pre-deploy-local': '',
      'post-deploy' : 'npm install && pm2 reload ecosystem.config.js --env production',
      'pre-setup': ''
    }
  }
}
```

We'll make a few adjustments to the `apps` object.

```javascript
// ecosystem.config.js

module.exports = {
  apps : [{
    name: "ajcwebdev-pm2",
    script: "./index.js",
    env: {
      NODE_ENV: "development",
    },
    env_production: {
      NODE_ENV: "production",
    }
  }]
}
```

### Create GitHub Repository

Create a [repository on GitHub](https://github.com/new) or use the `gh repo create` command with the [GitHub CLI](https://cli.github.com/).

![02-blank-github-repository](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m3ajhvpliu1kupftqc2r.png)

In your Node project initialize a Git repository.

```bash
git init
git add .
git commit -m "Cause I need a server"
```

Create a new repository, set the remote name from the current directory, and push the project to the newly created repository.

```bash
gh repo create ajcwebdev-pm2 \
  --public \
  --source=. \
  --remote=upstream \
  --push
```

Verify that your project was [pushed to main](https://github.com/ajcwebdev/ajcwebdev-pm2.git).

![03-github-repository-with-project](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/32pqvee9otn8fqlvwn8g.png)

That was the easy part. [Here be servers](https://en.wikipedia.org/wiki/Here_be_dragons).

## Deploy Linux Server on Digital Ocean Droplet

There are many ways to host a Linux server, if you are comfortable with other providers you should be able to host this example project essentially anywhere you can host a Node server. We will create an account on [Digital Ocean](https://cloud.digitalocean.com/) which provides $100 of free credits to get started.

![04-digital-ocean-project](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3as63xv6bvjri3amrlal.png)

Click "Get Started with a Droplet" to get started with a droplet.

![05-choose-an-image-and-choose-a-plan](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/f02so4301f5qjly4etrn.png)

Select [Ubuntu 21.04 x64](https://releases.ubuntu.com/21.04/) and the [Shared CPU plan](https://docs.digitalocean.com/products/droplets/resources/choose-plan/#shared-vs-dedicated).

![06-choose-cpu-options](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gaaldab9d6fldu5um3yp.png)

Select the cheapest option, Regular Intel with SSD and $5 a month.

![07-choose-a-datacenter-region](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5ikop7thqztmxomlxr5l.png)

We do not need block storage. Pick the datacenter region closest to your location.

![08-authentication-ssh-keys](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pd79seysc2v1xt92z88i.png)

### Setup SSH Keys

Click "New SSH key" to enter a new SSH key.

![09-add-public-ssh-key](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ftuubtt4ehscefgx79te.png)

SSH keys provide a more secure way of logging into a virtual private server than using a password alone.

### Generate an RSA Key Pair

There are several ways to use SSH; one is to use automatically generated public-private key pairs to simply encrypt a network connection, and then use password authentication to log on.

Another is to use a manually generated public-private key pair to perform the authentication, allowing users or programs to log in without having to specify a password.

```bash
ssh-keygen
```

Terminal output:

```
Generating public/private rsa key pair.
```

SSH is an authentication method used to gain access to an encrypted connection between systems with the intent of managing or operating the remote system.

SSH keys are 2048 bits by default. This is generally considered to be good enough for security, but if you think your 13 line JavaScript project might be a target for [Advanced persistent threats](https://en.wikipedia.org/wiki/Advanced_persistent_threat) you can include the `-b` argument with the number of bits you would like such as `ssh-keygen -b 4096`.

```
Enter file in which to save the key (/Users/ajcwebdev/.ssh/id_rsa): 
```

This prompt allows you to choose the location to store your RSA private key. Press ENTER to leave the default which stores them in the `.ssh` hidden directory in your user’s home directory.

### Create a Password

```
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

Terminal output:

```
Your identification has been saved in
/Users/ajcwebdev/.ssh/id_rsa

Your public key has been saved in
/Users/ajcwebdev/.ssh/id_rsa.pub

The key fingerprint is:
SHA256:s9sV2rydQ6A4FtVgq2fckCFu7fZbYAhamXnUR/7SVNI ajcwebdev@macbook.local

The key's randomart image is:
+---[RSA 3072]----+
|.oO.o   . ...    |
| = B + o o oE    |
|  = = . = =      |
| o = o + * o     |
|  = . + S = .    |
|   . o . O       |
|      o + o .    |
|       o +oo     |
|        +oo+.    |
+----[SHA256]-----+
```

### Copy Key to the Clipboard

```bash
pbcopy < ~/.ssh/id_rsa.pub
```

Paste the key into the SSH key content input and `id_rsa.pub` for the name input.

### Choose a Hostname

![10-finalize-and-create-choose-hostname](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yj6x62ualdd5gfobl8wo.png)

In a minute or so your server will be created and deployed.

![11-digital-ocean-server](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/odmts2bzlyt0dip1oqf9.png)

### Login to Server from Terminal

The username is `root` and the password is whatever you used when you created your server.

```bash
ssh root@144.126.219.200
```

### Enter Password

```
Enter passphrase for key '/Users/ajcwebdev/.ssh/id_rsa':
```

## Install Server Dependencies and Start Server

When you provision a Digital Ocean droplet or other common Linux based virtual machines, it is likely that the server does not include Node by default. Since the purpose of this tutorial is to deploy a Node application from scratch, we have chosen a fresh Linux box that needs to have Node installed. However, because of its ubiquity in web development, many hosting providers include the ability to provision a server with Node pre-installed.

### Install Node

Let’s begin by installing the latest LTS release of Node.js, using the [NodeSource](https://github.com/nodesource/distributions) package archives. First, install the NodeSource Personal Package Archive in order to get access to its contents. Use `curl` to retrieve the installation script for Node 12.

```bash
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```

Install Node with `apt-get` and check Node version.

```bash
sudo apt-get install -y nodejs
node -v
```

Terminal output:

```
v12.22.1
```

Install Yarn with `apt-get` and check Yarn version.

```bash
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/yarnkey.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update && sudo apt-get install yarn
yarn -v
```

Terminal output:

```
1.22.17
```

### Clone GitHub Repository and Install Node Modules

```bash
git clone https://github.com/ajcwebdev/ajcwebdev-pm2.git
cd ajcwebdev-pm2
yarn
```

### Start App as a Process with PM2

```bash
yarn pm2 start index.js
```

```
[PM2] Spawning PM2 daemon with pm2_home=/root/.pm2
[PM2] PM2 Successfully daemonized
[PM2] Starting /root/ajcwebdev-pm2/index.js in fork_mode (1 instance)
[PM2] Done.
┌─────┬──────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ id  │ name     │ namespace   │ version │ mode    │ pid      │ uptime │ ↺    │ status    │ cpu      │ mem      │ user     │ watching │
├─────┼──────────┼─────────────┼─────────┼─────────┼──────────┼────────┼──────┼───────────┼──────────┼──────────┼──────────┼──────────┤
│ 0   │ index    │ default     │ 1.0.0   │ fork    │ 15233    │ 0s     │ 0    │ online    │ 0%       │ 30.1mb   │ root     │ disabled │
└─────┴──────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┴──────────┴──────────┴──────────┴──────────┘
```

Display the application’s log with `pm2 log`.

```bash
yarn pm2 log
```

```
[TAILING] Tailing last 15 lines for [all] processes (change the value with --lines option)

/root/.pm2/pm2.log last 15 lines:

PM2        | 2021-12-27T06:25:30: PM2 log: PM2 version          : 5.1.2
PM2        | 2021-12-27T06:25:30: PM2 log: Node.js version      : 12.22.8
PM2        | 2021-12-27T06:25:30: PM2 log: Current arch         : x64
PM2        | 2021-12-27T06:25:30: PM2 log: PM2 home             : /root/.pm2
PM2        | 2021-12-27T06:25:30: PM2 log: PM2 PID file         : /root/.pm2/pm2.pid
PM2        | 2021-12-27T06:25:30: PM2 log: RPC socket file      : /root/.pm2/rpc.sock
PM2        | 2021-12-27T06:25:30: PM2 log: BUS socket file      : /root/.pm2/pub.sock
PM2        | 2021-12-27T06:25:30: PM2 log: Application log path : /root/.pm2/logs
PM2        | 2021-12-27T06:25:30: PM2 log: Worker Interval      : 30000
PM2        | 2021-12-27T06:25:30: PM2 log: Process dump file    : /root/.pm2/dump.pm2
PM2        | 2021-12-27T06:25:30: PM2 log: Concurrent actions   : 2
PM2        | 2021-12-27T06:25:30: PM2 log: SIGTERM timeout      : 1600
PM2        | 2021-12-27T06:25:30: PM2 log: ===============================================================================
PM2        | 2021-12-27T06:25:30: PM2 log: App [index:0] starting in -fork mode-
PM2        | 2021-12-27T06:25:30: PM2 log: App [index:0] online

/root/.pm2/logs/index-error.log last 15 lines:
/root/.pm2/logs/index-out.log last 15 lines:
0|index    | Server running on Port 8080
```

Open [144.126.219.200:8080](http://144.126.219.200:8080).

![12-node-app-running-on-digital-ocean-8080](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/l3wjosoagi015yfdckry.png)