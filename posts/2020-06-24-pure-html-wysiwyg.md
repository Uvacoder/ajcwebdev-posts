---
title: pure html wysiwyg
description: Making content editable, part of the HTML developer guide, gives an example of a simple but complete rich text editor.
date: 2020-06-24
tags:
  - html
  - richtext
  - editor
  - wysiwyg
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/85vdc4xu4a8tmcyo3mhm.jpg
layout: layouts/post.njk
---

I stumbled upon the most incredible thing the other day on MDN. On the page for [Making content editable](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Editable_content) which is part of the [HTML developer guide](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Editable_content) they give an example of a simple but complete rich text editor.

![00](https://dev-to-uploads.s3.amazonaws.com/i/kna0891ybslqbmgsmhwy.jpg)

I was intrigued, so I copy-pasted the whole block of code into a blank document in TextEdit and saved it as `index.html` and opened it up.

![01](https://dev-to-uploads.s3.amazonaws.com/i/awn0arv0q16liuo4541o.jpg)

I clicked and entered some text.

![02](https://dev-to-uploads.s3.amazonaws.com/i/c7pi2jnmymi8c5w2uxrl.jpg)

I entered some more text.

![03](https://dev-to-uploads.s3.amazonaws.com/i/z2del75qo2xwdq67q8cx.jpg)

I clicked the mysteriously named "Show HTML" checkbox wondering what it could possibly do.

![04](https://dev-to-uploads.s3.amazonaws.com/i/wszp96jzmju45nwln7g1.jpg)

I don't know about you, but this absolutely BLEW MY MIND. I started digging into the code.

# pure html

First of all the title of this article is actually a bald faced lie so I apologize for that. This is definitely not pure html and requires a decent amount of JavaScript and CSS to work.

![05](https://dev-to-uploads.s3.amazonaws.com/i/6jak0hskmnqcm6o1p57e.jpg)

I titled it "pure html" because it keeps everything in a single index.html file that cleanly separates the script, styling, and html body. Vue and Svelte developers should recognize the format immediately.

# `script`

![06-script](https://dev-to-uploads.s3.amazonaws.com/i/1cmocynedh9uj80zt49b.jpg)

The script tag starts by declaring two variables (`var`!!) and contains five functions for (I'm assuming based on the names):
* initializing the document
* formatting the document
* validating the input
* setting document mode (switching between html or output)
* printing the document

# `style`

![07-style](https://dev-to-uploads.s3.amazonaws.com/i/nfdb7635e4thqixfb5lq.jpg)

The styling provides just a little CSS to format the editor and give a few resets.

# `body`

The body contains one giant html form

![08-form](https://dev-to-uploads.s3.amazonaws.com/i/kuowyq05jienmpfdmj24.jpg)

Nested inside the form:
* input tag
* div for first toolbar
* div for second toolbar
* div for text box
* paragraph tag containing an input tag for Show HTML button
* paragraph tag containing an input tag for send button

# `toolBar1`

The first toolbar is a series of five select tags for formatting, font, size, color, and background color.

![09-toolbar1](https://dev-to-uploads.s3.amazonaws.com/i/iqzg55m32101b9kp5bum.jpg)

Each select has an `onchange` handler and a series of `option` tags for the different choices. 

# `toolBar2`

The second toolbar contains a fair bit of code since it has a lot going on.

![10-toolbar2](https://dev-to-uploads.s3.amazonaws.com/i/dqzblm33hkrspwt4axbf.jpg)

The links inside the src attributes where originally much longer but I deleted most of it just for the sake of getting a more readable screenshot of the whole toolbar. Refer to the code on the MDN site for implementation