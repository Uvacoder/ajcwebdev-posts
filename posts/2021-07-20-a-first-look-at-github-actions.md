---
title: a first look at github actions
description: GitHub Actions can be used to automate, customize, and execute software development workflows from within a GitHub repository.
date: 2021-07-20
tags:
  - github
  - actions
  - cicd
  - workflows
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9xshohp5ofx2w2tow67w.png
layout: layouts/post.njk
---

[GitHub Actions](https://docs.github.com/en/actions) can be used to automate, customize, and execute software development workflows from within a GitHub repository. You can discover, create, and share actions to perform CI/CD jobs and combine actions in a customized workflow.

## Create an Action

We will create a boilerplate workflow to help you get started with Actions.

```bash
mkdir -p ajcwebdev-actions/.github/workflows
cd ajcwebdev-actions
touch .github/workflows/hello.yml
echo '.DS_Store' > .gitignore
```

`on` controls when the workflow will run. `push` and `pull_request` events trigger the workflow but only for the main branch. `workflow_dispatch` allows you to run this workflow manually from the Actions tab.

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:
```

A workflow run is made up of one or more `jobs` that can run sequentially or in parallel. Our workflow contains a single job called `build` that is running on `ubuntu-latest`.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
```

`steps` represent a sequence of tasks that will be executed as part of the job. `uses` checks-out your repository under `$GITHUB_WORKSPACE`, so your job can access it. `run` will run a single command (`echo "Hello from GitHub Actions"`) which will print `Hello from GitHub Actions` using the runner's shell.

```yaml
    steps:
      - uses: actions/checkout@v2
      - name: Run a one-line script
        run: echo "Hello from GitHub Actions"
```

The action will then run a multi-line script to print a series of messages containing common environment variables such as the repository name and job status.

```yaml
      - name: Run a multi-line script
        run: |
          echo "Job was triggered by a ${{ github.event_name }} event."
          echo "Job is now running on a ${{ runner.os }} server hosted on GitHub."
          echo "The branch name is ${{ github.ref }}."
          echo "The repository name is ${{ github.repository }}."
          echo "Job status is ${{ job.status }}."
```

Here is the complete GitHub Action.

```yaml
# .github/workflows/hello.yml

name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run a one-line script
        run: echo "Hello from GitHub Actions"
      - name: Run a multi-line script
        run: |
          echo "Job was triggered by a ${{ github.event_name }} event."
          echo "Job is now running on a ${{ runner.os }} server hosted on GitHub."
          echo "The branch name is ${{ github.ref }}."
          echo "The repository name is ${{ github.repository }}."
          echo "Job status is ${{ job.status }}."
```

## Push your project to a GitHub repository

Initialize the repository, add all changes to the staging area, and commit all staged changes.

```bash
git init
git add .
git commit -m "Action Jackson"
```

### Create a new blank repository

You can create a blank repository by visiting [repo.new](https://repo.new) or using the [`gh repo create`](https://cli.github.com/manual/gh_repo_create) command with the [GitHub CLI](https://cli.github.com/). Enter the following command to create a new repository, set the remote name from the current directory, and push the project to the newly created repository.

```bash
gh repo create ajcwebdev-actions \
  --public \
  --source=. \
  --description="An Example GitHub Action Project" \
  --remote=upstream \
  --push
```

If you created a repository from the GitHub website instead of the CLI then you will need to set the remote and push the project with the following commands.

```bash
git remote add origin https://github.com/ajcwebdev/ajcwebdev-actions.git
git push -u origin main
```

Go to the [actions](https://github.com/ajcwebdev/ajcwebdev-actions/actions) tab on your GitHub repository to see your action.

![01-actions-tab](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mj69m00h1i0xr6cgp8o7.png)

Click your action to see the specific workflow.

![02-action-summary](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ly6c0e9yqyrokel278l7.png)

Click "build" to see more details.

![03-build-info](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gkpq3ymt5qcbk29bu2gr.png)

Click "Set up job" for more info.

```
Current runner version: '2.285.1'
Operating System
  Ubuntu
  20.04.3
  LTS

Virtual Environment
  Environment: ubuntu-20.04
  Version: 20211219.1
  Included Software: https://github.com/actions/virtual-environments/blob/ubuntu20/20211219.1/images/linux/Ubuntu2004-README.md
  Image Release: https://github.com/actions/virtual-environments/releases/tag/ubuntu20%2F20211219.1

Virtual Environment Provisioner
  1.0.0.0-main-20211214-1

GITHUB_TOKEN Permissions
  Actions: write
  Checks: write
  Contents: write
  Deployments: write
  Discussions: write
  Issues: write
  Metadata: read
  Packages: write
  Pages: write
  PullRequests: write
  RepositoryProjects: write
  SecurityEvents: write
  Statuses: write

Secret source: Actions
Prepare workflow directory
Prepare all required actions
Getting action download info
Download action repository 'actions/checkout@v2' (SHA:ec3a7ce113134d7a93b817d10a8272cb61118579)
```

Click "Run actions/checkout@v2" for more info.

```
Syncing repository: ajcwebdev/ajcwebdev-actions
Getting Git version info
Deleting the contents of '/home/runner/work/ajcwebdev-actions/ajcwebdev-actions'
Initializing the repository
Disabling automatic garbage collection
Setting up auth
Fetching the repository
Determining the checkout info
Checking out the ref
/usr/bin/git log -1 --format='%H'
'14b8a71b0852e18cd03880acad4cf4558b3bd0bd'
```

![04-action-output](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/szzpmt3s68cea1il4bkm.png)

## Create JavaScript Action

Our action will print "Hello World" in the logs or "Hello [who-to-greet]" if you provide a custom name. Initialize a `package.json` file with `npm`.

```bash
npm init -y
```

### Add Actions Toolkit Packages

This guide uses the GitHub Actions Toolkit Node.js module, [`actions/toolkit`](https://github.com/actions/toolkit), to speed up development. The actions toolkit is a collection of Node.js packages for building JavaScript actions with more consistency. Install the actions toolkit `core` and `github` packages.

```bash
npm i @actions/core @actions/github
```

The toolkit [`@actions/core`](https://github.com/actions/toolkit/tree/main/packages/core) package provides an interface to the workflow commands, input and output variables, exit statuses, and debug messages. The toolkit also offers a [`@actions/github`](https://github.com/actions/toolkit/tree/main/packages/github) package that returns an authenticated Octokit REST client and access to GitHub Actions contexts.

### Create Action Metadata File

Docker and JavaScript actions require a [metadata file](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions) containing data that defines the inputs, outputs and main entrypoint for the action. Create a file called `action.yml` for our action.

```bash
touch action.yml
```

The metadata filename must be either `action.yml` or `action.yaml`. This file defines the `who-to-greet` input and `time` output. It also tells the action runner how to start running this JavaScript action.

```yaml
# action.yml

name: 'Hello World'
description: 'Greet someone and record the time for posterity'
inputs:
  who-to-greet:
    description: 'Who to greet'
    required: true
    default: 'World'
outputs:
  time:
    description: 'The time at which the greeting commenced'
runs:
  using: 'node12'
  main: 'index.js'
```

### Create JavaScript Action File

Create a file called `index.js` for the JavaScript code.

```bash
touch index.js
```

`index.js` will contain the JavaScript code to interact with our action metadata.

```js
// index.js

const core = require('@actions/core')
const github = require('@actions/github')

try {
  const nameToGreet = core.getInput('who-to-greet')
  console.log(`Hello ${nameToGreet}!`)

  const time = (new Date()).toTimeString()
  core.setOutput("time", time)

  const payload = JSON.stringify(github.context.payload, undefined, 2)
  console.log(`The event payload: ${payload}`)

} catch (error) {
  core.setFailed(error.message)
}
```

`getInput` uses the `who-to-greet` input defined in the action metadata file and initializes the value to a variable called `nameToGreet`. The variable is then logged to the console. `setOutput` uses the JavaScript [`Date()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) constructor to create a `Date` object and sets it as an output variable that actions running later in a job can use.

GitHub Actions provide context information that can be accessed with the `github` package. Our action prints the webhook event payload to the log. `github.context.payload` gets the JSON webhook payload for the event that triggered the workflow and logs it to the console. If an error is thrown in the above `index.js` example, `core.setFailed(error.message);` uses the actions toolkit [`@actions/core`](https://github.com/actions/toolkit/tree/main/packages/core) package to log a message and set a failing exit code.

## Commit and Tag JavaScript Action

GitHub downloads each action run in a workflow during runtime and executes it as a complete package of code before you can use workflow commands like `run` to interact with the runner machine. This means you must include any package dependencies required to run the JavaScript code. For this example that includes the toolkit `core` and `github` packages.

It's best practice to also [add a version tag](https://docs.github.com/en/actions/creating-actions/about-custom-actions#using-release-management-for-actions) for releases of your action.

```bash
git add .
git commit -m "JavaScript Action"
git tag -a -m "First JavaScript action release" v0.1
git push --follow-tags
```

### Test JavaScript Action with a Workflow

```bash
touch .github/workflows/javascript.yml
```

Update the `uses` line with your username and the name of your public repository.

```yaml
# .github/workflows/javascript.yml

on: [push]

jobs:
  hello_world_job:
    runs-on: ubuntu-latest
    name: A job to say hello to ajcwebdev
    steps:
      - name: Hello world action step
        id: hello
        uses: ajcwebdev/ajcwebdev-actions@v0.1
        with:
          who-to-greet: 'ajcwebdev'
      - name: Get the output time
        run: echo "The time was ${{ steps.hello.outputs.time }}"
```

When this workflow is triggered, the runner will download the `ajcwebdev-actions` action from your public repository and then execute it.