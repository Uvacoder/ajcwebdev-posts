---
title: how to display a custom greeting based on the day of the week
description: With just a couple lines of JavaScript you can create a message that displays a different greeting depending on the day of the week.
date: 2021-11-10
tags:
  - html
  - javascript
  - date
  - DOM
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5vzr5k8b5kszgzt10wp8.jpeg
layout: layouts/post.njk
---

I discovered a cool little trick while source diving through [Scott Mathson](https://scottmathson.com/)'s web site. With just a couple lines of JavaScript you can create a message that displays a different greeting depending on the day of the week.

### Create a script with a weekday array

Create a `<script>` tag with `type` of `text/javascript`. Define a variable called `weekday` with a different greeting set to each index.

```html
<script type="text/javascript">

  var weekday = new Array(7);

  weekday[0] = "spectacular Sunday";
  weekday[1] = "marvelous Monday";
  weekday[2] = "terrific Tuesday";
  weekday[3] = "wonderful Wednesday";
  weekday[4] = "totally cool Thursday";
  weekday[5] = "fantastic Friday";
  weekday[6] = "sweet Saturday";

</script>
```

### Set weekday value to the current date

Also inside the script tag, create a variable called `currentDate` set with the [`Date()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) object and then set the current day to `weekdayValue`.

```javascript
var currentDate = new Date();
weekdayValue = currentDate.getDay();
```

### Write to the Document

Use the [Document.write()](https://developer.mozilla.org/en-US/docs/Web/API/Document/write) method to write a string of text to the document with paragraph tags containing the weekday value..

```javascript
document.write(
  '<p>'
    + 'Have a ' + weekday[weekdayValue] + '!' + 
  '</p>'
);
```

### Noscript fallback

Lastly, you'll want to include a `<noscript>` tag in case the user has JavaScript turned off in their browser.

```html
<noscript>
  <p>
    Have a great day!
  </p>
</noscript>
```

## Full script

```html
<script type="text/javascript">

  var weekday = new Array(7);

  weekday[0] = "spectacular Sunday";
  weekday[1] = "marvelous Monday";
  weekday[2] = "terrific Tuesday";
  weekday[3] = "wonderful Wednesday";
  weekday[4] = "totally cool Thursday";
  weekday[5] = "fantastic Friday";
  weekday[6] = "sweet Saturday";

  var currentDate = new Date();
  weekdayValue = currentDate.getDay();

  document.write(
    '<p>'
      + 'Have a ' + weekday[weekdayValue] + '!' + 
    '</p>'
  );
</script>

<noscript>
  <p>
    Have a great day!
  </p>
</noscript>
```