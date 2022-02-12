---
title: a first look at aws cdk
description: AWS Cloud Development Kit (CDK) is a framework for defining cloud infrastructure in code and provisioning it through AWS CloudFormation.
date: 2021-04-30
tags:
  - aws
  - cdk
  - cloudformation
  - s3
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hkvpkmk2urt0ld3eoa45.png
layout: layouts/post.njk
---

AWS [Cloud Development Kit (CDK)](https://aws.amazon.com/cdk/) is a software development framework for defining cloud infrastructure in code and provisioning it through AWS [CloudFormation](https://aws.amazon.com/cloudformation/). It supports TypeScript, JavaScript, Python, Java, C#/.Net, and (almost) Go.

Developers can use one of the supported programming languages to define reusable cloud components known as ***Constructs*** that are composed together into ***Stacks*** and ***Apps***.

![01-constructs](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n0mrliwpxev3ai1mba1n.png)

## Outline

* [Setup](#setup)
  * [Configure AWS CLI](#configure-aws-cli)
  * [Install CDK CLI](#install-cdk-cli)
  * [Create Project Directory](#create-project-directory)
  * [Initialize Project](#initialize-project)
  * [List Stacks](#list-stacks)
* [Project Structure](#project-structure)
  * [CDK Configuration](#cdk-configuration)
  * [App Entry Point](#app-entry-point)
  * [HelloCdkStack](#hellocdkstack)
* [Add S3 Bucket](#add-s3-bucket)
  * [Define S3 Bucket Construct](#define-s3-bucket-construct)
  * [Generate CloudFormation Template](#generate-cloudformation-template)
  * [Deploy Stack to AWS](#deploy-stack-to-aws)

## Setup

The code for this article can be found [on my GitHub](https://github.com/ajcwebdev/ajcwebdev-cdk).

### Configure AWS CLI

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

### Install CDK CLI

The `aws-cdk` can be [globally installed with `npm`](https://docs.aws.amazon.com/cdk/latest/guide/getting_started.html).

```bash
npm install -g aws-cdk
```

Check `aws-cdk` version.

```bash
cdk --version
```

Output:

```
1.118.0 (build a4f0418)
```

### Create Project Directory

Files will be generated based on the project name. This means you can't give your project a cute, personal name like `ajcwebdev-cdk` which I always do in all my tutorials but okay fine AWS I'll play by your rules.

```bash
mkdir hello-cdk
cd hello-cdk
```

### Initialize Project

```bash
cdk init app --language javascript
```

Output:

```
Applying project template app for javascript

# Welcome to your CDK JavaScript project!

This is a blank project for JavaScript development with CDK.

The `cdk.json` file tells the CDK Toolkit how to execute your app. The build step is not required when using JavaScript.

## Useful commands

* `npm run test`   perform the jest unit tests
* `cdk deploy`     deploy this stack to your default AWS account/region
* `cdk diff`       compare deployed stack with current state
* `cdk synth`      emits the synthesized CloudFormation template

Initializing a new git repository...
Executing npm install...
✅ All done!
```

### List Stacks

```bash
cdk ls
```

Output:

```
HelloCdkStack
```

## Project Structure

```
├── bin
│   └── hello-cdk.js
├── lib
│   └── hello-cdk-stack.js
├── test
│   └── hello-cdk-test.js
├── .gitignore
├── .npmignore
├── cdk.json
├── jest.config.js
├── package-lock.json
├── package.json
└── README.md
```

With one exception, our `package.json` should be straight forward if you have worked with `npm` before. It includes a few scripts and our dependencies.

```json
{
  "name": "hello-cdk",
  "version": "0.1.0",
  "bin": {
    "hello-cdk": "bin/hello-cdk.js"
  },
  "scripts": {
    "build": "echo \"The build step is not required when using JavaScript!\" && exit 0",
    "cdk": "cdk",
    "test": "jest"
  },
  "devDependencies": {
    "@aws-cdk/assert": "1.118.0",
    "aws-cdk": "1.118.0",
    "jest": "^26.4.2"
  },
  "dependencies": {
    "@aws-cdk/core": "1.118.0"
  }
}
```

It also includes a less well known (at least to me) option, the [`bin` field](https://docs.npmjs.com/cli/v7/configuring-npm/package-json#bin). Some packages have one or more executable files that need to be installed into the PATH. The `bin` field is a map of a command name to a local file name. On install, `npm` will symlink that file into `prefix/bin` for global installs, or `./node_modules/.bin/` for local installs.

### CDK Configuration

Many features of the CDK Toolkit require one or more AWS CloudFormation templates be synthesized, which in turn requires running your application. `cdk.json` uses a configuration option to specify the exact command necessary to run your app and is located in the main directory of your project.

```json
{
  "app": "node bin/hello-cdk.js",
  "context": {
    "@aws-cdk/aws-apigateway:usagePlanKeyOrderInsensitiveId": true,
    "@aws-cdk/core:enableStackNameDuplicates": "true",
    "aws-cdk:enableDiffNoFail": "true",
    "@aws-cdk/core:stackRelativeExports": "true",
    "@aws-cdk/aws-ecr-assets:dockerIgnoreSupport": true,
    "@aws-cdk/aws-secretsmanager:parseOwnedSecretName": true,
    "@aws-cdk/aws-kms:defaultKeyPolicies": true,
    "@aws-cdk/aws-s3:grantWriteWithoutAcl": true,
    "@aws-cdk/aws-ecs-patterns:removeDefaultDesiredCount": true,
    "@aws-cdk/aws-rds:lowercaseDbIdentifier": true,
    "@aws-cdk/aws-efs:defaultEncryptionAtRest": true,
    "@aws-cdk/aws-lambda:recognizeVersionProps": true,
    "@aws-cdk/aws-cloudfront:defaultSecurityPolicyTLSv1.2_2021": true
  }
}
```

Your configuration option can be specified using the `app` key. The CDK Toolkit provides an appropriate command when creating a new project with `cdk init`. The CDK Toolkit looks for `cdk.json` in the current working directory when attempting to run your app.

### App Entry Point

Files referenced in `bin` must start with `#!/usr/bin/env node` to make sure the scripts aren't started without the node executable.

```javascript
// bin/hello-cdk.js

#!/usr/bin/env node

const cdk = require('@aws-cdk/core')
const { HelloCdkStack } = require('../lib/hello-cdk-stack')

const app = new cdk.App()

new HelloCdkStack(
  app, 'HelloCdkStack', {}
)
```

### HelloCdkStack

The code that defines your stack goes inside the `constructor`, under `super`.

```javascript
// lib/hello-cdk-stack.js

const cdk = require('@aws-cdk/core');

class HelloCdkStack extends cdk.Stack {
  /**
   *
   * @param {cdk.Construct} scope
   * @param {string} id
   * @param {cdk.StackProps=} props
   */
  constructor(scope, id, props) {
    super(scope, id, props);
  }
}

module.exports = { HelloCdkStack }
```

## Add S3 Bucket

Right now the app doesn't do anything since the stack it contains doesn't define any resources. We can add an Amazon S3 bucket by installing the [aws-cdk/aws-s3](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-s3-readme.html) module from the [AWS Construct Library](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html).

```bash
npm i @aws-cdk/aws-s3
```

### Define S3 Bucket Construct

Inside your stack, initialize an `s3` variable by importing it from `@aws-cdk/aws-s3` with the [CommonJS `require()` syntax](https://en.wikipedia.org/wiki/CommonJS). Create `MyFirstBucket` with the [Bucket class](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html).

```javascript
// lib/hello-cdk-stack.js

const cdk = require('@aws-cdk/core');
const s3 = require('@aws-cdk/aws-s3');

class HelloCdkStack extends cdk.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    new s3.Bucket(this, 'MyFirstBucket', {
      versioned: true
    });
  }
}

module.exports = { HelloCdkStack }
```

Our dependencies in `package.json` now includes `@aws-cdk/aws-s3`.

```json
"dependencies": {
  "@aws-cdk/aws-s3": "^1.101.0",
  "@aws-cdk/core": "1.101.0"
}
```

### Generate CloudFormation Template

Synthesize an AWS CloudFormation template for the app. Hey put that [Moog](https://en.wikipedia.org/wiki/Moog_synthesizer) away, not that kind of synth!

```bash
cdk synth
```

This will output a CloudFormation file that will be mostly gibberish if you've never seen a [CloudFormation template](https://aws.amazon.com/cloudformation/resources/templates/) before.

```yaml
Resources:
  MyFirstBucketB8884501:
    Type: AWS::S3::Bucket
    Properties:
      VersioningConfiguration:
        Status: Enabled
    UpdateReplacePolicy: Retain
    DeletionPolicy: Retain
    Metadata:
      aws:cdk:path: HelloCdkStack/MyFirstBucket/Resource
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Metadata:
      aws:cdk:path: HelloCdkStack/CDKMetadata/Default
    Condition: CDKMetadataAvailable
Conditions:
  CDKMetadataAvailable:
    Fn::Or:
      - Fn::Or:
          - Fn::Equals:
              - Ref: AWS::Region
              - af-south-1
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-east-1
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-northeast-1
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-northeast-2
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-south-1
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-southeast-1
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-southeast-2
          - Fn::Equals:
              - Ref: AWS::Region
              - ca-central-1
          - Fn::Equals:
              - Ref: AWS::Region
              - cn-north-1
          - Fn::Equals:
              - Ref: AWS::Region
              - cn-northwest-1
      - Fn::Or:
          - Fn::Equals:
              - Ref: AWS::Region
              - eu-central-1
          - Fn::Equals:
              - Ref: AWS::Region
              - eu-north-1
          - Fn::Equals:
              - Ref: AWS::Region
              - eu-south-1
          - Fn::Equals:
              - Ref: AWS::Region
              - eu-west-1
          - Fn::Equals:
              - Ref: AWS::Region
              - eu-west-2
          - Fn::Equals:
              - Ref: AWS::Region
              - eu-west-3
          - Fn::Equals:
              - Ref: AWS::Region
              - me-south-1
          - Fn::Equals:
              - Ref: AWS::Region
              - sa-east-1
          - Fn::Equals:
              - Ref: AWS::Region
              - us-east-1
          - Fn::Equals:
              - Ref: AWS::Region
              - us-east-2
      - Fn::Or:
          - Fn::Equals:
              - Ref: AWS::Region
              - us-west-1
          - Fn::Equals:
              - Ref: AWS::Region
              - us-west-2
```

You'll eventually want to learn what all this junk means and how it works, but the reason [so](https://www.pulumi.com/docs/intro/concepts/) [many](https://aws.amazon.com/serverless/sam/) [libraries](https://docs.serverless-stack.com/faq) [exist](https://www.terraform.io/intro/vs/cloudformation.html) to abstract CloudFormation away or replace it is because it's a huge pain and no one wants to write it. We now treat it moreso as a compile target because [compilers are the new frameworks](https://tomdale.net/2017/09/compilers-are-the-new-frameworks/).

### Deploy Stack to AWS

We will use the AWS CDK Toolkit to deploy our project. However, since `cdk synth` generates valid AWS CloudFormation templates we could take it and deploy it using the AWS CloudFormation console or other tools.

```bash
cdk deploy
```

Output with `account-id` and `resource-id` redacted:

```
HelloCdkStack: deploying...
HelloCdkStack: creating CloudFormation changeset...

✅  HelloCdkStack

Stack ARN:
arn:aws:cloudformation:us-west-1:1234:stack/HelloCdkStack/1234
```

We'll figure out what an [ARN](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html) is/does in future articles, but basically it's an AWS [NFT](https://en.wikipedia.org/wiki/Non-fungible_token).

>A non-fungible token is a unit of data that certifies a digital asset to be unique and therefore not interchangeable.