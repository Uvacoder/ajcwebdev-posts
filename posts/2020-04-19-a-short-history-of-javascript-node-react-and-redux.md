---
title: a short history of javascript, node, react, and redux
description: JavaScript is multi-paradigm, dynamically typed programming language that supports first-class functions and prototypical object-orientation.
date: 2020-04-19
tags:
  - javascript
  - node
  - react
  - redux
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/hmvci85bp2ekk219qr0q.png
layout: layouts/post.njk
---

## JavaScript

JavaScript is multi-paradigm, dynamically typed programming language that supports first-class functions and prototypical object-orientation. Along with HTML and CSS it is the underlying technology of the browser and the world wide web.

### LiveScript

Before JavaScript, web pages were static and lacked the capability for dynamic behavior after the page was loaded in the browser. In 1995, Netscape decided to add a scripting language to Navigator, the successor to the popular Mosaic browser.

To hedge their bets they pursued two routes: collaborating with Sun Microsystems to embed Java and hiring Brendan Eich to embed Scheme. After hiring Eich, Netscape decided that the best option was to devise a new language with syntax similar to Java to capitalize on its popularity.

The new language and its interpreter implementation were officially called LiveScript when first shipped as part of a Navigator release in September 1995. But the name was changed to JavaScript three months later.

The standard implementation of JavaScript today is known as ECMAScript due to the ongoing copyright disputes over the name. JavaScript has APIs for working with text, dates, regular expressions, data structures, and the Document Object Model (DOM).

### DOM

The Document Object Model is an object representation of an html document that serves as a programming interface to select and manipulate the page. The DOM can be used to change document structure, content, and styling. It creates and propagates event objects with information about event type and target.

[![Image](https://sedaily-topics.s3.amazonaws.com/topic_images/0_11140431928791017)](https://data-flair.training/blogs/javascript-dom/)

The object model is a tree structure with each DOM element in a tree node. When a web page is loaded, the browser first looks for the HTML file. The browser uses the HTML and CSS files as a blueprint to build the page. The browser parses these instructions and builds a model for how the page should look and act using Javascript.

### Events

Every user interaction with a site is an event: a click, moving the mouse, scrolling the page, pressing a key on the keyboard, etc. JavaScript allows us to add features and make modifications to our site by directly reacting to user interactions such as a button click, drag and drop, or zoom.

## JavaScript Everywhere

In the early days of web development, many programmers using PHP, Perl, and Ruby looked down on JavaScript as a toy language. But as websites became more interactive, JavaScript started to become the elephant in the room for every web developer because JavaScript was the only language that ran in the browser.

If a developer wanted to provide a high degree of client side interaction their only choice was to implement it in JavaScript. A commonly held sentiment among some developers was that this was an unfortunate inconvenience of web development. Whenever possible code that could be written on the back end should be written on the back end.

But as web sites grew increasingly interactive, developers found it increasingly difficult, and illogical, to avoid writing JavaScript. Users wanted rich client side interactions. Instead of continuing to swim against the current some developers began to embrace this inevitability.

A new generation of technologies emerged that aimed to code their entire stack in JavaScript. In an attempt to create the worst marketing buzzword possible it was called isomorphic JavaScript. Others more sensibly called it "JavaScript everywhere," or as I like to say, "hella JavaScript."

### MEAN Stack

Ryan Dahl created NodeJS in 2009 because he was frustrated by Apache Server's inability to scale concurrent connections into the hundreds of thousands. He augmented Google's V8 Javascript engine with an event loop and input/output functionality.

That same year, AngularJS was created by Miško Hevery as the underlying framework behind an online JSON storage service.

MongoDB was also created around the same time as an internal component of 10gen's planned PaaS product. As the database started to gain traction it eventually became the sole focus of the company and. In 2013, they rebranded to Mongo Inc. MongoDB also leveraged JSON by providing a document schema instead of the dominant relational model.

The final piece came in 2010 when TJ Holowaychuk created a Sinatra inspired server framework for Node called Express.js that handled routing and middleware.

One of the first attempts to build a full stack solution with only JavaScript arrived in 2012 with Meteor.js, a framework that used Node and MongoDB. The next year Valeri Karpov coined a new term in an article published on MongoDB's blog, [The MEAN Stack: MongoDB, ExpressJS, AngularJS and Node.js](https://www.mongodb.com/blog/post/the-mean-stack-mongodb-expressjs-angularjs-and).

### Jamstack

The MEAN stack proved impractical for many developers due to the prohibitively large bundle size of Angular, the sprawling dependencies of Node, and the lack of ACID transactions in MongoDB.

[![Image](https://sedaily-topics.s3.amazonaws.com/topic_images/0_9599177836188364)](https://medium.com/memory-leak/the-jamstack-its-pretty-sweet-e0834e4e6bb7)

The Jamstack is a radical departure that attempts to serve static files from globally distributed CDN's, removing the server and the database from the stack entirely.

GraphQL API's are used as a glue layer for message passing between the CDN, 3rd party plugins, and users of your app. Lastly, markup can be used for creating websites, documents, notes, books, presentations, email messages, and technical documentation.

## Node

Node.js is a JavaScript runtime environment for executing JavaScript code on a server instead of a web browser. It was created in 2009 by Ryan Dahl out of his frustration with Apache Server's inability to scale concurrent connections into the hundreds of thousands. Many attempts at putting JavaScript on the server had been attempted, starting in the mid-90s with Netscape's LiveWire Pro Web.

Node's power and success comes from two key features, it is *event driven* with *asynchronous input-output*:
* **Event Driven**: A programming paradigm in which the flow of the program is determined by events such as user actions (mouse clicks, key presses), sensor outputs, or messages from other programs or threads.
* **Asynchronous I/O**: A form of input/output processing that permits other processing to continue before the transmission has finished.

[![Image](https://sedaily-topics.s3.amazonaws.com/topic_images/0_4129993339286535.jpg)](https://twitter.com/imneutralchaos/status/494959181871316992)

>*The thesis is that IO has to be done differently. We're doing it wrong... everything... the way that we're thinking about doing IO really makes things difficult. Writing servers and writing any soft of application is difficult because of how we're doing IO.*
>
>***Ryan Dahl - [Node.js (November 8, 2009)](https://www.youtube.com/watch?v=ztspvPYybIY)***

### V8

Node builds on top of Google's V8 Javascript engine with the addition of an event loop and non-blocking IO. V8 compiles JavaScript directly to native machine code using just-in-time compilation. It's important to emphasize that Node and V8 are not written in JavaScript, they are runtime environments written in C/C++ that execute JavaScript code.

![Image](https://sedaily-topics.s3.amazonaws.com/topic_images/0_45149077309280927)

### Socket.io

A popular early use case for Node was to build websocket servers like a chat server. Lots of browser connect to the server and messages are sent back and forth between the client and the server while the socket stays open. In 2010 Guillermo Rauch built socket.io, a framework for this specific use case.

![Image](https://sedaily-topics.s3.amazonaws.com/topic_images/0_27532092743435954)

### Joyent

After giving the inaugural presentation for Node, Ryan Dahl was approached by Mark Mayo from Joyent. They were also working on server side JavaScript and wanted to hire Ryan Dahl to [build out Node while working as a Joyent employee](https://www.youtube.com/watch?v=L_JKb61EalQ). Bryan Cantrill described Node as being to [Joyent what Java was to Sun](http://dtrace.org/blogs/bmc/2010/07/30/hello-joyent/). For some reason he seemed to think this was a positive comparison.

![Image](https://sedaily-topics.s3.amazonaws.com/topic_images/0_1017477370946176)

In January 2012, Dahl believed that the Node project was "done," and stepped aside. He promoted Isaac Schlueter to manage the project and also sold the intellectual property of Node to Joyent.

After two years Schlueter believed the greatest opportunity for Node was in the growing ecosystem of packages and modules. He passed the Node project along to Timothy J. Fontaine so he could focus on npm inc. Unlike Dahl, Schlueter maintained legal ownership of his intellectual property.

### npm

Node's rise to prominence was helped by its tight integration with npm, a package manager created by Isaac Schlueter that made it incredibly easy for programmers to publish modules. This allowed easy code sharing and enabled a Cambrian explosion of JavaScript programs.

>*I think node needs a package manager.  There are a lot of very useful modules out there, but it's tricky right now to actually use more than one of them together. Here's a proposal for a very lightweight and simple way to alleviate the situation.  I'm calling it npm, and it should be able to install itself fairly soon. :)*
>
>***Isaac Schlueter - [Preview: npm, the node package manager (September 30, 2009)](https://groups.google.com/forum/?hl=en#!topic/nodejs/erDWyS4xPw8)***

Npm was owned and maintained by a private company, npm inc, for most of Node's lifetime. This lead to friction with the open source community and in 2019 former CTO of npm inc, C J Silverio, [announced a competing package manager](https://www.youtube.com/watch?v=MO8hZlgK5zc) to address concerns about centralized ownership of the package registry.

>*Success is a catastrophe for a lot of projects. It's a catastrophe that you need to survive and success for npm was a catastrophe, here's why. Npm's package registry is centralized.*
>
>*It's not just a CLI tool that grabs the code and puts it on your hard drive. The CLI is probably the least important part of the npm machinery despite how frequently you interact with it. Npm is most importantly a centralized package registry and repository.*
>
>***CJ Silverio - [The Economics of Open Source (June 3, 2019)](https://www.youtube.com/watch?v=MO8hZlgK5zc)***

![Image](https://sedaily-topics.s3.amazonaws.com/topic_images/0_16581183117663367)

Npm inc was acquired by GitHub in March 2020 (GitHub itself was acquired by Microsoft in 2018). The jury is still out whether this is better or worse.

### io.js
 
On Thanksgiving Day in 2014, Fedor Indutny started io.js, a fork of Node.js. Other members of the team described it as a "table flipping moment" for Fedor, who was frustrated by the length of time required for Joyent to approve his pull requests. Even though the fork was sparked by a single individual over a seemingly singular issue, it became a rallying cry for many in the community who saw larger systemic issues.

Node was not staying up-to-date with the latest releases of the Google V8 JavaScript engine and lacked support for new features in ES2015. The community was dissatisfied with Joyent's stewardship of the project and io.js was created as an open governance alternative with a separate technical committee.

![Image](https://sedaily-topics.s3.amazonaws.com/topic_images/0_24258206719389475)

In February 2015, the intent to form a neutral Node.js Foundation was announced. By June 2015, the Node.js and io.js communities voted to work together under the Node.js Foundation. In September 2015, Node.js v0.12 and io.js v3.3 were merged back together into Node v4.0. This merge brought V8 ES6 features into Node.js and a long-term support release cycle.

>*The Node.js and io.js developer communities today are announcing a collaboration to merge their respective code bases and continue their work in a neutral forum, the Node.js Foundation, hosted by The Linux Foundation.*
>
>***Linux Foundation - [Node.js Foundation Advances Community Collaboration, Announces New Members and Ratified Technical Governance (June 15, 2015](https://www.linuxfoundation.org/press-release/2015/06/node-js-foundation-advances-community-collaboration-announces-new-members-and-ratified-technical-governance/))***

### Node Today

In a JS Party interview on April 2, 2020, Guillermo Rauch lamented the failure of Node to go further as an industry trend, believing that it would have been much bigger and have more enterprise success. Even the creator of Node believes that we need to move beyond it.

Dahl gave a talk called "10 Things I Regret About Node.js" at JS Conf in 2018 which also annouced a Node competitor called Deno which aims to address Node's shortcomings in security, project building, and modules. Deno 1.0 was released on May 13, 2020. Despite these criticisms, Node remains the most popular choice for bootcamps and online tutorials focusing on fullstack projects.

The advantages of building your front end and your back end in the same language has proved to be a force multiplier, especially for new programmers trained only in JavaScript. Companies deploying Node today include Netflix, PayPal, Trello, Capital One, LinkedIn, Yahoo, Mozilla, Uber, Groupon, Ebay, and Walmart.

## React

React is a JavaScript library for building user interfaces. Jordan Walke created React in 2011 while working on internal tools for the Facebook Ads platform. It was first publicly deployed on Facebook's newsfeed. Pete Hunt learned of the framework in 2012 and began architecting Instagram as a single page web app built entirely with React and Backbone.Router.

React was open sourced at JSConf in May 2013. Early adopters like Paul Seiffert and Clay Allsopp used it as a replacement for Backbone's view layer. The team began pitching React as the V in MVC, or the view layer of the model-view-controller pattern.

### JSX: JavaScript Syntax Extension

After being open sourced the majority of the attention and controversy was directed at an auxillarly part of the library, JSX. JSX is a language extension for JavaScript based on a similar extension for PHP that is internally popular at Facebook.

>*XHP is a PHP extension which augments the syntax of the language to both make your front-end code easier to understand and help you avoid cross-site scripting attacks. XHP does this by making PHP understand XML document fragments, similar to what E4X does for ECMAScript (JavaScript).*
>
>***Marcel Laverdet, [XHP: A New Way to Write PHP (February 9, 2010)](https://www.facebook.com/notes/facebook-engineering/xhp-a-new-way-to-write-php/294003943919/)***

ECMAScript for XML (E4X) was a programming language extension that added native XML support to ECMAScript, which at the time included ActionScript, JavaScript, and JScript. It aimed to provide an alternative to the standard DOM interface with a simpler syntax for accessing XML documents.

![JSX](https://sedaily-topics.s3.amazonaws.com/topic_images/0_8967121357493488)

This was controversial because mixing program logic with presentation code was considered a violation of separation of concerns. For example, the Handlebar's documentation included the following quote:

>*Handlebars.js and Mustache are both logicless templating languages that keep the view and the code separated like we all know they should be.*

### Composable Components

Members of the core team frequently emphasized that the library did not depend on JSX. On June 5th, Pete Hunt published a blog post to explain the philosophical justification behind React called [Why did we build React?](https://reactjs.org/blog/2013/06/05/why-react.html) He emphasized the composable nature of React components.

>*React is a library for building composable user interfaces. It encourages the creation of reusable UI components which present data that changes over time. Traditionally, web application UIs are built using templates or HTML directives. These templates dictate the full set of abstractions that you are allowed to use to build your UI. React approaches building user interfaces differently by breaking them into components.*
>
>***Pete Hunt - [Why did we build React? (June 5, 2013)](https://reactjs.org/blog/2013/06/05/why-react.html)***

![Function component](https://sedaily-topics.s3.amazonaws.com/topic_images/0_3784954379404779)

### Flux: One-way data binding

Another key architectual decision of React was its emphasis on one-way data binding instead of two-way binding used in frameworks like AngularJS and Knockout.

>*React was born out of frustrations with the common pattern of writing two-way data bindings in complex MVC apps. React is an implementation of one-way data bindings. "One-way data binding" foregoes the wiring of model properties to DOM manipulation for something which sounds like a really bad idea: Every time anything on our model changes, throw away the entire UI and re-render it from scratch.*
>
>***Lee Byron - [Quora Answer (June 7, 2013)](https://www.quora.com/How-is-Facebooks-React-JavaScript-library-How-does-it-compare-with-other-popular-JavaScript-libraries/answer/Lee-Byron)***

To fully achieve this React needed something to approximate the model in an MVC architecture. This lead to the creation of Flux and a reimaging of the entire MVC architecture.

>*Flux works well for us because the single directional data flow makes it easy to understand and modify an application as it becomes more complicated. We found that two-way data bindings lead to cascading updates, where changing one data model led to another data model updating, making it very difficult to predict what would change as the result of a single user interaction.*
>
>***Jing Chen, Bill Fisher - [Flux: An Application Architecture for React (May 6, 2014)](https://reactjs.org/blog/2014/05/06/flux.html)***

Aside from blog posts and presentations explaining Flux, Facebook did not actually open source the code for their Flux implementation. This lead to many different libraries being created with wide spread confusion among developers over the different trade offs these libraries contained. Explanations of the libraries involved complex flow diagrams like this:

![Fluxxor](https://sedaily-topics.s3.amazonaws.com/topic_images/0_841985202629173)

The community eventually gravitated around Redux, an implementation Dan Abramov created for his presentation about hot loading. It contained a mostly linear flow that resembled the Elm architecture. Redux was the premiere state management solution for many years although today the community is starting to explore alternative state management solutions.

![Redux](https://sedaily-topics.s3.amazonaws.com/topic_images/0_8331088438870871)

### React Router, Relay, React Native: World Domination

The React ecosystem expanded dramatically throughout 2015 as the community built out sophisticated solutions for routing, data fetching, and mobile. The core remained a lightweight library, but the different pieces of the ecosystem started to resemble a larger full featured framework like Ember when combined. React is now the dominant frontend framework for JavaScript and is deployed by companies such as Airbnb, Uber, Netflix, Pinterest, and Twitter.

## Redux

>*Flux and React are both all about keeping a simple mental model.*
>
>***Jing Chen (July 25, 2014)***

Flux is an application architecture that Facebook uses for building client-side web applications. It complements React's composable view components by utilizing a unidirectional data flow. It is not a framework or a library, but a design pattern inspired by CQRS.

It was first debuted at F8 in May 2014 by Jing Chen, Pete Hunt, and Tom Occhino. Jing Chen started her presentation by describing issues they'd encountered while scaling an MVC application.

>*MVC works pretty well for small applications. Everything has its own particular role to play. The problem is that it doesn't make room for new features. Let's look at what happens when we add a lot of models and we add a lot of views to the system. There's just an explosion of arrows.*
>
>***Jing Chen - [Hacker Way: Rethinking Web App Development at Facebook (May 4, 2014)](https://www.youtube.com/watch?v=nYkdrAPrdcw)***

![Image](https://sedaily-topics.s3.amazonaws.com/topic_images/0_8200405328446236)

She described a recurring bug in the Facebook Chat system. Users would frequently see a red number over their chat icon signifying an unread message, but when they clicked the icon there would not be any new messages. Facebook's engineers would think that they fixed the bug but it would continually reappear due to the fragility of the coupled architecture.

Some engineers referred to this as the "Banana vs. Jungle" problem: you ask for a banana but instead you get back a banana, a gorilla holding the banana, and a jungle containing the gorilla. The Facebook engineers had discovered the need for command-query separation.

>*We found that two-way data bindings lead to cascading updates, where changing one data model led to another data model updating, making it very difficult to predict what would change as the result of a single user interaction.*
>
>***Bill Fisher, Jing Chen - [Flux: An Application Architecture for React (May 6, 2014)](https://reactjs.org/blog/2014/05/06/flux.html)***

### Command Query Responsibility Segregation

Command–query separation is a principle stating that every method should either be a *command* that performs an action, or a *query* that returns data to the caller, but not both. In other words, asking a question should not change the answer.

Command query responsibility segregation (CQRS) applies the CQS principle by using separate Query and Command objects to retrieve and modify data, respectively. CQRS fits well with event-based programming models, see [Javascript Topic Page](https://www.softwaredaily.com/topic/javascript) for a description of how JavaScript handles events in the browser.

## Dispatcher, Store, Views

Flux eschews MVC in favor of a unidirectional data flow as described on the [React Topic Page](https://www.softwaredaily.com/topic/reactjs). When interacting with a **view (React component)** an action is propagated through a central **dispatcher** to **stores** that hold the application's data and business logic. The stores then update all affected views.

The stores accept updates and reconcile them as appropriate, rather than depending on something external to update its data in a consistent way. Nothing outside the store has insight into how it manages data for its domain and there are no direct setter methods.

The flux documentation suggests the following diagram should be the primary mental model for Flux. The dispatcher, stores and views are independent nodes with distinct inputs and outputs. Actions are simple objects containing new data and an identifying type property:

![Unidirectional data flow diagram (Action -> Dispatcher -> Store -> View)](https://facebook.github.io/flux/img/overview/flux-simple-f8-diagram-1300w.png)
*Data in a Flux application flows in a single direction -* ***[Flux Documentation](https://facebook.github.io/flux/docs/in-depth-overview/)***

The views may cause a new action to be propagated through the system in response to user interactions:

![Flux diagram of client action](https://sedaily-topics.s3.amazonaws.com/topic_images/0_1861436422799183)

Redux is a predictable state container for JavaScript apps. It aims to help applications behave consistently and run in different environments (client, server, and native). While Redux was originally created to be used with React it can also be integrated with any other view library.

### Reducers, Actions, Store

Reducers are pure functions that take in the state and an action as parameters. The action describes how the state will change. The store is a global variable that holds all of your applications state. The store is known as the single source of truth because it is a global variable that holds all the state in the app.

Redux was created by Dan Abramov for a presentation he gave about hot loading. Redux was a secondary concern for him, but his succinct explanation led to many adopting his version of the Flux architecture.

>*Stores are stateful. If you re-execute the store code it's going to get the new initial state. The state is lost, the subscriptions are lost, it's a bummer. But it doesn't have to be this way.*
>
>*As I was thinking more about trying to work around this and to fit Flux into React hot loader workflow I figured out that store in Flux does too many things. It has too much boilerplate. And I'm not talking about the switch statements. The real boilerplate is in concepts, not in syntax.*
>
>***Dan Abramov - [Live React: Hot Reloading with Time Travel (July 5, 2015)](https://www.youtube.com/watch?v=xsSnOQynTHs)***

Since then Redux has been the dominant state management solution for React application. But in a series of SEDaily interviews with React luminaries, many expressed a need to move beyond Redux. There is a large ecosystem of third party libraries for handling state in React including, MobX, Kea, Overmind, and Easy Peasy.

Redux has also inspired similar approaches outside of the React ecosystem like Vuex for Vue and ngrx for Angular. Within the React library the Context API and functional hooks have started to be used as a substitute for Redux. Facebook is also working on a new, experimental state library called Recoil.

>*RecoilJS tries to make our life easier by providing a Reactish API for even more flexible state management across complex applications. It defines a graph attached to our React components tree so that state changes flow from atoms which are the roots of this graph to our components through selectors which are pure functions.*
>
>***Marios Fakiolas - RecoilJS is meant to rock your React world (May 24, 2020)***

![Image](https://sedaily-topics.s3.amazonaws.com/topic_images/0_995610565814496.jpg)

There is a very, very large amount of legacy React projects built with Redux. There will continue to be many projects built with Redux, but there will also be many projects that will explore new state implementations. If you are a React developer you should start thinking about state management now, before you find yourself drowning in action creators and reducer functions.

![Image](https://sedaily-topics.s3.amazonaws.com/topic_images/0_4744775498652225.jpg)