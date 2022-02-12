---
title: a first look at graphQL helix
description: GraphQL Helix is a runtime agnostic collection of utility functions for building your own GraphQL HTTP server.
date: 2021-09-20
tags:
  - graphql
  - helix
  - serverless
  - docker
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ljpp4e0ucezkfq7gbs1x.png
layout: layouts/post.njk
---

[GraphQL Helix](https://github.com/contrawork/graphql-helix) is a framework and runtime agnostic collection of utility functions for building your own GraphQL HTTP server. Instead of providing a complete HTTP server or middleware plugin function, GraphQL Helix only provides a handful of functions for turning an HTTP request into a GraphQL execution result. You decide how to send back the response.

## Outline

* [Motivations and API](#motivations-and-api)
* [Serve GraphQL Helix Locally](#serve-graphql-helix-locally)
* [Deploy GraphQL Helix with Serverless Framework](#deploy-graphql-helix-with-serverless-framework)
* [Deploy GraphQL Helix with Amplify](#deploy-graphql-helix-with-amplify)
* [Deploy GraphQL Helix with Docker and Fly](#deploy-graphql-helix-with-docker-and-fly)
* [Resources](#resources)

## Motivations and API

Daniel Rearden listed the following reasons pushing him to create Helix, believing that these factors were absent from popular solutions like [Apollo Server](https://github.com/apollographql/apollo-server), [express-graphql](https://github.com/graphql/express-graphql) and [Mercurius](https://github.com/mercurius-js/mercurius):
* *Wanted bleeding-edge GraphQL features like `@defer`, `@stream` and `@live` directives.*
* *Wanted to not be tied down to a specific framework or runtime environment.*
* *Wanted control over how server features like persisted queries were implemented.*
* *Wanted something other than WebSockets (i.e. SSE) for subscriptions.*

### `renderGraphiQL` and `shouldRenderGraphiQL`

`renderGraphiQL` returns the HTML to render a GraphiQL instance. `shouldRenderGraphiQL` uses the method and headers in the request to determine whether a GraphiQL instance should be returned instead of processing an API request.

### `getGraphQLParameters`

`getGraphQLParameters` extracts the GraphQL parameters from the request including the `query`, `variables` and `operationName` values.

### `processRequest`

`processRequest` takes the `schema`, `request`, `query`, `variables`, `operationName` and a number of other optional parameters and returns one of three kinds of results, depending on how the server should respond:

1. `RESPONSE` - regular JSON payload
2. `MULTIPART RESPONSE` - multipart response (when `@stream` or `@defer` directives are used)
3. `PUSH` - stream of events to push back down the client for a subscription

## Serve GraphQL Helix Locally

```bash
mkdir ajcwebdev-graphql-helix
cd ajcwebdev-graphql-helix
yarn init -y
yarn add express graphql-helix graphql
touch index.js
echo 'node_modules\n.DS_Store' > .gitignore
```

### `index.js`

```js
// index.js

const express = require("express")
const {
  getGraphQLParameters,
  processRequest,
  renderGraphiQL,
  shouldRenderGraphiQL,
} = require("graphql-helix")
const {
  GraphQLObjectType,
  GraphQLSchema,
  GraphQLString,
} = require("graphql")

const schema = new GraphQLSchema({
  query: new GraphQLObjectType({
    name: "Query",
    fields: () => ({
      hello: {
        type: GraphQLString,
        resolve: () => "Hello from GraphQL Helix!",
      }
    }),
  }),
})

const app = express()

app.use(express.json())

app.use("/graphql", async (req, res) => {
  const request = {
    body: req.body,
    headers: req.headers,
    method: req.method,
    query: req.query,
  }

  if (shouldRenderGraphiQL(request)) {
    res.send(renderGraphiQL())
  }

  else {
    const {
      operationName,
      query,
      variables
    } = getGraphQLParameters(request)

    const result = await processRequest({
      operationName,
      query,
      variables,
      request,
      schema,
    })

    if (result.type === "RESPONSE") {
      result.headers.forEach((
        { name, value }
      ) => res.setHeader(name, value))
      res.status(result.status)
      res.json(result.payload)
    }
  }
})

const port = process.env.PORT || 4000

app.listen(port, () => {
  console.log(`GraphQL server is running on port ${port}.`)
})
```

### Run test queries on GraphQL Helix Locally

Start the server with `node index.js`.

```bash
node index.js
```

Open [localhost:4000/graphql](http://localhost:4000/graphql) and send a `hello` query.

```graphql
query HELLO_QUERY { hello }
```

![01-graphql-helix-localhost-4000](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/soal8cgc3q1ymhc9urz6.png)

```bash
curl --request POST \
  --url http://localhost:4000/graphql \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```

### GraphQL Helix Final Project Structure

```
├── .gitignore
├── index.js
└── package.json
```

## Deploy GraphQL Helix with Serverless Framework

The [Serverless Framework](https://www.serverless.com/) is an open source framework for building applications on AWS Lambda. It provides a CLI for developing and deploying [AWS Lambda](https://www.serverless.com/framework/docs/providers/aws/guide/intro/) functions, along with the AWS infrastructure resources they require.

```bash
mkdir graphql-helix-serverless
cd graphql-helix-serverless
yarn init -y
yarn add express graphql-helix graphql serverless-http
touch index.js serverless.yml
echo 'node_modules\n.DS_Store\n.serverless' > .gitignore
```

### `index.js`

The `serverless-http` package is a piece of middleware that handles the interface between Node applications and the specifics of API Gateway. It allows you to wrap your API for serverless use without needing an HTTP server, port, or socket.

```js
// index.js

const serverless = require('serverless-http')
const express = require("express")
const {
  getGraphQLParameters,
  processRequest,
  renderGraphiQL,
  shouldRenderGraphiQL,
} = require("graphql-helix")
const {
  GraphQLObjectType,
  GraphQLSchema,
  GraphQLString,
} = require("graphql")

const schema = new GraphQLSchema({
  query: new GraphQLObjectType({
    name: "Query",
    fields: () => ({
      hello: {
        type: GraphQLString,
        resolve: () => "Hello from GraphQL Helix on Serverless!",
      }
    }),
  }),
})

const app = express()

app.use(express.json())

app.use("/graphql", async (req, res) => {
  const request = {
    body: req.body,
    headers: req.headers,
    method: req.method,
    query: req.query,
  }

  if (shouldRenderGraphiQL(request)) {
    res.send(renderGraphiQL())
  }

  else {
    const {
      operationName,
      query,
      variables
    } = getGraphQLParameters(request)

    const result = await processRequest({
      operationName,
      query,
      variables,
      request,
      schema,
    })

    if (result.type === "RESPONSE") {
      result.headers.forEach((
        { name, value }
      ) => res.setHeader(name, value))
      res.status(result.status)
      res.json(result.payload)
    }
  }
})

const handler = serverless(app)

module.exports.start = async (event, context) => {
  const result = await handler(event, context)

  return result
}
```

### `serverless.yml`

The resources and functions are defined in a file called `serverless.yml` which includes:
* The `provider` for the Node `runtime` and AWS `region`
* The `handler` and `events` for your `functions`.

```yaml
# serverless.yml

service: ajcwebdev-graphql-helix-express
frameworkVersion: '2'

provider:
  name: aws
  stage: dev
  runtime: nodejs14.x
  versionFunctions: false
  lambdaHashingVersion: 20201221

  httpApi:
    cors:
      allowedOrigins:
        - '*'
      allowedMethods:
        - GET
        - POST
        - HEAD
      allowedHeaders:
        - Accept
        - Authorization
        - Content-Type

functions:
  endpoint:
    handler: index.start
    events:
      - httpApi:
          path: '*'
          method: '*'
```

The handler is named `index.start` because it is formatted as `<FILENAME>.<HANDLER>`.

### Upload to AWS with `sls deploy`

Once the project is defined in code it can be deployed with the `sls deploy` command. This command creates a CloudFormation stack defining any necessary resources such as API gateways or S3 buckets.

```bash
sls deploy --verbose
```

```yaml
service: ajcwebdev-graphql-helix-express
stage: dev
region: us-east-1
stack: ajcwebdev-graphql-helix-express-dev
resources: 10
api keys:
  None
endpoints:
  ANY - https://cuml5hnx0b.execute-api.us-east-1.amazonaws.com
functions:
  endpoint: ajcwebdev-graphql-helix-express-dev-endpoint
layers:
  None
```

### Run test queries on GraphQL Helix Serverless

Open [cuml5hnx0b.execute-api.us-east-1.amazonaws.com/graphql](https://cuml5hnx0b.execute-api.us-east-1.amazonaws.com/graphql) and send a `hello` query.

```graphql
query HELLO_QUERY { hello }
```

![02-graphql-helix-serverless-framework](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kixhj8e2rf5oa51b42hy.png)

```bash
curl --request POST \
  --url https://cuml5hnx0b.execute-api.us-east-1.amazonaws.com/graphql \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```

### GraphQL Helix Serverless Final Project Structure

```
├── .gitignore
├── index.js
├── package.json
├── serverless.yml
└── yarn.lock
```

## Deploy GraphQL Helix with Amplify

[AWS Amplify](https://aws.amazon.com/amplify/) is a set of tools and services to help frontend web and mobile developers build fullstack applications with AWS infrastructure. It includes a [CLI](https://docs.amplify.aws/cli) for creating and deploying [CloudFormation stacks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacks.html) along with a [Console](https://console.aws.amazon.com/amplify/home) and [Admin UI](https://sandbox.amplifyapp.com/getting-started) for managing frontend web apps, backend environments, CI/CD, and user data.

```bash
mkdir graphql-helix-amplify
cd graphql-helix-amplify
amplify init
```

The `amplify init` command creates a boilerplate project that is setup for generating CloudFormation templates.

```
? Enter a name for the project ajcwebdevhelix
The following configuration will be applied:

Project information
| Name: ajcwebdevhelix
| Environment: dev
| Default editor: Visual Studio Code
| App type: javascript
| Javascript framework: none
| Source Directory Path: src
| Distribution Directory Path: dist
| Build Command: npm run-script build
| Start Command: npm run-script start

? Initialize the project with the above configuration? Yes
Using default provider  awscloudformation
? Select the authentication method you want to use: AWS profile

For more information on AWS Profiles, see:
https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html

? Please choose the profile you want to use default
```

### Create backend with `amplify add api`

`amplify add api` configures a Lambda handler and API gateway to serve the function.

```bash
amplify add api
```

```
? Please select from one of the below mentioned services: REST
? Provide a friendly name for your resource to be used as a label for this category in the project: helixresource
? Provide a path (e.g., /items): /graphql
? Choose a Lambda source: Create a new Lambda function
? Provide the AWS Lambda function name: helixfunction
? Choose the function runtime that you want to use: NodeJS
? Choose the function template that you want to use: Hello World
? Do you want to access other resources created in this project from your Lambda function? N
? Do you want to edit the local lambda function now? N
? Restrict API access: N
? Do you want to add another path? N
```

```bash
cd amplify/backend/function/helixfunction/src
yarn add graphql-helix graphql express serverless-http
cd ../../../../../
```

### `index.js`

```javascript
// amplify/backend/function/helixfunction/src/index.js

const serverless = require('serverless-http')
const express = require("express")
const {
  getGraphQLParameters,
  processRequest,
  renderGraphiQL,
  shouldRenderGraphiQL,
} = require("graphql-helix")
const {
  GraphQLObjectType,
  GraphQLSchema,
  GraphQLString,
} = require("graphql")

const schema = new GraphQLSchema({
  query: new GraphQLObjectType({
    name: "Query",
    fields: () => ({
      hello: {
        type: GraphQLString,
        resolve: () => "Hello from GraphQL Helix on Amplify!",
      }
    }),
  }),
})

const app = express()

app.use(express.json())

app.use("/graphql", async (req, res) => {
  const request = {
    body: req.body,
    headers: req.headers,
    method: req.method,
    query: req.query,
  }

  if (shouldRenderGraphiQL(request)) {
    res.send(renderGraphiQL())
  }

  else {
    const {
      operationName,
      query,
      variables
    } = getGraphQLParameters(request)

    const result = await processRequest({
      operationName,
      query,
      variables,
      request,
      schema,
    })

    if (result.type === "RESPONSE") {
      result.headers.forEach((
        { name, value }
      ) => res.setHeader(name, value))
      res.status(result.status)
      res.json(result.payload)
    }
  }
})

module.exports.handler = serverless(app)
```

### Upload to AWS with `amplify push`

`amplify push` uploads the stack templates to an S3 bucket and calls the CloudFormation API to create or update resources in the cloud.

```bash
amplify push
```

```
✔ Successfully pulled backend environment dev from the cloud.

    Current Environment: dev
    
┌──────────┬───────────────┬───────────┬───────────────────┐
│ Category │ Resource name │ Operation │ Provider plugin   │
├──────────┼───────────────┼───────────┼───────────────────┤
│ Function │ helixfunction │ Create    │ awscloudformation │
├──────────┼───────────────┼───────────┼───────────────────┤
│ Api      │ helixresource │ Create    │ awscloudformation │
└──────────┴───────────────┴───────────┴───────────────────┘

? Are you sure you want to continue? Yes

REST API endpoint: https://acj63jadzb.execute-api.us-west-1.amazonaws.com/dev
```

### Run test queries on GraphQL Helix Amplify

Open [acj63jadzb.execute-api.us-west-1.amazonaws.com/dev/graphql](https://acj63jadzb.execute-api.us-west-1.amazonaws.com/dev/graphql) and send a `hello` query.

```graphql
query HELLO_QUERY { hello }
```

![03-graphql-helix-amplify](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ffbps9dht81uqnxi0h76.png)

```bash
curl --request POST \
  --url https://acj63jadzb.execute-api.us-west-1.amazonaws.com/dev/graphql \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```

### GraphQL Helix Amplify Final Project Structure

```
├── .gitignore
└── amplify
    └── backend
        ├── api
        │   └── helixresource
        │       ├── api-params.json
        │       ├── helixresource-cloudformation-template.json
        │       └── parameters.json
        └── function
            └── helixfunction
                ├── function-parameters.json
                ├── helixfunction-cloudformation-template.json
                └── src
                    ├── event.json
                    ├── index.js
                    ├── package.json
                    └── yarn.lock
```

## Deploy GraphQL Helix with Docker and Fly

[Fly](https://fly.io/) is a platform for full stack applications and databases that need to run globally. Fly executes your code close to users and scales compute in cities where your app is busiest. You can run arbitrary Docker containers and host popular databases like Postgres.

```bash
mkdir graphql-helix-docker
cd graphql-helix-docker
npm init -y
npm i express graphql-helix graphql
touch index.js Dockerfile .dockerignore docker-compose.yml
echo 'node_modules\n.DS_Store' > .gitignore
```

### `index.js`

```js
// index.js

const express = require("express")
const {
  getGraphQLParameters,
  processRequest,
  renderGraphiQL,
  shouldRenderGraphiQL,
} = require("graphql-helix")
const {
  GraphQLObjectType,
  GraphQLSchema,
  GraphQLString,
} = require("graphql")

const schema = new GraphQLSchema({
  query: new GraphQLObjectType({
    name: "Query",
    fields: () => ({
      hello: {
        type: GraphQLString,
        resolve: () => "Hello from GraphQL Helix on Docker!",
      }
    }),
  }),
})

const app = express()

app.use(express.json())

app.use("/graphql", async (req, res) => {
  const request = {
    body: req.body,
    headers: req.headers,
    method: req.method,
    query: req.query,
  }

  if (shouldRenderGraphiQL(request)) {
    res.send(renderGraphiQL())
  }

  else {
    const {
      operationName,
      query,
      variables
    } = getGraphQLParameters(request)

    const result = await processRequest({
      operationName,
      query,
      variables,
      request,
      schema,
    })

    if (result.type === "RESPONSE") {
      result.headers.forEach((
        { name, value }
      ) => res.setHeader(name, value))
      res.status(result.status)
      res.json(result.payload)
    }
  }
})

const port = process.env.PORT || 8080

app.listen(port, () => {
  console.log(`GraphQL server is running on port ${port}.`)
})
```

### `Dockerfile`

Docker can build images automatically by reading the instructions from a [`Dockerfile`](https://docs.docker.com/engine/reference/builder/). A `Dockerfile` is a text document that contains all the commands a user could call on the command line to assemble an image.

```dockerfile
FROM node:14-alpine
LABEL org.opencontainers.image.source https://github.com/ajcwebdev/graphql-helix-docker
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm i
COPY . ./
EXPOSE 8080
CMD [ "node", "index.js" ]
```

For a more in depth explanation of these commands, see my previous article, [A First Look at Docker](https://dev.to/ajcwebdev/a-first-look-at-docker-3hfg).

### `.dockerignore`

Before the docker CLI sends the context to the docker daemon, it looks for a file named `.dockerignore` in the root directory of the context.

```
node_modules
Dockerfile
.dockerignore
.git
.gitignore
npm-debug.log
```

If this file exists, the CLI modifies the context to exclude files and directories that match patterns in it. This helps avoid sending large or sensitive files and directories to the daemon.

### `docker-compose.yml`

[Compose](https://docs.docker.com/compose/) is a tool for defining and running multi-container Docker applications. After configuring your application’s services with a YAML file, you can create and start all your services with a single command.

```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "49160:8080"
```

### Run test queries on GraphQL Helix Docker

The `docker compose up` command aggregates the output of each container. It builds, (re)creates, starts, and attaches to containers for a service.

```bash
docker compose up
```

```
Attaching to web_1
web_1  | GraphQL server is running on port 8080.
```

To test your app, get the port of your app that Docker mapped:

```bash
docker ps
```

Docker mapped the `8080` port inside of the container to the port `49160` on your machine.

```
CONTAINER ID
50935f5f4ae6

IMAGE
graphql-helix-docker_web

COMMAND
"docker-entrypoint.s…"

CREATED
47 seconds ago

STATUS
Up 46 seconds

PORTS
0.0.0.0:49160->8080/tcp, :::49160->8080/tcp

NAMES
graphql-helix-docker_web_1
```

Open [localhost:49160/graphql](http://localhost:49160/graphql) and send a `hello` query.

```graphql
query HELLO_QUERY { hello }
```

![04-graphql-helix-docker](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vzdhb58peq9id0pr07ca.png)

```bash
curl --request POST \
  --url http://localhost:49160/graphql \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```

### Launch app on Fly with `fly launch`

Run `fly launch` in the directory with your source code to configure your app for deployment.

```bash
fly launch \
  --name graphql-helix-docker \
  --region sjc
```

This will create and configure a fly app by inspecting your source code and prompting you to deploy.

```
Creating app in /Users/ajcwebdev/graphql-helix-docker
Scanning source code
Detected Dockerfile app
Automatically selected personal organization: Anthony Campolo
Created app graphql-helix-docker in organization personal
Wrote config file fly.toml
Your app is ready. Deploy with `flyctl deploy`
? Would you like to deploy now? No
```

Open `fly.toml` and add the following `PORT` number under `env`.

```toml
[env]
  PORT = 8080
```

### Deploy application with `fly deploy`

```bash
fly deploy
```

```
Image: registry.fly.io/graphql-helix-docker:deployment-1631689218
Image size: 124 MB

==> Creating release
Release v2 created

You can detach the terminal anytime without stopping the deployment
Monitoring Deployment

1 desired, 1 placed, 1 healthy, 0 unhealthy [health checks: 1 total, 1 passing]
--> v0 deployed successfully
```

Check the application's status with `fly status`.

```bash
fly status
```

```
App
  Name     = graphql-helix-docker          
  Owner    = personal                      
  Version  = 0                             
  Status   = running                       
  Hostname = graphql-helix-docker.fly.dev  

Deployment Status
  ID          = 47cb82b9-aaf1-5ee8-df1b-b4f10e389f16         
  Version     = v0                                           
  Status      = successful                                   
  Description = Deployment completed successfully            
  Instances   = 1 desired, 1 placed, 1 healthy, 0 unhealthy  

Instances
ID       TASK VERSION REGION DESIRED STATUS  HEALTH CHECKS      RESTARTS CREATED   
a8d02b87 app  0       sjc    run     running 1 total, 1 passing 0        4m28s ago
```

### Run test queries on GraphQL Helix Docker Fly

Open [graphql-helix-docker.fly.dev/graphql](https://graphql-helix-docker.fly.dev/graphql) and send a `hello` query.

```graphql
query HELLO_QUERY { hello }
```

![05-graphql-helix-docker-fly](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/opusale23qxq9oglj6sz.png)

```bash
curl --request POST \
  --url https://graphql-helix-docker.fly.dev/graphql \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```

### GraphQL Helix Docker Final Project Structure

```
├── .dockerignore
├── .gitignore
├── docker-compose.yml
├── Dockerfile
├── fly.toml
├── index.js
└── package.json
```

## Resources

[Building a GraphQL server with GraphQL Helix](https://dev.to/danielrearden/building-a-graphql-server-with-graphql-helix-2k44) provides a comprehensive description of GraphQL Helix's implementation. The `examples` folder in the `graphql-helix` repo also includes [example applications](https://github.com/contrawork/graphql-helix/tree/master/examples/) such as:

* [HTTP Server](https://github.com/contrawork/graphql-helix/tree/master/examples/http)
* [Express](https://github.com/contrawork/graphql-helix/tree/master/examples/express)
* [Fastify](https://github.com/contrawork/graphql-helix/tree/master/examples/fastify)
* [Koa](https://github.com/contrawork/graphql-helix/tree/master/examples/koa)
* [Live Queries](https://github.com/contrawork/graphql-helix/tree/master/examples/live-queries)
* [Persisted Queries](https://github.com/contrawork/graphql-helix/tree/master/examples/persisted-queries)
* [WebSockets](https://github.com/contrawork/graphql-helix/tree/master/examples/graphql-ws)
* [Content Security Policy](https://github.com/contrawork/graphql-helix/tree/master/examples/csp)
* [Next.js](https://github.com/contrawork/graphql-helix/tree/master/examples/nextjs)