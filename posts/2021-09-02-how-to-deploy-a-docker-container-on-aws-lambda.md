---
title: how to deploy a docker container on aws lambda
description: This example shows how to use the Serverless Framework to deploy a Docker container on AWS Lambda.
date: 2021-09-02
tags:
  - aws
  - lambda
  - docker
  - serverless
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/87ws4w1gmynplrnouvsd.png
layout: layouts/post.njk
---

AWS Lambda is easy to use and manage because it has an execution environment with a specific runtime on a known environment. But this also causes problems if you have a use case outside the predetermined environments. To address these issues, AWS introduced [Lambda layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html) for packaging zip files with libraries and dependencies needed in your Lambda functions.

But [Lambda layers still had limitations](https://lumigo.io/blog/lambda-layers-when-to-use-it/) such as testing, static analysis, and versioning. In December 2020, [AWS Lambda released Docker container support](https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/). The Serverless framework has expanded on this with the [following example](https://www.serverless.com/blog/container-support-for-lambda) designed to make it easier to use this new feature.

## Outline

* [Create template with serverless create](#create-template-with-serverless-create)
  * [Complete Serverless YAML File](#complete-serverless-yaml-file)
  * [Dockerfile](#dockerfile)
  * [Function Handler](#function-handler)
* [Deploy with serverless deploy](#deploy-with-serverless-deploy)
  * [Invoke the hello function with serverless invoke](#invoke-the-hello-function-with-serverless-invoke)

## Create template with serverless create

We will use this starter template to generate a boilerplate already configured and setup in our `serverless.yml` file. All the code for this project can be found [on my GitHub](https://github.com/ajcwebdev/ajcwebdev-docker-lambda).

```bash
serverless create --template aws-nodejs-docker \
  --path ajcwebdev-docker-lambda
```

The Serverless Framework lets you define a Dockerfile and point at it in `serverless.yml`. The Framework makes sure the container is available in ECR and that it is setup and configured for Lambda.

```yaml
service: ajcwebdev-docker-lambda
frameworkVersion: '2'
```

Under `provider` is an `ecr` section for defining images that will be built locally and uploaded to ECR. The entire packaging process occurs in the context of the container. AWS uses your Docker configuration to build, optimize and prepare your container for use in Lambda.

```yaml
provider:
  name: aws
  lambdaHashingVersion: 20201221
  ecr:
    images:
      appimage:
        path: ./
```

The `functions` section tells the framework what the image reference name is (`appimage`) that we can use elsewhere in our configuration, and where the content of the Docker image resides with the `path` property. A `Dockerfile` of some type should reside in the specified folder containing the executable code for our function.

```yaml
functions:
  hello:
    image:
      name: appimage
```

We use the same value for `image.name` as we do for the image we defined, `appimage`. It can be named anything as long as the same value is referenced.

### Complete Serverless YAML File

```yaml
service: ajcwebdev-docker-lambda
frameworkVersion: '2'
provider:
  name: aws
  lambdaHashingVersion: 20201221
  ecr:
    images:
      appimage:
        path: ./
functions:
  hello:
    image:
      name: appimage
```

### Dockerfile

The easiest way to setup a Lambda ready Docker image is to use base images provided by AWS. The [AWS ECR Gallery](https://gallery.ecr.aws/) contains a list of all available images. We are using the Node [v12 image](https://gallery.ecr.aws/lambda/nodejs:12).

```dockerfile
FROM public.ecr.aws/lambda/nodejs:12

COPY app.js ./

CMD ["app.handler"]
```

The `CMD` property defines a file called `app.js` with a function called `handler`.

### Function Handler

In our service's directory, we have a file called `app.js` with the function inside.

```js
// app.js

'use strict'

module.exports.handler = async (event) => {
  return {
    statusCode: 200,
    body: JSON.stringify(
      {
        message: `Cause I don't want a server, but I do still want a container`
      },
      null,
      2
    ),
  }
}
```

`app.js` returns a JSON object containing a message clarifying exactly why anyone would ever want this thing in the first place.

## Deploy with serverless deploy

We are now able to generate our container, deploy it to ECR and execute functions. In order to build images locally and push them to ECR, you need to have Docker [installed](https://docs.docker.com/get-docker/) and running on your local machine.

```bash
cd ajcwebdev-docker-lambda
serverless deploy
```

The `serverless deploy` command deploys your entire service via CloudFormation. After the deployment finishes you will be provided the information for your service.

```
Service Information

service: ajcwebdev-docker-lambda
stage: dev
region: us-east-1
stack: ajcwebdev-docker-lambda-dev
resources: 6
api keys:
  None
endpoints:
  None
functions:
  hello: ajcwebdev-docker-lambda-dev-hello
layers:
  None
```

### Invoke the hello function with serverless invoke

The `serverless invoke` command invokes deployed function and allows sending event data to the function, reading logs and displaying other important information about the function invocation.

```bash
serverless invoke --function hello
```

If your function/container was deployed correctly then you will receive the following message:

```json
{
  "statusCode": 200,
  "body": "{\n  \"message\": \"Cause I don't want a server, but I do still want a container\"\n}"
}
```