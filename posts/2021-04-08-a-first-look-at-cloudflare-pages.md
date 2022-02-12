---
title: a first look at cloudflare pages
description: Cloudflare Pages is a Jamstack platform for frontend developers to collaborate and deploy websites.
date: 2021-04-08
tags:
  - cloudflare
  - react
  - jamstack
  - cdn
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3pjmiqf1pcvibr0bkm0a.jpeg
layout: layouts/post.njk
---

[Cloudflare Pages](https://pages.cloudflare.com/) is a Jamstack platform for frontend developers to collaborate and deploy websites. Simply connect to a GitHub repo, supply the necessary build commands and publish directories, and deploy your site to the world with just `git push`.

What's that? That sounds like they just created a carbon copy of Netlify on their own CDN and slapped a new name on it? You say that like it's a bad thing.

## Create a React app

The code for this article can be found at [ajcwebdev-cfpages](https://github.com/ajcwebdev/ajcwebdev-cfpages).

```bash
npx create-react-app ajcwebdev-cfpages
cd ajcwebdev-cfpages
```

Create a [blank repository](https://repo.new/) on GitHub with the same name as your React project.

```bash
git branch -M main
git remote add origin https://github.com/ajcwebdev/ajcwebdev-cfpages.git
git push -u origin main
```

Sign up for [Cloudflare Pages](https://pages.cloudflare.com/).

![01-cloudflare-pages-dashboard](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/a8v738niu7rzglxdhey3.png)

Click "Create a project."

![02-connect-git-repository](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/acazjqgeohulu708dh2l.png)

Select your React project and click the "Begin setup" button at the bottom.

![03-setup-build-and-deploy](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1vt3zcziu2383nag0wlm.png)

Your project name and production branch will be set automatically.

![04-blank-build-settings](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/62o3bczyjj7qrco8x1qi.png)

The build settings are blank, but you can select the Create React App framework preset for the build command and publish directory.

![05-create-react-app-framework-preset](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gn7nuygoqcbossy4bmzf.png)

Click "Save and deploy."

![06-initializing-build](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t1gvisf2yie9bopvr9hj.png)

Go take a nice long walk around the block, grab some coffee, take out your dry cleaning, file your taxes, complete that 10,000 piece puzzle you've been putting off, and then come back and your website build should be done.

![07-build-and-deployment-settings](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hztevixwe7gsyootr8w9.png)

Once the build finishes you will see the build and deployment settings at the bottom and a link to your site at the top.

![08-success-site-built-and-deployed](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yzvu77r1f15ydwt00snf.png)

Click the link to [ajcwebdev-cfpages.pages.dev](https://ajcwebdev-cfpages.pages.dev/) and you should see the following page.

![09-ajcwebdev-cfpages-deployed](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/thmui679mlk2se7japsb.png)