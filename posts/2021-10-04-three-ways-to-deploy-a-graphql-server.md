---
title: three ways to deploy a serverless graphQL API
description: How to deploy Apollo Server and GraphQL Yoga on serverless functions with Netlify Functions, Serverless Framework, and AWS Amplify.
date: 2021-10-04
tags:
  - graphql
  - serverless
  - lambda
  - express
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vznt9sqbsy28hh48nzr8.png
layout: layouts/post.njk
---

There are a wide range of different options for GraphQL servers with varying deployment methods. Typical methods for hosting your server include virtual machines (AWS EC2, Digital Ocean Droplet) or containers (Cloud Run, AWS ECS). However, if your server does not need to persist state, then you can host it using a serverless function. This includes:
* Bundling your application into a zip file
* Storing that static file somewhere with blob storage like an S3 bucket
* Serving that file using an API gateway and serverless function like an AWS Lambda

## Outline

* [Pros and Cons](#pros-and-cons)
* [Deployment Providers](#deployment-providers)
  * [Netlify Functions](#netlify-functions)
  * [Serverless Framework](#serverless-framework)
  * [Amplify](#amplify)
* [Serve Apollo Server Locally](#serve-apollo-server-locally)
* [Deploy Apollo Server Lambda with Netlify](#deploy-apollo-server-lambda-with-netlify)
* [Deploy Apollo Server Lambda with Serverless Framework](#deploy-apollo-server-lambda-with-serverless-framework)
* [Deploy Apollo Server Lambda with Amplify](#deploy-apollo-server-lambda-with-amplify)
* [Serve GraphQL Yoga Locally](#serve-graphql-yoga-locally)
* [Deploy GraphQL Yoga with Netlify](#deploy-graphql-yoga-with-netlify)
* [Deploy GraphQL Yoga with Serverless Framework](#deploy-graphql-yoga-with-serverless-framework)
* [Deploy GraphQL Yoga with Amplify](#deploy-graphql-yoga-with-amplify)

## Pros and Cons

The benefits include removing large amounts of operations work such as managing the operating system of underlying VMs or creating optimized Dockerfiles. You also no longer need to worry about scaling your application or paying for idle time when your server is not being used.

Since the server state cannot be persisted, the drawbacks include removing the ability to use websockets or subscriptions. The functions will only work for basic `GET` and `POST` methods. You also need to write your code with the syntax of the underlying runtime which requires keeping in mind information such as the available Node version and compatible dependencies.

## Deployment Providers

This article looks at three different methods of deploying two different GraphQL servers: Apollo Server and GraphQL Yoga. They will be deployed with Netlify Functions, Serverless Framework, and AWS Amplify. All three use the same underlying technology, AWS Lambda functions. They differ in terms of the conventions and abstractions around creating, developing, and deploying those functions.

### Netlify Functions

[Netlify Functions](https://docs.netlify.com/functions/overview/) are scripts that you can write and deploy directly on Netlify. Netlify lets you deploy serverless Lambda functions without an AWS account, and with function management handled directly within Netlify. It requires the least amount of knowledge of AWS, but also gives you the least amount of control over the underlying infrastructure.

Lambda functions can be added to a project by creating a JavaScript file in a [configured functions directory](https://docs.netlify.com/functions/configure-and-deploy/#configure-the-functions-folder). The function endpoint is determined by its filename or the name of its dedicated parent directory. It exports a `handler` that receives an event object similar to what you would receive from AWS API Gateway.

### Serverless Framework

The [Serverless Framework](https://www.serverless.com/) is an open source framework for building applications on AWS Lambda. It provides a CLI for developing and deploying [AWS Lambda](https://www.serverless.com/framework/docs/providers/aws/guide/intro/) functions, along with the AWS infrastructure resources they require.

The resources and functions are defined in a file called `serverless.yml` which includes:
* The `provider` for the Node `runtime` and AWS `region`
* The `handler` and `events` for your `functions`

Once the project is defined in code it can be deployed with the `sls deploy` command. This command creates a CloudFormation stack defining any necessary resources such as API gateways or S3 buckets.

### Amplify

[AWS Amplify](https://aws.amazon.com/amplify/) is a set of tools and services to help frontend web and mobile developers build fullstack applications with AWS infrastructure. It includes a [CLI](https://docs.amplify.aws/cli) for creating and deploying [CloudFormation stacks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacks.html) along with a [Console](https://console.aws.amazon.com/amplify/home) and [Admin UI](https://sandbox.amplifyapp.com/getting-started) for managing frontend web apps, backend environments, CI/CD, and user data.

Amplify provides open source libraries in various languages including:
* [JavaScript](https://github.com/aws-amplify/amplify-js)
* [iOS](https://github.com/aws-amplify/aws-sdk-ios)
* [Android](https://github.com/aws-amplify/aws-sdk-android)
* [Flutter](https://github.com/aws-amplify/amplify-flutter)

The `amplify init` command creates a boilerplate project that is setup for generating CloudFormation templates. `amplify add api` configures a Lambda handler and API gateway to serve the function. `amplify push` uploads the stack templates to an S3 bucket and calls the CloudFormation API to create or update resources in the cloud.

## Serve Apollo Server Locally

[Apollo Server](https://github.com/apollographql/apollo-server) is an open-source, spec-compliant GraphQL server. Apollo Server Core implements the core logic of Apollo Server.

```bash
mkdir apollo-server
cd apollo-server
yarn init -y
yarn add apollo-server graphql
touch index.js
echo 'node_modules\n.DS_Store' > .gitignore
```

### ApolloServer

`apollo-server` exports a base version of the `ApolloServer` constructor which requires two parameters, `typeDefs` for a schema definition and `resolvers` for a set of resolvers. The `listen` method is used to launch the server.

```js
// index.js

const { ApolloServer, gql } = require('apollo-server')

const typeDefs = gql`type Query { hello: String }`
const resolvers = {
  Query: { hello: () => "Hello from Apollo on Localhost!" }
}

const server = new ApolloServer({ typeDefs, resolvers })

server.listen().then(
  ({ url }) => { console.log(`Server ready at ${url}`) }
)
```

`ApolloServer` is usually imported either from a batteries-included `apollo-server` package or from integration packages like `apollo-server-lambda`.

### Run test queries on Apollo Server Locally

Start the server with `node index.js`.

```bash
node index.js
```

Open [localhost:4000](http://localhost:4000/) and run the `hello` query.

```graphql
query HELLO_QUERY { hello }
```

![01-apollo-server-localhost-4000](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g5a31nqppryo82yuo9f6.png)

```bash
curl --request POST \
  --url http://localhost:4000/ \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```

### Apollo Server Final Project Structure

```
├── .gitignore
├── index.js
└── package.json
```

## Deploy Apollo Server Lambda with Netlify

In Apollo Server 3, `apollo-server-lambda` is implemented as a wrapper around `apollo-server-express` and uses [`serverless-express`](https://www.npmjs.com/package/@vendia/serverless-express) to parse AWS Lambda events into Express requests. This will cause problems with Netlify Functions, so instead we will install v2.

```bash
mkdir -p apollo-server-netlify/netlify/functions
cd apollo-server-netlify
yarn init -y
yarn add apollo-server-lambda@2 graphql
touch netlify/functions/index.js
echo 'node_modules\n.DS_Store\n.netlify' > .gitignore
```

### ApolloServer

Each JavaScript file to be deployed as a synchronous serverless Lambda function must export a `handler` method. Netlify provides the `event` and `context` parameters when the serverless function is invoked.

```js
// netlify/functions/index.js

const { ApolloServer, gql } = require('apollo-server-lambda')

const typeDefs = gql`type Query { hello: String }`
const resolvers = {
  Query: { hello: () => "Hello from Apollo on Netlify!" }
}

const server = new ApolloServer({ typeDefs, resolvers })

exports.handler = server.createHandler()
```

### Run test queries on Apollo Server Lambda Netlify Locally

Start the development server with `netlify dev`.

```bash
netlify dev
```

Open [localhost:8888/.netlify/functions/index](http://localhost:8888/.netlify/functions/index/) and run the `hello` query.

```graphql
query HELLO_QUERY { hello }
```

![02-apollo-server-lambda-netlify-localhost-8888](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rf93qml70lz9u9aqox8v.png)

```bash
curl --request POST \
  --url http://localhost:8888/.netlify/functions/index \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```

### Create GitHub Repo and Connect to Netlify

```bash
git init
git add .
git commit -m "apollo server lambda netlify"
gh repo create ajcwebdev-apollo-server-netlify
git push -u origin main
netlify deploy --prod
```

### Run test queries on Apollo Server Lambda Netlify

Open [ajcwebdev-apollo-server-netlify.netlify.app/.netlify/functions/index](https://ajcwebdev-apollo-server-netlify.netlify.app/.netlify/functions/index) and run the `hello` query.

```graphql
query HELLO_QUERY { hello }
```

![03-apollo-server-lambda-netlify-function](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/afv0xnk96bodgywk1fcf.png)

```bash
curl --request POST \
  --url https://ajcwebdev-apollo-server-netlify.netlify.app/.netlify/functions/index \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```

### Apollo Server Lambda Netlify Final Project Structure

```
├── .gitignore
├── netlify
│   └── functions
│       └── index.js
└── package.json
```

## Deploy Apollo Server Lambda with Serverless Framework

```bash
mkdir apollo-server-serverless
cd apollo-server-serverless
yarn init -y
yarn add apollo-server-lambda graphql
touch index.js serverless.yml
echo 'node_modules\n.DS_Store\n.serverless' > .gitignore
```

### ApolloServer

```js
// index.js

const { ApolloServer, gql } = require('apollo-server-lambda')

const typeDefs = gql`type Query { hello: String }`
const resolvers = {
  Query: { hello: () => 'Hello from Apollo on Serverless Framework!' }
}

const server = new ApolloServer({ typeDefs, resolvers })

exports.handler = server.createHandler()
```

### Serverless Framework Configuration

The handler is formatted as `<FILENAME>.<HANDLER>`. I have included `us-west-1` for the region, feel free to enter the AWS region closest to your current location.

```yaml
# serverless.yml

service: ajcwebdev-apollo-server
provider:
  name: aws
  runtime: nodejs14.x
  region: us-west-1
functions:
  graphql:
    handler: index.handler
    events:
    - http:
        path: /
        method: post
        cors: true
    - http:
        path: /
        method: get
        cors: true
```

### Upload to AWS with sls deploy

```bash
sls deploy
```

```yaml
service: ajcwebdev-apollo-server
stage: dev
region: us-west-1
stack: ajcwebdev-apollo-server-dev
resources: 12
api keys:
  None
endpoints:
  POST - https://q6b9hu1h71.execute-api.us-west-1.amazonaws.com/dev/
  GET - https://q6b9hu1h71.execute-api.us-west-1.amazonaws.com/dev/
functions:
  graphql: ajcwebdev-apollo-server-dev-graphql
layers:
  None
```

You can see all of your AWS resources in the various AWS consoles.

CloudFormation

![04-apollo-server-serverless-cloudformation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/524j8hu5xotdnqxsu3pg.png)

Lambda Function

![05-apollo-server-serverless-lambda-function](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/egy9rrqxubnhmj4nx1qy.png)

Lambda Application

![06-apollo-server-serverless-lambda-application](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jf6mgdmmljy51snrsycu.png)

API Gateway

![07-apollo-server-serverless-api-gateway](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m1ujovq6la2xr820tw95.png)

### Run test queries on Apollo Server Lambda Serverless

Open [q6b9hu1h71.execute-api.us-west-1.amazonaws.com/dev/](https://q6b9hu1h71.execute-api.us-west-1.amazonaws.com/dev/) and run the `hello` query.

```graphql
query HELLO_QUERY { hello }
```

![08-apollo-server-lambda-serverless-framework](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gqonhmytwfiblu4ykwcm.png)

```bash
curl --request POST \
  --url https://q6b9hu1h71.execute-api.us-west-1.amazonaws.com/dev/ \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```

### Apollo Server Lambda Serverless Final Project Structure

```
├── .gitignore
├── index.js
├── package.json
└── serverless.yml
```

## Deploy Apollo Server Lambda with Amplify

```bash
mkdir apollo-server-amplify
cd apollo-server-amplify
amplify init
```

```
? Enter a name for the project ajcwebdevapollo
The following configuration will be applied:

Project information
| Name: ajcwebdevapollo
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

CloudFormation

![09-apollo-server-amplify-cloudformation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/f7cbamaoz75ubycem7w0.png)

### Create backend with amplify add api

```bash
amplify add api
```

```
? Please select from one of the below mentioned services: REST
? Provide a friendly name for your resource to be used as a label for this category in the project: ajcwebdevapollo
? Provide a path (e.g., /book/{isbn}): /graphql
? Choose a Lambda source Create a new Lambda function
? Provide an AWS Lambda function name: apolloserver
? Choose the runtime that you want to use: NodeJS
? Choose the function template that you want to use: Hello World
? Do you want to access other resources created in this project from your Lambda function? N
? Do you want to edit the local lambda function now? N
? Restrict API access: N
? Do you want to add another path? N
```

```bash
cd amplify/backend/function/apolloserver/src
yarn add apollo-server-lambda graphql
cd ../../../../../
```

### ApolloServer

```javascript
// amplify/backend/function/apollofunction/src/index.js

const { ApolloServer, gql } = require('apollo-server-lambda')

const typeDefs = gql`type Query { hello: String }`
const resolvers = {
  Query: { hello: () => 'Hello from Apollo on Amplify!' }
}

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ event, context }) => ({
    headers: event.headers,
    functionName: context.functionName,
    event,
    context,
  }),
})
  
exports.handler = server.createHandler()
```

### Upload to AWS with amplify push

Deploy the function containing the GraphQL API.

```bash
amplify push
```

```
✔ Successfully pulled backend environment dev from the cloud.

    Current Environment: dev
    
┌──────────┬─────────────────┬───────────┬───────────────────┐
│ Category │ Resource name   │ Operation │ Provider plugin   │
├──────────┼─────────────────┼───────────┼───────────────────┤
│ Function │ apolloserver    │ Create    │ awscloudformation │
├──────────┼─────────────────┼───────────┼───────────────────┤
│ Api      │ ajcwebdevapollo │ Create    │ awscloudformation │
└──────────┴─────────────────┴───────────┴───────────────────┘
? Are you sure you want to continue? Yes

All resources are updated in the cloud

REST API endpoint: https://kl21tioy61.execute-api.us-east-1.amazonaws.com/dev
```

Lambda Function

![10-apollo-server-amplify-lambda-function](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ncvzptx1kapewdtfrd35.png)

Lambda Application

![11-apollo-server-amplify-lambda-application](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/r89y1gaacss2iqj17l5z.png)

API Gateway

![12-apollo-server-amplify-api-gateway](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/peorlldj908d8rr3y79g.png)

### Run test queries on Apollo Server Lambda Amplify

Open [kl21tioy61.execute-api.us-east-1.amazonaws.com/dev/graphql](https://kl21tioy61.execute-api.us-east-1.amazonaws.com/dev/graphql) and run the `hello` query.

```graphql
query HELLO_QUERY { hello }
```

![13-apollo-server-lambda-amplify](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nntdr11bx4lv9mryswdz.png)

```bash
curl --request POST \
  --url https://kl21tioy61.execute-api.us-east-1.amazonaws.com/dev/graphql \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```

### Apollo Server Lambda Amplify Final Project Structure

```
├── .gitignore
└── amplify
    └── backend
        ├── api
        │   └── ajcwebdevapollo
        │       ├── ajcwebdevapollo-cloudformation-template.json
        │       ├── api-params.json
        │       └── parameters.json
        └── function
            └── apolloserver
                ├── apolloserver-cloudformation-template.json
                ├── function-parameters.json
                └── src
                    ├── event.json
                    ├── index.js
                    └── package.json
```

## Serve GraphQL Yoga Locally

[GraphQL Yoga](https://github.com/dotansimha/graphql-yoga) is a fully-featured GraphQL Server with a focus on easy setup and performance. Originally created by Prisma, it is now mained by Dotan and the Guild. It includes features aimed at providing a great developer experience including file uploading, GraphQL subscriptions support with WebSockets, and TypeScript typing.

`graphql-yoga` is built on other packages that provide functionality required for building a GraphQL server such as web server frameworks like [`express`](https://github.com/expressjs/express) and [`apollo-server`](https://github.com/apollographql/apollo-server), GraphQL subscriptions with [`graphql-subscriptions`](https://github.com/apollographql/graphql-subscriptions) and [`subscriptions-transport-ws`](https://github.com/apollographql/subscriptions-transport-ws), GraphQL engine & schema helpers including [`graphql.js`](https://github.com/graphql/graphql-js) and [`graphql-tools`](https://github.com/apollographql/graphql-tools), and an interactive GraphQL IDE with [`graphql-playground`](https://github.com/graphcool/graphql-playground).

```bash
mkdir graphql-yoga
cd graphql-yoga
yarn init -y
yarn add graphql-yoga
touch index.js
echo 'node_modules\n.DS_Store' > .gitignore
```

### GraphQLServer

`GraphQLServer` is a `constructor` with a `props` argument that accepts a wide array of fields. We will only be using a handful including:

* `typeDefs` - Contains GraphQL type definitions in SDL or file path to type definitions
* `resolvers`- Contains resolvers for the fields specified in `typeDefs`
* `schema` - An instance of [`GraphQLSchema`](http://graphql.org/graphql-js/type/#graphqlschema)

```js
// index.js

const { GraphQLServer } = require('graphql-yoga')

const typeDefs = `type Query { hello: String }`
const resolvers = {
  Query: { hello: () => 'Hello from GraphQL Yoga on Localhost!' }
}

const server = new GraphQLServer({ typeDefs, resolvers })

server.start(
  () => console.log('Server is running on localhost:4000')
)
```

If you provide `typeDefs` and `resolvers` but omit the [`schema`](https://blog.graph.cool/graphql-server-basics-the-schema-ac5e2950214e), `graphql-yoga` will construct the `GraphQLSchema` instance using `makeExecutableSchema` from [`graphql-tools`](https://github.com/apollographql/graphql-tools).

### Run test queries on GraphQL Yoga Locally

```bash
node index.js
```

Open [http://localhost:4000](http://localhost:4000) and run the `hello` query.

```graphql
query HELLO_QUERY { hello }
```

![14-graphql-yoga-localhost-4000](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/i0hsyi1kz80saoiftem4.png)

```bash
curl --request POST \
  --url http://localhost:4000/ \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```

### GraphQL Yoga Final Project Structure

```
├── .gitignore
├── index.js
└── package.json
```

## Deploy GraphQL Yoga with Netlify

```bash
mkdir -p graphql-yoga-netlify/netlify/functions
cd graphql-yoga-netlify
yarn init -y
yarn add graphql-yoga
touch netlify/functions/index.js
echo 'node_modules\n.DS_Store\n.netlify' > .gitignore
```

### GraphQLServerLambda

```js
// netlify/functions/index.js

const { GraphQLServerLambda } = require('graphql-yoga')

const typeDefs = `type Query { hello: String }`
const resolvers = {
  Query: { hello: () => 'Hello from GraphQL Yoga on Netlify!' }
}

const lambda = new GraphQLServerLambda({ typeDefs, resolvers })

exports.handler = lambda.handler
```

### Run test queries on GraphQL Yoga Netlify Locally

Start the development server with `netlify dev`.

```bash
netlify dev
```

Open [localhost:8888/.netlify/functions/index](http://localhost:8888/.netlify/functions/index/) and run the `hello` query.

![15-graphql-yoga-netlify-localhost-8888](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zs6jl2mcxhoi4fvy6hx0.png)

```bash
curl --request POST \
  --url http://localhost:8888/.netlify/functions/index \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```

### Create GitHub Repo and Connect GraphQL Yoga to Netlify

```bash
git init
git add .
git commit -m "graphql yoga netlify"
gh repo create ajcwebdev-graphql-yoga-netlify
git push -u origin main
netlify deploy --prod
```

### Run test queries on GraphQL Yoga Netlify

Open [ajcwebdev-graphql-yoga-netlify.netlify.app/.netlify/functions/index](https://ajcwebdev-graphql-yoga-netlify.netlify.app/.netlify/functions/index) and run the `hello` query.

```graphql
query HELLO_QUERY { hello }
```

![16-graphql-yoga-netlify-function](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zd0y3s9cv2zc5kq3mvi4.png)

```bash
curl --request POST \
  --url https://ajcwebdev-graphql-yoga-netlify.netlify.app/.netlify/functions/index \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```

### GraphQL Yoga Netlify Final Project Structure

```
├── .gitignore
├── netlify
│   └── functions
│       └── index.js
└── package.json
```

## Deploy GraphQL Yoga with Serverless Framework

```bash
mkdir graphql-yoga-serverless
cd graphql-yoga-serverless
yarn init -y
yarn add graphql-yoga
touch index.js serverless.yml
echo 'node_modules\n.DS_Store\n.serverless' > .gitignore
```

### GraphQLServerLambda

```js
// index.js

const { GraphQLServerLambda } = require('graphql-yoga')

const typeDefs = `type Query { hello: String }`
const resolvers = {
  Query: { hello: () => 'Hello from GraphQL Yoga on Serverless Framework!' }
}

const lambda = new GraphQLServerLambda({ typeDefs, resolvers })

exports.handler = lambda.graphqlHandler
exports.playground = lambda.playgroundHandler
```

### Serverless Framework Configuration

```yaml
# serverless.yml

service: yoga-example 
provider:
  name: aws
  runtime: nodejs14.x
  region: us-west-1
functions:
  graphql:
    handler: index.handler
    events:
    - http:
        path: /
        method: post
        cors: true
  playground:
    handler: index.playground
    events:
    - http:
        path: /
        method: get
        cors: true
```

### Upload to AWS with sls deploy

```bash
sls deploy
```

```yaml
service: yoga-example
stage: dev
region: us-west-1
stack: yoga-example-dev
resources: 16
api keys:
  None
endpoints:
  POST - https://vptcz65b06.execute-api.us-west-1.amazonaws.com/dev/
  GET - https://vptcz65b06.execute-api.us-west-1.amazonaws.com/dev/
functions:
  graphql: yoga-example-dev-graphql
  playground: yoga-example-dev-playground
layers:
  None
```

### Run test queries on GraphQL Yoga Serverless

Open [vptcz65b06.execute-api.us-west-1.amazonaws.com/dev/](https://vptcz65b06.execute-api.us-west-1.amazonaws.com/dev/) and run the `hello` query.

```graphql
query HELLO_QUERY { hello }
```

![17-graphql-yoga-serverless-framework](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/r11pxpmah3ny3s7vajns.png)

```bash
curl --request POST \
  --url https://vptcz65b06.execute-api.us-west-1.amazonaws.com/dev/ \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```

### GraphQL Yoga Serverless Final Project Structure

```
├── .gitignore
├── index.js
├── package.json
└── serverless.yml
```

## Deploy GraphQL Yoga with Amplify

```bash
mkdir graphql-yoga-amplify
cd graphql-yoga-amplify
amplify init
```

```
? Enter a name for the project ajcwebdevgraphqlyoga
The following configuration will be applied:

Project information
| Name: ajcwebdevgraphqlyoga
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

### Create backend with amplify add api

```bash
amplify add api
```

```
? Please select from one of the below mentioned services: REST
? Provide a friendly name for your resource to be used as a label for this category in the project: ajcwebdevyoga
? Provide a path (e.g., /book/{isbn}): /graphql
? Choose a Lambda source: Create a new Lambda function
? Provide the AWS Lambda function name: graphqlyoga
? Choose the function runtime that you want to use: NodeJS
? Choose the function template that you want to use: Hello World
? Do you want to access other resources created in this project from your Lambda function? N
? Do you want to edit the local lambda function now? N
? Restrict API access: N
? Do you want to add another path? N
```

```bash
cd amplify/backend/function/graphqlyoga/src
yarn add graphql-yoga
cd ../../../../../
```

### GraphQLServerLambda

```js
// amplify/backend/function/graphqlyoga/src/index.js

const { GraphQLServerLambda } = require('graphql-yoga')

const typeDefs = `type Query { hello: String }`
const resolvers = {
  Query: { hello: () => 'Hello from GraphQL Yoga on Amplify!' }
}

const lambda = new GraphQLServerLambda({ typeDefs, resolvers })

exports.handler = lambda.handler
```

### Upload to AWS with amplify push

Deploy the function containing the GraphQL API.

```bash
amplify push
```

```
✔ Successfully pulled backend environment dev from the cloud.

    Current Environment: dev
    
┌──────────┬───────────────┬───────────┬───────────────────┐
│ Category │ Resource name │ Operation │ Provider plugin   │
├──────────┼───────────────┼───────────┼───────────────────┤
│ Function │ graphqlyoga   │ Create    │ awscloudformation │
├──────────┼───────────────┼───────────┼───────────────────┤
│ Api      │ ajcwebdevyoga │ Create    │ awscloudformation │
└──────────┴───────────────┴───────────┴───────────────────┘
? Are you sure you want to continue? Yes

All resources are updated in the cloud

REST API endpoint: https://zmvy0jw9dc.execute-api.us-east-1.amazonaws.com/dev
```

CloudFormation

![18-graphql-yoga-amplify-cloudformation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2yx0g1wtx26wnclkk39a.png)

Lambda Function

![19-graphql-yoga-amplify-lambda-function](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sbtdj4u6t6elixltt8g5.png)

Lambda Application

![20-graphql-yoga-amplify-lambda-application](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zt8jw4vbdtdstpddrv5h.png)

API Gateway

![21-graphql-yoga-amplify-api-gateway](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tfbnbk98xs26daea38cg.png)

### Run test queries on GraphQL Yoga Amplify

Open [zmvy0jw9dc.execute-api.us-east-1.amazonaws.com/dev/graphql](https://zmvy0jw9dc.execute-api.us-east-1.amazonaws.com/dev/graphql) and run the `hello` query.

```graphql
query HELLO_QUERY { hello }
```

![22-graphql-yoga-amplify](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wzrjaxga93okfg8x4dqk.png)

```bash
curl --request POST \
  --url https://zmvy0jw9dc.execute-api.us-east-1.amazonaws.com/dev/graphql \
  --header 'content-type: application/json' \
  --data '{"query":"{ hello }"}'
```

### GraphQL Yoga Amplify Final Project Structure

```
├── .gitignore
└── amplify
    └── backend
        ├── api
        │   └── ajcwebdevyoga
        │       ├── ajcwebdevyoga-cloudformation-template.json
        │       ├── api-params.json
        │       └── parameters.json
        └── function
            └── graphqlyoga
                ├── amplify.state
                ├── graphqlyoga-cloudformation-template.json
                ├── function-parameters.json
                └── src
                    ├── event.json
                    ├── index.js
                    └── package.json
```

You're still here? Wow, didn't think anyone would actually make it to the end.