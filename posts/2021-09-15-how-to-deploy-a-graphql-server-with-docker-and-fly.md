---
title: how to deploy a graphQL server with docker and fly
description: Step by step instructions for creating a GraphQL server with Express GraphQL, building a Docker image of the server, and deploying the container to Fly.
date: 2021-09-15
tags:
  - docker
  - graphql
  - express
  - fly
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dfbqzd79c4ivhym46ovy.png
layout: layouts/post.njk
---

[Express GraphQL](https://github.com/graphql/express-graphql/) is a library for building production ready GraphQL HTTP middleware. Despite the emphasis on Express in the repo name, you can create a GraphQL HTTP server with any HTTP web framework that supports connect styled middleware. This includes [Connect](https://github.com/senchalabs/connect) itself, [Express](https://expressjs.com) and [Restify](http://restify.com/).

[Docker](https://www.docker.com/) is a set of tools that use OS-level virtualization to deliver software in isolated packages called containers. Containers bundle their own software, libraries and configuration files. [Fly](https://fly.io/) is a platform for full stack applications and databases that need to run globally. You can run arbitrary Docker containers and host popular databases like Postgres.

## Outline

* Create a Node project with a GraphQL Express server
  * Create project, install dependencies, and include `.gitignore`
  * Create `graphqlHTTP` server in `index.js`
  * Run server and execute test query
* Create a container image
  * `Dockerfile`
  * `.dockerignore`
  * Build your image with `docker build`
  * List Docker images with `docker images`
* Run the Docker container with `docker run` and execute test query
  * List containers with `docker ps`
  * Print the output of your app with `docker logs`
* Docker Compose file
  * Create and start containers with `docker compose up`
* Publish to GitHub Container Registry
  * Initialize Git, create a blank repository, and push to newly created repo
  * Login to `ghcr.io` with `docker login`
  * Tag image with `docker tag`
  * Push to registry with `docker push`
  * Pull your image with `docker pull`
* Deploy to Fly
  * Install and authenticate `flyctl` CLI
  * Launch app on Fly with `fly launch`
  * Deploy application with `fly deploy`
  * Show the application's current status with `fly status`

## 1. Create a Node project with a GraphQL Express server

This article will demonstrate how to create a Docker container with Express GraphQL. The code for this article is available [on my GitHub](https://github.com/ajcwebdev/ajcwebdev-express-graphql-docker).

### Create project, install dependencies, and include `.gitignore`

```bash
mkdir ajcwebdev-express-graphql-docker
cd ajcwebdev-express-graphql-docker
npm init -y
npm i express express-graphql graphql
touch index.js Dockerfile .dockerignore docker-compose.yml
echo 'node_modules\n.DS_Store\npackage-lock.json' > .gitignore
```

### Create `graphqlHTTP` server in `index.js`

Enter the following code into `index.js` to import the `graphqlHTTP` function from `express-graphql`.

```js
// index.js

const express = require('express')
const { graphqlHTTP } = require('express-graphql')
const { buildSchema } = require('graphql')

const schema = buildSchema(`type Query { hello: String }`)
const rootValue = { hello: () => 'Hello from Express GraphQL!' }

const app = express()

app.use(
  '/graphql',
  graphqlHTTP({
    schema,
    rootValue,
    graphiql: {
      headerEditorEnabled: true
    },
  }),
)

app.listen(8080, '0.0.0.0')

console.log('Running Express GraphQL server at http://localhost:8080/graphql')
```

`graphqlHTTP` accepts a wide range of options, some of the most common include:

- **`schema`** - A `GraphQLSchema` instance from `GraphQL.js`
- **`rootValue`** - A value to pass as the `rootValue` to the `execute()` function
- **`graphiql`** - If passed `true` or an options object it will present GraphiQL when the GraphQL endpoint is loaded in a browser
- **`headerEditorEnabled`** - Optional boolean which enables the header editor when `true`

### Run server and execute test query

```bash
node index.js
```

```
Running Express GraphQL server at http://localhost:8080/graphql
```

`express-graphql` will accept requests with the parameters:

- **`query`** - A string GraphQL document to be executed
- **`variables`** - The runtime values to use for any GraphQL query variables as a JSON object
- **`operationName`** - Specifies which operation should be executed if the provided `query` contains multiple named operations

```graphql
query HELLO_QUERY { hello }
```

![01-express-graphql-hello-localhost-8080](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6tb9ltduj8k6lfiwwwb9.png)

```bash
curl --request POST \
  --url http://localhost:8080/graphql \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```

## 2. Create a container image

We need to build a Docker image of your app to run this app inside a Docker container.

### `Dockerfile`

Docker can build images automatically by reading the instructions from a [`Dockerfile`](https://docs.docker.com/engine/reference/builder/). A `Dockerfile` is a text document that contains all the commands a user could call on the command line to assemble an image. Using `docker build` users can create an automated build that executes several command-line instructions in succession.

```dockerfile
FROM node:14-alpine
LABEL org.opencontainers.image.source https://github.com/ajcwebdev/ajcwebdev-express-graphql-docker
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm i
COPY . ./
EXPOSE 8080
CMD [ "node", "index.js" ]
```

### `.dockerignore`

Before the docker CLI sends the context to the docker daemon, it looks for a file named `.dockerignore` in the root directory of the context. If this file exists, the CLI modifies the context to exclude files and directories that match patterns in it. This helps avoid sending large or sensitive files and directories to the daemon.

```
node_modules
Dockerfile
.dockerignore
.git
.gitignore
npm-debug.log
```

This will prevent your local modules and debug logs from being copied onto your Docker image and possibly overwriting modules installed within your image.

### Build your image with `docker build`

The [`docker build`](https://docs.docker.com/engine/reference/commandline/build/) command builds an image from a Dockerfile and a "context". A build’s context is the set of files located in the specified `PATH` or `URL`. The `URL` parameter can refer to three kinds of resources:
* Git repositories
* Pre-packaged tarball contexts
* Plain text files

```bash
docker build . -t ajcwebdev/ajcwebdev-express-graphql-docker
```

The `-t` flag lets you tag your image so it's easier to find later using the `docker images` command.

### List Docker images with `docker images`

Your image will now be listed by Docker. The [`docker images`](https://docs.docker.com/engine/reference/commandline/images/) command will list all top level images, their repository and tags, and their size.

```bash
docker images
```

```
REPOSITORY
ajcwebdev/ajcwebdev-express-graphql-docker

TAG
latest

IMAGE ID
d833d418e179

CREATED
About a minute ago

SIZE
122MB
```

## 3. Run the Docker container with `docker run` and execute test query

Docker runs processes in isolated containers. A container is a process which runs on a host. The host may be local or remote. When an operator executes [`docker run`](https://docs.docker.com/engine/reference/run/), the container process that runs is isolated in that it has its own file system, its own networking, and its own isolated process tree separate from the host.

```bash
docker run -p 49160:8080 -d ajcwebdev/ajcwebdev-express-graphql-docker
```

`-d` runs the container in detached mode, leaving the container running in the background. The `-p` flag redirects a public port to a private port inside the container.

### List containers with `docker ps`

To test your app, get the port of your app that Docker mapped:

```bash
docker ps
```

```
CONTAINER ID
4bdd108175ab

IMAGE
ajcwebdev/ajcwebdev-express-graphql-docker

COMMAND
"docker-entrypoint.s…"

CREATED
16 seconds ago

STATUS
Up 14 seconds

PORTS
0.0.0.0:49160->8080/tcp, :::49160->8080/tcp

NAMES
silly_greider
```

### Print the output of your app with `docker logs`

```bash
docker logs <container id>
```

```
Running Express GraphQL server at http://localhost:8080/graphql
```

Docker mapped the `8080` port inside of the container to the port `49160` on your machine. Open [localhost:49160/graphql](http://localhost:49160/graphql) and send a hello query.

```graphql
query HELLO_QUERY { hello }
```

![02-localhost-49160-graphql](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bpim4bg6oi7z09r4w3hw.png)

```bash
curl --request POST \
  --url http://localhost:49160/graphql \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```

## 4. Docker Compose file

[Compose](https://docs.docker.com/compose/) is a tool for defining and running multi-container Docker applications. After configuring your application’s services with a YAML file, you can create and start all your services with a single command. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.

```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "49160:8080"
```

### Create and start containers with `docker compose up`

Stop your currently running container before running the next command or the port will be in use.

```bash
docker stop <container id>
```

The `docker compose up` command aggregates the output of each container. It builds, (re)creates, starts, and attaches to containers for a service.

```bash
docker compose up
```

```
Attaching to web_1
web_1  | Running Express GraphQL server at http://localhost:8080/graphql
```

## 5. Publish to GitHub Container Registry

We can publish this image to the GitHub Container Registry with GitHub Packages. This will require pushing our project to a GitHub repository.

### Initialize Git, create a blank repository, and push to newly created repo

```bash
git init
git add .
git commit -m "A container for my graph"
gh repo create ajcwebdev-express-graphql-docker
git push -u origin main
```

[GitHub Packages](https://docs.github.com/en/packages/learn-github-packages/introduction-to-github-packages) is a platform for hosting and managing packages that combines your source code and packages in one place including containers and other dependencies. You can integrate GitHub Packages with GitHub APIs, GitHub Actions, and webhooks to create an end-to-end DevOps workflow that includes your code, CI, and deployment solutions.

GitHub Packages offers different package registries for commonly used package managers, such as npm, RubyGems, Maven, Gradle, and Docker. GitHub's [Container registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry) is optimized for containers and supports Docker and OCI images.

### Login to `ghcr.io` with `docker login`

To login, create a [PAT (personal access token)](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token) with the ability to read, write, and delete packages and include it instead of `xxxx`.

```bash
export CR_PAT=xxxx
```

Login with your own username in place of `ajcwebdev`.

```bash
echo $CR_PAT | docker login ghcr.io -u ajcwebdev --password-stdin
```

### Tag image with `docker tag`

```bash
docker tag ajcwebdev/ajcwebdev-express-graphql-docker ghcr.io/ajcwebdev/ajcwebdev-express-graphql-docker
```

### Push to registry with `docker push`

```bash
docker push ghcr.io/ajcwebdev/ajcwebdev-express-graphql-docker:latest
```

### Pull your image with `docker pull`

To test that our project has a docker image published to a public registry, pull it from your local development environment.

```bash
docker pull ghcr.io/ajcwebdev/ajcwebdev-express-graphql-docker
```

```
Using default tag: latest
latest: Pulling from ajcwebdev/ajcwebdev-express-graphql-docker
Digest: sha256:3ff756a3310fcfee7be355e86a6b8f6c7882f94c3a767b1f614f274ae1c82ba4
Status: Image is up to date for ghcr.io/ajcwebdev/ajcwebdev-express-graphql-docker:latest
ghcr.io/ajcwebdev/ajcwebdev-express-graphql-docker:latest
```

You can view this published container [on my GitHub](https://github.com/users/ajcwebdev/packages/container/package/ajcwebdev-express-graphql-docker).

## 6. Deploy to Fly

### Install and authenticate `flyctl` CLI

You can download the CLI on [Mac, Linux, or Windows](https://fly.io/docs/getting-started/installing-flyctl/).

```bash
brew install superfly/tap/flyctl
```

If you are a new user you can create an account with `fly auth signup`.

```bash
fly auth signup
```

You will also be prompted for credit card payment information, required for charges outside the free plan on Fly. See [Pricing](https://fly.io/docs/about/pricing) for more details. If you already have an account you can login with `fly auth login`.

```bash
fly auth login
```

### Launch app on Fly with `fly launch`

Run `fly launch` in the directory with your source code to configure your app for deployment. This will create and configure a fly app by inspecting your source code and prompting you to deploy.

```bash
fly launch \
  --name ajcwebdev-express-graphql-docker \
  --region sjc
```

```
Creating app in /Users/ajcwebdev/ajcwebdev-express-graphql-docker
Scanning source code
Detected Dockerfile app
Automatically selected personal organization: Anthony Campolo
Created app ajcwebdev-express-graphql-docker in organization personal
Wrote config file fly.toml
Your app is ready. Deploy with `flyctl deploy`
? Would you like to deploy now? No
```

This creates a `fly.toml` file.

```toml
app = "ajcwebdev-express-graphql-docker"

kill_signal = "SIGINT"
kill_timeout = 5
processes = []

[env]

[experimental]
  allowed_public_ports = []
  auto_rollback = true

[[services]]
  http_checks = []
  internal_port = 8080
  processes = ["app"]
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

### Deploy application with `fly deploy`

```bash
fly deploy
```

### Show the application's current status with `fly status`

Status includes application details, tasks, most recent deployment details and in which regions it is currently allocated.

```bash
fly status
```

```
App
  Name     = ajcwebdev-express-graphql-docker          
  Owner    = personal                                  
  Version  = 0                                         
  Status   = running                                   
  Hostname = ajcwebdev-express-graphql-docker.fly.dev  

Deployment Status
  ID          = fd7bf249-c37f-7b16-5643-9bfd104a2077         
  Version     = v0                                           
  Status      = successful                                   
  Description = Deployment completed successfully            
  Instances   = 1 desired, 1 placed, 1 healthy, 0 unhealthy  

Instances
ID       TASK VERSION REGION DESIRED STATUS  HEALTH CHECKS      RESTARTS CREATED   
9eb4eaf9 app  0       sjc    run     running 1 total, 1 passing 0        1m15s ago
```

Visit [ajcwebdev-express-graphql-docker.fly.dev/graphql](https://ajcwebdev-express-graphql-docker.fly.dev/graphql) to see the site and run a test query.

```graphql
query HELLO_QUERY { hello }
```

![03-ajcwebdev-express-graphql-docker-fly-dev-hello](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/e7ab8m96y7j8frtxnwg8.png)

```bash
curl --request POST \
  --url https://ajcwebdev-express-graphql-docker.fly.dev/graphql \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```