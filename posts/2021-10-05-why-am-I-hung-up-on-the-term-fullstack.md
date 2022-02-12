---
title: why am I hung up on the term fullstack?
description: what is the definition of fullstack?
date: 2021-10-05
tags:
  - fullstack
  - webdev
  - react
  - frontend
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3ugusvo9vnt7sxqzi2hh.png
layout: layouts/post.njk
---

>*Stacks evolve over time. But it’s not just what technologies they use, but what technology we even consider a part of a stack. What fullstack means morphs over time.*
>
>***Chris Coyier - [What Does it Mean to Be “Full Stack”?](https://css-tricks.com/what-does-it-mean-to-be-full-stack/)***

Today I got into a Twitter #BEEF about the appropriateness of the term "fullstack framework." Some of you may know that I am the host of a podcast called [Fullstack Jamstack](https://fsjam.org) and have made my name as a developer advocate for [RedwoodJS](https://redwoodjs.com), a self proclaimed "fullstack" JavaScript framework.

However, I do not make any claims to ownership of the term. My thoughts on the term and its definition are fiercely personal, and I welcome others to have their own passionate beliefs around it. I want to elicit dialogue around the term, not tell you what it means.

## Why is this personal for me?

Before joining the RedwoodJS team I was learning how to code at Lambda School, learning a "fullstack web development" curriculum. This curriculum contained roughly:
* 10% HTML/CSS
* 85% JavaScript/React
* 5% Node/Express/Postgres

I felt this was an uneven split between the frontend and backend. It didn't seem accurate to label this as a "fullstack" curriculum, instead it seemed more accurate to call it a "frontend" curriculum with a very small amount of backend material included at the end.

## What is the definition of "Full?"

I start [with the postulate](https://twitter.com/ajcwebdev/status/1445444206948876293) that the term "full" contains the most essential quality of "completeness." To say something is "full," is to say there is nothing left to add. This is why I would consider a stack without any database or storage to be incomplete.

How often do you use an application where everything you did with that application disappears after you take a break from using it? If you're just reading a blog post that's probably fine, not everything we do on the web requires persistence. But if you're writing that blog post, it's a different story.

## Is there actually just a frontend and backend?

A recent turn of phrase has come up called the ["front of the backend"](https://css-tricks.com/front-of-the-front-back-of-the-front/) to describe the illusive middle ground between the client side and the server side. Does this mean there is a "back of the frontend?"

There is the HTML and CSS for the content and styling and JavaScript (or some sort of JavaScript framework) for the client side interaction. But the data fetching logic and state management can be decoupled from the content and styling itself, forming its own layer of the application. This is the space Redux and React Query play in.

On the backend you have an operating system, server side language (or framework), and database for authentication, authorization, and persistence. But you can abstract a layer above that with an API mesh, serverless functions, or containers that let you ignore the actual operations of the underlying system.

## What else should be included in the definition of "fullstack?"

What about accessibility, internationalization, localization, security, deployment, DevOps, automated testing, design systems, data collection, analytics, emails, etc? How wide does the definition go? Is it even reasonable for us to expect a single developer to *ever* be fullstack?

Unfortunately I don't have the answer to that question. But as long as we are going to be selling the term for our bootcamps, our frameworks, and our podcasts, then we need to think carefully about what we mean when we use the term.

*Thank you to [Alex](https://twitter.com/ralex1993) for his thoughtful notes and pushback on this piece.*