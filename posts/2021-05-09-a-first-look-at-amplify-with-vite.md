---
title: a first look at amplify with vite
description: AWS Amplify is a set of tools and services to help frontend web and mobile developers build scalable fullstack applications with AWS infrastructure.
date: 2021-05-09
tags:
  - aws
  - amplify
  - react
  - vite
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fw4ka27ogxkv4g5nd0p9.jpg
layout: layouts/post.njk
---

[AWS Amplify](https://aws.amazon.com/amplify/) is a set of tools and services to help frontend web and mobile developers build scalable fullstack applications with AWS infrastructure that includes:
* [Amplify Console](https://console.aws.amazon.com/amplify/home) for managing frontend web app, backend environments, CI/CD, and the Admin UI
* [Amplify CLI](https://docs.amplify.aws/cli) for creating and deploying [CloudFormation stacks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacks.html)
* Open source libraries for [JavaScript](https://github.com/aws-amplify/amplify-js), [iOS](https://github.com/aws-amplify/aws-sdk-ios), [Android](https://github.com/aws-amplify/aws-sdk-android), and [Flutter](https://github.com/aws-amplify/amplify-flutter)
* [Admin UI](https://sandbox.amplifyapp.com/getting-started) for modeling data, adding authentication, authorization and managing users and groups

With these tools combined, Amplify has achieved a unique synecdoche¹ by creating their own opinionated version of AWS within AWS itself.

## Outline

* [Setup](#setup)
  * [Configure AWS CLI](#configure-aws-cli)
  * [Install Amplify CLI](#install-amplify-cli)
  * [Initialize Project](#initialize-project)
  * [Start Development Server](#start-development-server)
* [Project Structure](#project-structure)
  * [App Component](#app-component)
* [Initialize Amplify Project](#initialize-amplify-project)
  * [Deploy to CloudFront and S3](#deploy-to-cloudfront-and-s3)

## Setup

This tutorial will follow the Getting Started guide for React from the [Amplify documentation](https://docs.amplify.aws/start/q/integration/react), except we will be using Vite and the [officially supported](https://vitejs.dev/guide/#scaffolding-your-first-vite-project) [React template](https://github.com/vitejs/vite/tree/main/packages/create-app/template-react) instead of [create-react-app](https://github.com/facebook/create-react-app). This way the kids will know that I am, "with it." The code for this article can be found on my [GitHub](https://github.com/ajcwebdev/ajcwebdev-amplify).

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

### Install Amplify CLI

```bash
npm install -g @aws-amplify/cli
```

### Initialize Project

You need a React app, which means first you have to create one. You could do the sensible thing and use a tool literally called [create-react-app](https://reactjs.org/docs/create-a-new-react-app.html), or you could listen to a random person on the internet and use this thing you've never heard of and have no idea how to pronounce. If you're not convinced you can refer to [this blog post](https://dev.to/ajcwebdev/a-first-look-at-vite-m8n) from the aforementioned random person on the internet.

[Vite](https://vitejs.dev/) (French word for "fast", pronounced `/vit/`, rhymes with "street") is a build tool that aims to provide a faster and leaner development experience for modern web projects. It consists of two parts:

* A dev server with Hot Module Replacement (HMR)
* A build command that bundles your code with [Rollup](https://rollupjs.org), pre-configured to output highly optimized static assets for production

```bash
yarn create @vitejs/app ajcwebdev-amplify --template react
```

Output:

```
success Installed "@vitejs/create-app@2.2.5" with binaries:
      - create-app
      - cva

Scaffolding project in /Users/ajcwebdev/ajcwebdev-amplify...

Done. Now run:

  cd ajcwebdev-amplify
  yarn
  yarn dev

✨  Done in 2.18s.
```

### Start Development Server

```bash
cd ajcwebdev-amplify
yarn
yarn dev
```

```
  vite v2.2.4 dev server running at:

  > Local:    http://localhost:3000/
  > Network:  http://10.0.0.175:3000/

  ready in 668ms.
```

![01-create-vite-app](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fdvsb936oo4qlhq2fw12.png)

You could click the counter. Or you could not click the counter and instead live the rest of life wondering whether it actually would have incremented or not.

## Project Structure

```
├── src
│   ├── App.css
│   ├── App.jsx
│   ├── index.css
│   └── main.jsx
├── .gitignore
├── index.html
├── package.json
├── vite.config.js
└── yarn.lock
```

Our `package.json` includes scripts for starting the development server, building for production, and serving local previews of production builds.

```json
{
  "name": "ajcwebdev-amplify",
  "version": "0.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "serve": "vite preview"
  },
  "dependencies": {
    "react": "^17.0.0",
    "react-dom": "^17.0.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react-refresh": "^1.3.1",
    "vite": "^2.2.3"
  }
}
```

### App Component

```jsx
// src/App.jsx

import React, { useState } from 'react'
import logo from './logo.svg'
import './App.css'

function App() {
  const [count, setCount] = useState(0)

  return (
    <div className="App">
      <header className="App-header">
        <img
          src={logo}
          className="App-logo"
          alt="logo"
        />

        <p>Hello Vite + React!</p>

        <p>
          <button onClick={() => setCount((count) => count + 1)}>
            count is: {count}
          </button>
        </p>

        <p>
          Edit <code>App.jsx</code> and save to test HMR updates.
        </p>
        
        <p>
          <a
            className="App-link"
            href="https://reactjs.org"
            target="_blank"
            rel="noopener noreferrer"
          >
            Learn React
          </a>

          {' | '}

          <a
            className="App-link"
            href="https://vitejs.dev/guide/features.html"
            target="_blank"
            rel="noopener noreferrer"
          >
            Vite Docs
          </a>
        </p>
      </header>
    </div>
  )
}

export default App
```

Include links to your own social accounts.

```jsx
// src/App.jsx

import React from 'react'
import logo from './logo.svg'
import './App.css'

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img
          src={logo}
          className="App-logo"
          alt="logo"
        />

        <p>ajcwebdev</p>

        <p>Amplify + Vite</p>

        <p>
          <a
            href="https://dev.to/ajcwebdev"
            target="_blank"
            rel="noopener noreferrer"
          >
            Blog
          </a>

          {' | '}

          <a
            href="https://github.com/ajcwebdev"
            target="_blank"
            rel="noopener noreferrer"
          >
            GitHub
          </a>
        </p>
      </header>
    </div>
  )
}

export default App
```

![02-create-vite-app-edited](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qh6elff8oc10tnywje4w.png)

## Initialize Amplify Project

To initialize a new Amplify project, run `amplify init` from the root directory of your frontend app.

```bash
amplify init
```

```
? Enter a name for the project ajcwebdevamplify
? Enter a name for the environment dev
? Choose your default editor: Visual Studio Code
? Choose the type of app that you're building javascript

Please tell us about your project
? What javascript framework are you using react
? Source Directory Path:  src
? Distribution Directory Path: dist
? Build Command:  yarn build
? Start Command: yarn dev

Using default provider  awscloudformation

? Select the authentication method you want to use: AWS profile

For more information on AWS Profiles, see:
https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html

? Please choose the profile you want to use default
```

### Deploy to CloudFront and S3

```bash
amplify add hosting
```

```
? Select the plugin module to execute Amazon CloudFront and S3
? Select the environment setup: DEV (S3 only with HTTP)
? hosting bucket name ajcwebdevamplify-20210509181751-hostingbucket
? index doc for the website index.html
? error doc for the website index.html

You can now publish your app using the following command:
Command: amplify publish
```

```bash
amplify publish
```

```
✔ Successfully pulled backend environment dev from the cloud.

Current Environment: dev

| Category | Resource name   | Operation | Provider plugin   |
| -------- | --------------- | --------- | ----------------- |
| Hosting  | S3AndCloudFront | Create    | awscloudformation |

? Are you sure you want to continue? Yes
```

Output:

```
All resources are updated in the cloud

Hosting endpoint:
http://ajcwebdevamplify-20210509181751-hostingbucket-dev.s3-website-us-west-1.amazonaws.com

yarn run v1.22.10
warning package.json: No license field
$ vite build
vite v2.2.4 building for production...
✓ 21 modules transformed.
dist/assets/favicon.17e50649.svg   1.49kb
dist/assets/logo.ecc203fb.svg      2.61kb
dist/index.html                    0.51kb
dist/assets/index.0673ce28.css     0.76kb / brotli: 0.40kb
dist/assets/index.e0cc5c52.js      1.32kb / brotli: 0.55kb
dist/assets/vendor.de62f314.js     127.23kb / brotli: 36.03kb
✨  Done in 1.87s.

frontend build command exited with code 0
Publish started for S3AndCloudFront
✔ Uploaded files successfully.
Your app is published successfully.

http://ajcwebdevamplify-20210509181751-hostingbucket-dev.s3-website-us-west-1.amazonaws.com
```

You will then be taken to the [very memorably named endpoint](http://ajcwebdevamplify-20210509181751-hostingbucket-dev.s3-website-us-west-1.amazonaws.com/) for your S3 bucket.

![03-create-vite-app-hosted](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5efaiwxvekngq4uhii7w.png)

[[1]](https://en.wikipedia.org/wiki/Synecdoche) - wherein a part of something represents the whole, or vice versa.