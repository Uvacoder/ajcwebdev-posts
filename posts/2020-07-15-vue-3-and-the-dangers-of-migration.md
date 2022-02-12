---
title: vue 3 and the dangers of migration
description: Vue 2 to Vue 3 has been a complicated migration path, which is why it took nearly 2 years to develop.
date: 2020-07-15
tags:
  - vue
  - javascript
  - composition
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/t3v0vxtef54rdr4wej94.png
layout: layouts/post.njk
---

At a conference talk in early 2019, Evan You listed the design goals for Vue 3 and claimed it would be faster, smaller, more maintainable, easier to target native, and will make the developers life easier. Sounds like a delicious free lunch, so what's the catch? Compatibility.

Vue 2 to Vue 3 has been a complicated migration path, which is why it took nearly 2 years to develop. Vue 3 was originally announced in Fall of 2018 and is now nearly production ready as we enter Q3 of 2020. The release candidate is coming out this month and the official release is planned for August. Evan You has been working with front end JavaScript frameworks for a long time and I’m sure the Angular 2.0 debacle of 2014 is still fresh in his mind.

A difficult migration path can damage an open source project almost beyond repair. It took Python a decade to fully migrate from 2 to 3, and the project was lucky enough to catch the machine learning wave throughout the late 2010s which brought a horde of new developers who had no concept of Python 2 (I was one of those new developers, I remember watching Python tutorials around 2017 and Python 2 was talked about like it was nuclear waste).

With a project like Vue you don’t just have to think about whether you will break code from a previous version. You need to think about whether you will break code from every single project that is built on top of Vue including Vue Router, Vuex, Vue CLI, vue-devtools, and vue-loader just to name a few. You can see the status of these different projects and how they are progressing at the vue-next github repo.

If you aren’t super plugged into the Vue ecosystem you may not have known that this massive version upgrade was happening, or that it caused a controversy in the Vue community in June 2019 from developers that are scared of change. That’s okay, none of that matters now. We’re almost there, so now is the time to start paying attention.

### Composition API

Evan has given numerous talks throughout 2019 and 2020 to explain the design principles and what to expect from Vue 3. The most significant addition is the Composition API, which React developers should instantly recognize as a Vue implementation of functional hooks. The Composition API was a primary catalyst for the outpouring of anger.

This looks to me like a case of the squeaky wheel getting the grease (translation: people who complain the loudest get the most attention even when they are a minority). Despite some difficulties with React hooks there is a fairly strong consensus from React developers that they are a valuable addition to the library, and class components are still around if you want to live in the past.

So will you be able to write Vue components like the good ol’ days in Vue 3 without breaking your code? It appears that you can. Originally it was announced that the old way of writing components would be deprecated. After the community response this was walked back. I have mixed feelings about this.

One of the biggest selling points of React hooks was that it would simplify state management and allow components to pass state around without prop drilling or having to bring in a 3rd party state management library. The different hooks work beautifully together as long as you are only using hooks. I find that having a project that mixes class components and functional components can cause problems when you try to pass state from one to other. I’m sure there are plenty of work arounds to get the two to play nice together, but it certainly isn’t trivial and can slow down a project.

If class components had been deprecated in React and everyone was forced to write functional components, React projects would be much more cohesive and components would be much easier to share between different projects. As it stands today there are many projects with this strange mix of class components and functional components and you really need to dig deep into the React internals to understand what’s happening under the hood.

In JavaScript there has always been a tension between the object oriented paradigm and the functional paradigm. This is further complicated by the fact that JavaScript was originally built with prototypical object orientation instead of class object oriented. Classes were added to the language in ES2015 but they don’t behave like classes from other languages, so really we are dealing with two and a half programming paradigms. This same tension is now playing out in the different frameworks, and it's not pretty. I wonder if Svelte will be able to avoid a similar fate.