---
title: bison - fullstack jamstack in-a-box
description: In case you weren't already confused enough about the differences between Redwood and Blitz we now have a third contender, Bison.
date: 2020-08-14
tags:
  - bison
  - prisma
  - nextjs
  - typescript
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hb95emki1qp2wfxb1jhp.png
layout: layouts/post.njk
---

In case you weren't already confused enough about the differences between [Redwood and Blitz](https://blog.logrocket.com/blitz-vs-redwood/) we now have [a third contender](https://www.youtube.com/watch?v=tEInOneENuo), Bison, with a very similar overall architecture but its own particular combination of technologies. [Chris Ball](https://community.redwoodjs.com/u/cball) appears to be the brains behind the project. It is supported by Echobind where he works as CTO.

Bison knows that while good artists borrow, great artists steal, and they have taken direct inspiration from Redwood with their [own version of Cells](https://github.com/echobind/bisonapp#Alternatives). But they also have their own vision of what they want the framework to be and where they want it to go:

>*Think of Bison as a bit closer to the metal, but preconfigured for maximum DX and efficiency. The good news is, if you disagree with any of the choices we've made, nothing is hidden from you and you're welcome to adapt the "framework" to fit your needs.*

In tandem with the public announcement of the project they also released a short [overview video](https://vimeo.com/447643624) and a long [deep dive](https://vimeo.com/447586198) of the entire architecture. I really love this approach and I hope other new frameworks on the horizon consider it.

Here's a break down of each stack and how they compare:

||Redwood|Blitz|Bison|
|-|------------|-------|------|
|**Routing**|Redwood Router|Next.js|Next.js|
|**Types**|TypeScript|TypeScript|TypeScript|
|**Not-an-ORM**|Prisma|Prisma|Prisma|
|**Code Generation**|Redwood CLI|Blitz CLI|GraphQL Codegen, Hygen Templates|
|**Styling**|Tailwind|Import CSS|Chakra UI|
|**Forms**|React Hook Form|Vanilla JS/TS|React Hook Form|
|**Testing**|Jest|Jest, RTL|Jest, RTL, Cypress|
|**Deploy**|Netlify|Vercel|Vercel|

All three frameworks seem to have their own authentication which would probably need a much longer post than this to directly compare. All three have their own unique opinions about project and file structure that would also make an interesting post by itself. All three support SQLite/Postgres out of the box with the option to switch in other databases of your choosing.