---
title: a first look at redwoodJS part 8 - deploy to the world
description: Deploy our frontend to Netlify
date: 2020-08-09
tags:
  - redwoodjs
  - react
  - netlify
  - postgres
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/7x38o4nssnlf0d9hdo5h.png
layout: layouts/post.njk
---

>*RedwoodJS is an attempt to build a fullstack framework for JavaScript and really deploy it in a serverless way. That’s one of the primary tenants that we have.*
>
>*Build it end to end with JavaScript. Deploy it to a serverless environment to give you the advantages of the scale that can bring, as well as the global distribution that can bring.*
>
>***Tom Preston-Werner - [Full Stack Radio](https://www.fullstackradio.com/episodes/138)***

# Part 8 - Deployment

You made it to the last part!

![00-rob-wooooo](https://dev-to-uploads.s3.amazonaws.com/i/jewl54ynr64v8hkfbugb.gif)

In this part we will deploy our frontend application with Serverless functions on Netlify and connect it to our backend Postgres database hosted on Railway.

## 8.1 GitHub Repo

First you will need a GitHub repo with your Redwood project. Mine is [here](https://github.com/ajcwebdev/ajcwebdev-redwood/). It contains branches that match up with the state of the project at the end of each of the first six parts. The default branch is part7 and will be the branch we deploy.

![01-github-repo](https://dev-to-uploads.s3.amazonaws.com/i/ttr1wngw4q6j0ztrpmd7.jpg)

## 8.2 Netlify

```bash
yarn rw setup deploy netlify
```

This creates a file at `/netlify.toml` containing the commands and file paths that Netlify needs to know about to build a Redwood app.

```toml
[build]
  command = "yarn rw deploy netlify"
  publish = "web/dist"
  functions = "api/dist/functions"

[dev]
  command = "yarn rw dev"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

Next you'll need an account on [Netlify](http://netlify.com/).

![02-netlify-sites](https://dev-to-uploads.s3.amazonaws.com/i/0ae0i5yvfydgmlq2yg3g.jpg)

Click `New site from Git` to create a new site from git.

![03-new-site-from-git](https://dev-to-uploads.s3.amazonaws.com/i/k5va79ftlji8n8pgyqn3.jpg)

You can also use GitLab or Bitbucket if you're a hipster. 

![04-create-a-new-site](https://dev-to-uploads.s3.amazonaws.com/i/hke4rkg7c25o0ylhqhu6.jpg)

Enter the name of your repo into the search bar.

![05-ajcwebdev-repo](https://dev-to-uploads.s3.amazonaws.com/i/gj0unavsfx7as0jh1iq2.jpg)

Select the repo.

![06-ajcwebdev/ajcwebdev](https://dev-to-uploads.s3.amazonaws.com/i/98n6n2564hlizskaqxwu.jpg)

It selects the default branch to deploy.

![07-branch-to-deploy](https://dev-to-uploads.s3.amazonaws.com/i/3idt3g6rvzspzvpx7z9g.jpg)

The build command may be entered by default.

![08-build-command](https://dev-to-uploads.s3.amazonaws.com/i/it7b79zxdyozz9slpqfe.jpg)

If the build command is blank enter the following (the screenshot is a lie):

```bash
yarn rw deploy netlify
```

Click `Deploy site` to deploy the site.

![09-site-deploy-in-progress](https://dev-to-uploads.s3.amazonaws.com/i/up5prlddtk9iiarv40ij.jpg)

If we go to the `Deploys` section we can see more information about the build.

![10-netlify-deploys](https://dev-to-uploads.s3.amazonaws.com/i/51r5znou958rtvrnglck.jpg)

Your build should take at least a minute or more, so don't freak out if it doesn't work immediately.

![11-deploy-published](https://dev-to-uploads.s3.amazonaws.com/i/6ovc5rl1dgistuphh93w.jpg)

Our deploy took 2 minutes and 15 seconds and we can also see a summary of the deploy.

![12-deploy-summary](https://dev-to-uploads.s3.amazonaws.com/i/tqf4gcnce5fr5vbw0j0x.jpg)

We haven't really deployed the site though, because right now we just have the frontend deployed to Netlify. But we haven't done anything with our database so we should expect an error:

![13-error-unexpected-token](https://dev-to-uploads.s3.amazonaws.com/i/oqil2vvnzn2nffpf9ljw.jpg)

## 8.3 Config/Environment Variables

![14-copy-postgres-vars](https://dev-to-uploads.s3.amazonaws.com/i/tdbwdkrtuhovjv7j71qr.jpg)

Select `Deploy settings` to go to your deploy settings.

![15-netlify-deploys](https://dev-to-uploads.s3.amazonaws.com/i/l7314ljzh8uf6u5vgt0o.jpg)

Under `Build & deploy` select `Environment`.

![16-environment-variables](https://dev-to-uploads.s3.amazonaws.com/i/oencxni0wlxc7fzjj7pz.jpg)

Click the `Edit variables` button to edit the variables.

![17-edit-variables](https://dev-to-uploads.s3.amazonaws.com/i/q0a0k2bapyqhzq6phdbq.jpg)

We're going to use the key/value pair from our Heroku Postgres app.

![18-set-environment-variables](https://dev-to-uploads.s3.amazonaws.com/i/laxg642p28rty060zpjb.jpg)

First enter `DATABASE_URL` for the key.

![19-DATABASE_URL](https://dev-to-uploads.s3.amazonaws.com/i/4a1q892c8p33dd4384n1.jpg)

Then paste the value.

![20-paste-postgres-vars](https://dev-to-uploads.s3.amazonaws.com/i/6rp62306rnvvhm9m0glw.jpg)

At the end of the value add `?connection_limit=1`. This makes sure that each AWS Lambda only opens one database connection.

![21-add-connection-limit](https://dev-to-uploads.s3.amazonaws.com/i/122c8k9jgxymmjas0wjr.jpg)

If we go back to our code in `schema.prisma` we can see that we set our datasource to the environment variable `DATABASE_URL` and our client to `native`.

```
datasource DS {
  provider = "postgres"
  url      = env("DATABASE_URL")
}

generator client {
  provider      = "prisma-client-js"
  binaryTargets = "native"
}
```

And then Prisma looks up our local environment file. We override these once you deploy to Netlify.

```
# schema.prisma defaults
DATABASE_URL=file:./dev.db

# disables Prisma CLI update notifier
PRISMA_HIDE_UPDATE_MESSAGE=true
```

Click the `Trigger deploy` button to trigger a deploy and select `Deploy site` to deploy the site. 

![22-trigger-deploy](https://dev-to-uploads.s3.amazonaws.com/i/zytfidzhpakovjtb3c1p.jpg)

You will now receive a great series of logs.

![23-build-logs-1](https://dev-to-uploads.s3.amazonaws.com/i/z40cboxq13wp1mrp3fn1.jpg)

The logs will detail the build process.

![24-build-logs-2](https://dev-to-uploads.s3.amazonaws.com/i/8yy4l7k14bufb3u4s7ig.jpg)

Do not concern yourself with the logs.

![25-build-logs-3](https://dev-to-uploads.s3.amazonaws.com/i/eootzwb58yjda38pbike.jpg)

Let the logs wash over you and through you.

![26-build-logs-4](https://dev-to-uploads.s3.amazonaws.com/i/prcyns2i2i2kbvmtu5wf.jpg)

Raft is a [bunch of logs that get you off Paxos island](https://www.youtube.com/watch?v=w9GP7MNbaRc&t=31m47s).

![27-build-logs-5](https://dev-to-uploads.s3.amazonaws.com/i/la1l401gkdl7hjmq3h0l.jpg)

Now let's go back to our site.

![28-website-empty](https://dev-to-uploads.s3.amazonaws.com/i/q1l566zivlxxzhm17k4k.jpg)

Lets create a new post.

![29-posts](https://dev-to-uploads.s3.amazonaws.com/i/zyj83rse42utpkes6fdp.jpg)

Click the `NEW POST` button to make a new post. Enter a title and body.

![30-new-post](https://dev-to-uploads.s3.amazonaws.com/i/mvvxj6fakcacjpfxni42.jpg)

Save the new post.

![31-first-post-created](https://dev-to-uploads.s3.amazonaws.com/i/39jf6jl3atvs4bkih0gq.jpg)

Lets try to edit our new post.

![32-first-post-edit](https://dev-to-uploads.s3.amazonaws.com/i/gnrv1fxfx8f75gcg5c4o.jpg)

Save your edit to the post.

![33-first-post-edit-working](https://dev-to-uploads.s3.amazonaws.com/i/gumgl08iaq5b3ndae1rk.jpg)

It looks like it's working. Lets check the front page to make sure it's really working.

![34-front-page-working](https://dev-to-uploads.s3.amazonaws.com/i/ebsgis2sngfwh7qoon6p.jpg)

For our final step, we will give our site a custom domain. We can do this in our Settings on Netlify. Go to Domain management and you should see a box for Custom domains.

![02-custom-domains](https://dev-to-uploads.s3.amazonaws.com/i/7p33rzro0uvqp1vdxyyq.jpg)

Netlify assigns a random domain name by default but gives the option to edit it.

![03-change-site-name](https://dev-to-uploads.s3.amazonaws.com/i/pmiu3qoa77l36epvxnxp.jpg)

I'll change my site name to `ajcwebdev-redwood`.

![04-ajcwebdev-redwood-netlify-app](https://dev-to-uploads.s3.amazonaws.com/i/lyhldf9h1ugjwave5vx9.jpg)

Click Save and it'll reflect your new site name.

![05-default-subdomain](https://dev-to-uploads.s3.amazonaws.com/i/hdljrrgq5g7yre3a8wbd.jpg)

And that's it! Right now you should either be feeling a great sense of accomplishment over building some amazing, or a horrible sinking feeling that you just wasted hours of your life building something useless. The choice is yours!