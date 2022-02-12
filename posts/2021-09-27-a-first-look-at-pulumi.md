---
title: a first look at pulumi
description: Pulumi provides open source infrastructure as code SDKs that enable you to create, deploy, and manage infrastructure on numerous popular clouds in multiple programming languages.
date: 2021-09-27
tags:
  - pulumi
  - iac
  - aws
  - lambda
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/593lhbmsyahk78tfgqdp.png
layout: layouts/post.njk
---

[Pulumi](https://www.pulumi.com) provides open source infrastructure as code SDKs that enable you to create, deploy, and manage infrastructure on numerous popular clouds in multiple programming languages.

## Setup

In this tutorial, we’ll show you how to write a Pulumi program that creates a serverless app serving static content with dynamic routes in AWS Lambda. All the code for this example can be found [on my GitHub](https://github.com/ajcwebdev/ajcwebdev-pulumi).

### Install Pulumi CLI

Instructions for downloading the CLI will [vary depending on your operating system](https://www.pulumi.com/docs/get-started/install/#installing-pulumi). This tutorial will use [Homebrew](https://brew.sh/) on MacOS.

```bash
brew install pulumi
```

Subsequent updates can be installed with `brew upgrade`.

```bash
brew upgrade pulumi
```

### Configure AWS Credentials

Make sure you have the [AWS CLI](https://aws.amazon.com/cli/) installed and an [AWS account](https://aws.amazon.com/). For general use, [`aws configure`](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html) is recommended as the fastest way to set up your AWS CLI installation.

```bash
aws configure
```

When you enter this command, the AWS CLI prompts you for four pieces of information:
* Access key ID
* Secret access key
* AWS Region
* Output format

Go to [My Security Credentials](https://console.aws.amazon.com/iam/home?#/security_credentials) to find your Access Key ID, Secret Access Key, and default region. You can leave the output format blank.

```
AWS Access Key ID: <YOUR_ACCESS_KEY_ID>
AWS Secret Access Key: <YOUR_SECRET_ACCESS_KEY>
Default region name: <YOUR_REGION_NAME>
Default output format [None]: 
```

### Login to Pulumi

```bash
pulumi login
```

You will be asked to hit `<ENTER>` to log in with your browser.

```
Manage your Pulumi stacks by logging in.

Run `pulumi login --help` for alternative login options.

Enter your access token from https://app.pulumi.com/account/tokens
    or hit <ENTER> to log in using your browser
```

After logging in you will get the following output:

```
Welcome to Pulumi!

Pulumi helps you create, deploy, and manage infrastructure on any cloud using
your favorite language. You can get started today with Pulumi at:

  https://www.pulumi.com/docs/get-started/

Tip of the day: Resources you create with Pulumi are given unique names (a randomly
generated suffix) by default. To learn more about auto-naming or customizing resource
names see https://www.pulumi.com/docs/intro/concepts/resources/#autonaming.

Logged in to pulumi.com as ajcwebdev (https://app.pulumi.com/ajcwebdev)
```

## Create a New Pulumi Project

We'll use the [pulumi new](https://www.pulumi.com/docs/reference/cli/pulumi_new/) command and generate a new project with the `hello-aws-javascript` template. It will be named `ajcwebdev-pulumi` with the `--name` flag.

```bash
mkdir ajcwebdev-pulumi
cd ajcwebdev-pulumi
pulumi new hello-aws-javascript --name ajcwebdev-pulumi
```

You will be asked to provide a description, stack name, and AWS region.

```
This command will walk you through creating a new Pulumi project.

Enter a value or leave blank to accept the (default), and press <ENTER>.
Press ^C at any time to quit.

project description: (A simple AWS serverless JavaScript Pulumi program) 
Created project 'ajcwebdev-pulumi'

Please enter your desired stack name.
To create a stack in an organization, use the format
<org-name>/<stack-name> (e.g. `acmecorp/dev`).

stack name: (dev) 
Created stack 'dev'

aws:region: The AWS region to deploy into: (us-east-1) us-west-1
Saved config
```

I selected the default option for the description and stack name, but changed the region from `us-east-1` to `us-west-1` because west coast best coast.

```
Installing dependencies...

> @pulumi/docker@3.1.0 install /Users/ajcwebdev/ajcwebdev-pulumi/node_modules/@pulumi/docker
> node scripts/install-pulumi-plugin.js resource docker v3.1.0

[resource plugin docker-3.1.0] installing
Downloading plugin: 16.13 MiB / 16.13 MiB [=========================] 100.00% 0s

> @pulumi/aws@4.21.2 install /Users/ajcwebdev/ajcwebdev-pulumi/node_modules/@pulumi/aws
> node scripts/install-pulumi-plugin.js resource aws v4.21.2

[resource plugin aws-4.21.2] installing
Downloading plugin: 80.45 MiB / 80.45 MiB [=========================] 100.00% 3s

> protobufjs@6.11.2 postinstall /Users/ajcwebdev/ajcwebdev-pulumi/node_modules/protobufjs
> node scripts/postinstall

> aws-sdk@2.995.0 postinstall /Users/ajcwebdev/ajcwebdev-pulumi/node_modules/aws-sdk
> node scripts/check-node-version.js

added 122 packages from 238 contributors and audited 122 packages in 12.274s

29 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities

Finished installing dependencies

Your new project is ready to go! ✨

To perform an initial deployment, run 'pulumi up'
```

### Pulumi Yaml Files

A Pulumi project is any folder which contains a `Pulumi.yaml` file specifying metadata about your project. The project file must begin with a capitalized P and can use either `.yml` or `.yaml` extensions.

```yaml
# Pulumi.yaml

name: ajcwebdev-pulumi
runtime: nodejs
description: A simple AWS serverless JavaScript Pulumi program
```

The key-value pairs for any given stack are stored in your project’s stack settings file, which is automatically named `Pulumi.<stack-name>.yaml`.

```yaml
# Pulumi.dev.yaml

config:
  aws:region: us-west-1
```

### Index File

Import the [pulumi/aws](https://pulumi.io/reference/pkg/nodejs/@pulumi/aws/index.html) package.

```js
// index.js

const pulumi = require("@pulumi/pulumi");
const aws = require("@pulumi/aws");
const awsx = require("@pulumi/awsx");
```

Create a public HTTP endpoint using AWS API Gateway that serves static files from the `www` folder using AWS S3 with a REST API served on `GET /name` with AWS Lambda.

```js
// index.js

const endpoint = new awsx.apigateway.API("hello", {
  routes: [
    {
      path: "/",
      localPath: "www",
    },
    {
      path: "/source",
      method: "GET",
      eventHandler: (req, ctx, cb) => {
        cb(undefined, {
          statusCode: 200,
          body: Buffer.from(JSON.stringify({ name: "AWS" }), "utf8").toString("base64"),
          isBase64Encoded: true,
          headers: { "content-type": "application/json" },
        })
      },
    },
  ],
})
```

Export the public URL for the HTTP service with `exports.url = endpoint.url`.

```js
// index.js

exports.url = endpoint.url
```

Full `index.js` file:

```js
const pulumi = require("@pulumi/pulumi")
const aws = require("@pulumi/aws")
const awsx = require("@pulumi/awsx")

const endpoint = new awsx.apigateway.API("hello", {
  routes: [
    {
      path: "/",
      localPath: "www",
    },
    {
      path: "/source",
      method: "GET",
      eventHandler: (req, ctx, cb) => {
        cb(undefined, {
          statusCode: 200,
          body: Buffer.from(JSON.stringify({ name: "AWS" }), "utf8").toString("base64"),
          isBase64Encoded: true,
          headers: { "content-type": "application/json" },
        })
      },
    },
  ],
})

exports.url = endpoint.url
```

### HTML Index File

The project comes with a hello world example in the `index.html` file.

```html
<!-- www/index.html -->

<!DOCTYPE html>

<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello Pulumi</title>
    <link rel="shortcut icon" href="favicon.png" type="image/png">
  </head>

  <body>
    <p>Hello, world!</p>
    <p>Made with ❤️ using <a href="https://pulumi.com">Pulumi</a></p>
    <p>Served from: <span id="source"></span></p>
  </body>

  <script>
    fetch("source").then(response => response.json()).then(json => {
      document.getElementById("source").innerText = json.name;
    });
  </script>
</html>
```

## Deploy to Pulumi Cloud

Create or update the resources in a stack with `pulumi up`.

```bash
pulumi up
```

```
Previewing update (dev)

View Live: https://app.pulumi.com/ajcwebdev/ajcwebdev-pulumi/dev/previews/ded2506b-97b6-4a20-9018-99ef77316769

     Type                                Name                       Plan       
 +   pulumi:pulumi:Stack                 ajcwebdev-pulumi-dev       create     
 +   └─ aws:apigateway:x:API             hello                      create     
 +      ├─ aws:iam:Role                  hello4c238266              create     
 +      ├─ aws:s3:Bucket                 hello                      create     
 +      ├─ aws:iam:Role                  hello66382ec5              create     
 +      ├─ aws:lambda:Function           hello66382ec5              create     
 +      ├─ aws:iam:RolePolicyAttachment  hello4c238266              create     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-019020e7     create     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-b5aeb6b6     create     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-e1a3786d     create     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-74d12784     create     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-6c156834     create     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-1b4caae3     create     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-7cd09230     create     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-a1de8170     create     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-4aaabb8e     create     
 +      ├─ aws:s3:BucketObject           hello4c238266/index.html   create     
 +      ├─ aws:s3:BucketObject           hello4c238266/favicon.png  create     
 +      ├─ aws:apigateway:RestApi        hello                      create     
 +      ├─ aws:apigateway:Deployment     hello                      create     
 +      ├─ aws:lambda:Permission         hello-f61e7551             create     
 +      └─ aws:apigateway:Stage          hello                      create     
 
Resources:
    + 22 to create

Do you want to perform this update?  [Use arrows to move, enter to select, type to filter]
> yes
  no
  details
```

Select yes.

```
Do you want to perform this update? yes
Updating (dev)

View Live: https://app.pulumi.com/ajcwebdev/ajcwebdev-pulumi/dev/updates/1

     Type                                Name                       Status      
 +   pulumi:pulumi:Stack                 ajcwebdev-pulumi-dev       created     
 +   └─ aws:apigateway:x:API             hello                      created     
 +      ├─ aws:iam:Role                  hello4c238266              created     
 +      ├─ aws:s3:Bucket                 hello                      created     
 +      ├─ aws:iam:Role                  hello66382ec5              created     
 +      ├─ aws:lambda:Function           hello66382ec5              created     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-019020e7     created     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-1b4caae3     created     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-e1a3786d     created     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-b5aeb6b6     created     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-4aaabb8e     created     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-a1de8170     created     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-6c156834     created     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-74d12784     created     
 +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-7cd09230     created     
 +      ├─ aws:iam:RolePolicyAttachment  hello4c238266              created     
 +      ├─ aws:s3:BucketObject           hello4c238266/index.html   created     
 +      ├─ aws:s3:BucketObject           hello4c238266/favicon.png  created     
 +      ├─ aws:apigateway:RestApi        hello                      created     
 +      ├─ aws:apigateway:Deployment     hello                      created     
 +      ├─ aws:lambda:Permission         hello-f61e7551             created     
 +      └─ aws:apigateway:Stage          hello                      created     
 
Outputs:
    url: "https://2inuue6w0a.execute-api.us-west-1.amazonaws.com/stage/"

Resources:
    + 22 created

Duration: 26s
```

Open [2inuue6w0a.execute-api.us-west-1.amazonaws.com/stage/](https://2inuue6w0a.execute-api.us-west-1.amazonaws.com/stage/).

![01-pulumi-boilerplate](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2f3qi23p9307ig3t0bqc.png)

### Update HTML File

Change stuff.

```html
<!-- www/index.html -->

<!DOCTYPE html>

<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>ajcwebdev-pulumi</title>
    <link rel="shortcut icon" href="favicon.png" type="image/png">
  </head>

  <body>
    <h2>ajcwebdev-pulumi</h2>
    <p>Served from: <span id="source"></span></p>
  </body>

  <script>
    fetch("source").then(response => response.json()).then(json => {
      document.getElementById("source").innerText = json.name;
    });
  </script>
</html>
```

Run `pulumi up` again to deploy your changes.

```bash
pulumi up
```

```
Previewing update (dev)

View Live: https://app.pulumi.com/ajcwebdev/ajcwebdev-pulumi/dev/previews/255daf60-eb38-43d3-b540-147bbbaa7093

     Type                       Name                      Plan       Info
     pulumi:pulumi:Stack        ajcwebdev-pulumi-dev                 
     └─ aws:apigateway:x:API    hello                                
 ~      └─ aws:s3:BucketObject  hello4c238266/index.html  update     [diff: ~source]
 
Resources:
    ~ 1 to update
    21 unchanged

Do you want to perform this update? yes
Updating (dev)

View Live: https://app.pulumi.com/ajcwebdev/ajcwebdev-pulumi/dev/updates/3

     Type                       Name                      Status      Info
     pulumi:pulumi:Stack        ajcwebdev-pulumi-dev                  
     └─ aws:apigateway:x:API    hello                                 
 ~      └─ aws:s3:BucketObject  hello4c238266/index.html  updated     [diff: ~source]
 
Outputs:
    url: "https://2inuue6w0a.execute-api.us-west-1.amazonaws.com/stage/"

Resources:
    ~ 1 updated
    21 unchanged

Duration: 4s
```

Check back to [2inuue6w0a.execute-api.us-west-1.amazonaws.com/stage/](https://2inuue6w0a.execute-api.us-west-1.amazonaws.com/stage/).

![02-ajcwebdev-pulumi](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2puhpc364ba4uhxmffyu.png)