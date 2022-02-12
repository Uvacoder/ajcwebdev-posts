---
title: a first look at redwoodJS part 5 - contact, react hook form
description: Combine everything we’ve learned up to this point to generate a contact page and take input from a user
date: 2020-07-03
tags:
  - redwoodjs
  - react
  - jamstack
  - javascript
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/7x38o4nssnlf0d9hdo5h.png
layout: layouts/post.njk
---

>*We see it as a Rails replacement. Anything you would normally do with Rails we hope that you’ll be able to do with Redwood.*
> 
>***Tom Preston-Werner - [Full Stack Radio](https://www.fullstackradio.com/episodes/138)***

# Part 5 - Contact, React Hook Form

If you've made it this far into my series of blog posts I commend you and hope you've found them useful. Here's what we've done so far:

* In [part 1](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-1017) we created our RedwoodJS app.
* In [part 2](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-part-2-44ph) we created links between different pages and a reusable layout.
* In [part 3](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-part-3-5ao5) we got the database up and running and learned CRUD operations for our blog posts.
* In [part 4](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-part-4-2m0g) we set up the frontend to query data from the backend to render a list of blog posts to the front page.

In this part we'll be combining everything we've learned up to this point to generate a contact page and take input from a user. We'll be using the same form tags that we learned about in part 4, which are wrappers around [react-hook-form](https://react-hook-form.com/get-started).

This is the simplest way to create a form, but Redwood can be used with other popular React form libraries like Formik or you can use react-hook-form directly.

# 5.1 `ContactPage`

The first step is to enter the `yarn redwood generate page` command to create our contact page.

```bash
yarn rw g page contact
```

```
✔ Generating page files...
  ✔ Successfully wrote file `./web/src/pages/ContactPage/ContactPage.stories.js`
  ✔ Successfully wrote file `./web/src/pages/ContactPage/ContactPage.test.js`
  ✔ Successfully wrote file `./web/src/pages/ContactPage/ContactPage.js`
✔ Updating routes file...
```

This should look familiar if you've followed along with the whole series.

```jsx
// web/src/pages/ContactPage/ContactPage.js

import { Link, routes } from '@redwoodjs/router'
import { MetaTags } from '@redwoodjs/web'

const ContactPage = () => {
  return (
    <>
      <MetaTags
        title="Contact"
        // description="Contact description"
        /* you should un-comment description and add a unique description, 155 characters or less
      You can look at this documentation for best practices : https://developers.google.com/search/docs/advanced/appearance/good-titles-snippets */
      />

      <h1>ContactPage</h1>
      <p>
        Find me in <code>./web/src/pages/ContactPage/ContactPage.js</code>
      </p>
      <p>
        My default route is named <code>contact</code>, link to me with `
        <Link to={routes.contact()}>Contact</Link>`
      </p>
    </>
  )
}

export default ContactPage
```

Our `ContactPage` component contains the same boilerplate we saw when we created our home page and our about page.

![03-ContactPage-rendered](https://dev-to-uploads.s3.amazonaws.com/i/qetimov8ns4bch6z2bjb.jpg)

Go to BlogLayout and add a link to the contact page.

```jsx
// web/src/layouts/BlogLayout/BlogLayout.js

import { Link, routes } from '@redwoodjs/router'

const BlogLayout = ({ children }) => {
  return (
    <>
      <header>
        <h1>ajcwebdev</h1>

        <nav>
          <ul>
            <li><Link to={routes.home()}>Home</Link></li>
          </ul>
          <ul>
            <li><Link to={routes.about()}>About</Link></li>
          </ul>
          <ul>
            <li><Link to={routes.contact()}>Contact</Link></li>
          </ul>
        </nav>
      </header>

      <main>{children}</main>

      <footer>
        <h3>Find me online:</h3>

        <ul>
          <li><a href="https://dev.to/ajcwebdev">Blog</a></li>
          <li><a href="https://twitter.com/ajcwebdev">Twitter</a></li>
          <li><a href="https://github.com/ajcwebdev">GitHub</a></li>
        </ul>
      </footer>
    </>
  )
}

export default BlogLayout
```

Now we'll import `BlogLayout` into `ContactPage.js` and wrap our contact page content in the `BlogLayout` component.

```jsx
// web/src/pages/ContactPage/ContactPage.js

import BlogLayout from 'src/layouts/BlogLayout'

const ContactPage = () => {
  return (
    <BlogLayout>
      <MetaTags
        title="Contact"
        description="The page that lets you tell me stuff about my things"
      />
      
      <h1>Contact</h1>
      <p>Tell me stuff about my things!</p>
    </BlogLayout>
  )
}

export default ContactPage
```

We can now navigate to any of our three pages.

![04-ContactPage-BlogLayout-rendered](https://dev-to-uploads.s3.amazonaws.com/i/59ek6foqd36xn2bfck1b.jpg)

# 5.3 `Form`

We're going to import the `Form` tags. Refer to the [Redwoodjs docs](https://redwoodjs.com/docs/form) to learn more about these tags.

```jsx
// web/src/pages/ContactPage/ContactPage.js

import BlogLayout from 'src/layouts/BlogLayout'
import {
  Form,
  Label,
  TextField,
  Submit
} from '@redwoodjs/forms'
```

Once the tags are imported, create a `Form` with a `Label`, `TextField`, and `Submit` button.

```jsx
// web/src/pages/ContactPage/ContactPage.js

// imports

const ContactPage = () => {
  return (
    <BlogLayout>
      <h1>Contact</h1>
      <p>Tell me stuff about my things!</p>

      <Form>
        <Label name="name" />

        <TextField name="input" />

        <Submit>Save</Submit>
      </Form>
    </BlogLayout>
  )
}

export default ContactPage
```

![05-ContactPage-Form-rendered](https://dev-to-uploads.s3.amazonaws.com/i/31ilgrhqz0in24nbikql.jpg)

We'll add a little CSS in a moment, but first see what happens if we try to input data.

![06-ContactPage-input](https://dev-to-uploads.s3.amazonaws.com/i/uo0syens75jsah0qnyo4.jpg)

If we click the save button we'll get an error.

![07-error-message](https://dev-to-uploads.s3.amazonaws.com/i/trce1ompqsf27c3vtcu1.jpg)

This makes sense, we haven't told our form what to do yet with the data. Let's create a function called `onSubmit` that will take in a `data` object and console log the `data` object.

```jsx
// web/src/pages/ContactPage/ContactPage.js

const ContactPage = () => {
  const onSubmit = (data) => {
    console.log(data)
  }
  
  return (
    <BlogLayout>
      <h1>Contact</h1>
      <p>Tell me stuff about my things!</p>
    
      <Form onSubmit={onSubmit}>
        <Label name="name" />

        <TextField name="input" />

        <Submit>Save</Submit>
      </Form>
    </BlogLayout>
  )
}

export default ContactPage
```

The `onSubmit` prop accepts a function name or anonymous function to be called if validation is successful. This function will be called with a single object containing key/value pairs of all Redwood form helper fields in your form.

Now if enter data into the form and click save we'll see the following in our console:

![08-console.log(data)](https://dev-to-uploads.s3.amazonaws.com/i/h1qrvfwyamgt0iopa81s.jpg)

# 5.4 `data`

Our input is contained in the `data` object. Right now it only has a key/value pair for name but we'll be adding more in a moment.

Before doing that, what can we do with this `data` object?

```jsx
// web/src/pages/ContactPage/ContactPage.js

const ContactPage = () => {
  const onSubmit = (data) => {
    console.log(data)
    console.log(data.name)
  }
```

We can pull out the value of name by console logging `data.name`:

![09-console.log(data.name)](https://dev-to-uploads.s3.amazonaws.com/i/osd91p0akqcwzss0adbi.jpg)

We want to be able to accept a longer message from our users, so we're going to import the `TextAreaField` tag.

```jsx
// web/src/pages/ContactPage/ContactPage.js

import {
  Form,
  Label,
  TextField,
  TextAreaField,
  Submit
} from '@redwoodjs/forms'
```

We now how a `TextField` for name and email, and a `TextAreaField` for a message.

```jsx
// web/src/pages/ContactPage/ContactPage.js

<Form onSubmit={onSubmit}>
  <Label name="name" />
  <TextField name="name" />

  <Label name="email" />
  <TextField name="email" />

  <Label name="message" />
  <TextAreaField name="message" />

  <Submit>Save</Submit>
</Form>
```

To make this look a little nicer we're going to include just a little CSS.

```css
/* web/src/index.css */

button, input, label, textarea {
  display: block;
  outline: none;
}

label {
  margin-top: 1rem;
}
```

Our buttons, inputs, and labels are now `display: block` which adds a line break after any appearance of these tags, and the label also has a little bit of margin on the top.

![10-ContactPage-email-message-rendered](https://dev-to-uploads.s3.amazonaws.com/i/u4ybsebzhtkfsvvzp5ak.jpg)

We'll test out all the fields:

![11-ContactPage-email-message-input](https://dev-to-uploads.s3.amazonaws.com/i/4wt88cv6lgox7fluhiia.jpg)

We now are getting back an object with three key/value pairs.

![12-ContactPage-email-message-console.log](https://dev-to-uploads.s3.amazonaws.com/i/39xsmt7mee7rxbd81xo2.jpg)

We can console log any part of the object that we want.

```jsx
// web/src/pages/ContactPage/ContactPage.js

const ContactPage = () => {
  const onSubmit = (data) => {
    console.log(data)
    console.log(data.name)
    console.log(data.email)
    console.log(data.message)
  }
```

Now if we look at our console we'll see each output and it even tells us which file and line corresponds to each piece of data.

![13-data.email-data.message-console.log](https://dev-to-uploads.s3.amazonaws.com/i/ess2h63bpmrvb7g88mk7.jpg)

# 5.5 `validation`

What happens if we only fill out some of the form and try to submit?

![14-just-name-input](https://dev-to-uploads.s3.amazonaws.com/i/uuz0e8qnzc73mijdzgrf.jpg)

The form doesn't care, it simply takes the empty inputs and returns an empty string.

![15-empty-email-message](https://dev-to-uploads.s3.amazonaws.com/i/yb6fweldvgf1oep19qth.jpg)

We want to add some validation so the user can't submit the form unless they've given input for all three fields.

# 5.6 `errorClassName`

We gave each `TextField` an `errorClassName` with the attribute `error`. The `validation` prop accepts an object containing options for react-hook-form.

{% raw %}
```jsx
// web/src/pages/ContactPage/ContactPage.js

<Form onSubmit={onSubmit}>
  <Label name="name" />
  <TextField
    name="name"
    errorClassName="error"
    validation={{ required: true }}
  />

  <Label name="email" />
  <TextField
    name="email"
    errorClassName="error"
    validation={{ required: true }}
  />

  <Label name="message" />
  <TextAreaField
    name="message"
    errorClassName="error"
    validation={{ required: true }}
  />

  <Submit>Save</Submit>
</Form>
```
{% endraw %}

Right now we're just adding the `required` attribute, but later we'll use the `validation` prop for a regular expression.

In our CSS we'll add the following properties for errors.

```css
/* web/src/index.css */

.error {
  color: red;
}

input.error, textarea.error {
  border: 1px solid red;
}
```

![16-textfield-errors](https://dev-to-uploads.s3.amazonaws.com/i/zskdp29l2n749xedk49z.jpg)

Now when we try to submit an empty field we see the color change to red.

![17-textfield-errors-with-name](https://dev-to-uploads.s3.amazonaws.com/i/8a4a9hmntix12u8yb0ey.jpg)

Once we give input the red error color goes away.

![18-invalid-email](https://dev-to-uploads.s3.amazonaws.com/i/qcvdmkctrawzwdk3mce7.jpg)

There's still an issue, which is we can submit an email that is invalid.

{% raw %}
```jsx
// web/src/pages/ContactPage/ContactPage.js

<TextField
  name="email"
  validation={{
    required: true,
    pattern: {
      value: /[^@]+@[^.]+\..+/,
    },
  }}
  errorClassName="error"
/>
```
{% endraw %}

Here's a regular expression provided in the Redwood tutorial.

![19-email-error](https://dev-to-uploads.s3.amazonaws.com/i/isd4dsj7incmt6ydjhqc.jpg)

# 5.7 `FieldError`

Now we get an error if we don't provide a valid email address. It would really be nice if we could tell our user why they are getting an error.

```jsx
// web/src/pages/ContactPage/ContactPage.js

import {
  Form,
  Label,
  TextField,
  TextAreaField,
  FieldError,
  Submit
} from '@redwoodjs/forms'
```

We're going to import `FieldError` to show error messages to our users.

{% raw %}
```jsx
// web/src/pages/ContactPage/ContactPage.js

<Form onSubmit={onSubmit}>
  <Label name="name" />
  <TextField
    name="name"
    errorClassName="error"
    validation={{ required: true }}
  />
  <FieldError name="name" />

  <Label name="email" />
  <TextField
    name="email"
    errorClassName="error"
    validation={{
      required: true,
      pattern: { value: /[^@]+@[^.]+\..+/, },
    }}
  />
  <FieldError name="email" />

  <Label name="message" />
  <TextAreaField
    name="message"
    errorClassName="error"
    validation={{ required: true }}
  />
  <FieldError name="message" />

  <Submit>Save</Submit>
</Form>
```
{% endraw %}

Now if we try to submit without giving input we are told that the field is required.

![20-name-email-message-required](https://dev-to-uploads.s3.amazonaws.com/i/m98sue4jhsi79cxzgypp.jpg)

If we enter an invalid email we are told that the email is not formatted correctly.

![21-email-formatted-incorrectly](https://dev-to-uploads.s3.amazonaws.com/i/lfa9jlhqjle8mcwizxgi.jpg)

If we add the `errorClassName` to the `Label` tags we'll also turn the labels red if there's an error.

{% raw %}
```jsx
// web/src/pages/ContactPage/ContactPage.js

<Form onSubmit={onSubmit}>
  <Label
    name="name"
    errorClassName="error"
  />
  <TextField
    name="name"
    errorClassName="error"
    validation={{ required: true }}
  />
  <FieldError name="name" />

  <Label
    name="email"
    errorClassName="error"
  />
  <TextField
    name="email"
    errorClassName="error"
    validation={{
      required: true,
      pattern: { value: /[^@]+@[^.]+\..+/, },
    }}
  />
  <FieldError name="email" />

  <Label
    name="message"
    errorClassName="error"
  />
  <TextAreaField
    name="message"
    errorClassName="error"
    validation={{ required: true }}
  />
  <FieldError name="message" />

  <Submit>Save</Submit>
</Form>
```
{% endraw %}

Now we have the label and the input field turn red if there's an error.

![22-label-errors](https://dev-to-uploads.s3.amazonaws.com/i/7t6bgs24k2jo3ip5ixyb.jpg)

Might as well make everything red while we're at it.

{% raw %}
```jsx
// web/src/pages/ContactPage/ContactPage.js

<Form onSubmit={onSubmit}>
  <Label
    name="name"
    errorClassName="error"
  />
  <TextField
    name="name"
    errorClassName="error"
    validation={{ required: true }}
  />
  <FieldError
    name="name"
    style={{ color: 'red' }}
  />

  <Label
    name="email"
    errorClassName="error"
  />
  <TextField
    name="email"
    errorClassName="error"
    validation={{
      required: true,
      pattern: { value: /[^@]+@[^.]+\..+/, },
    }}
  />
  <FieldError
    name="email"
    style={{ color: 'red' }}
  />

  <Label
    name="message"
    errorClassName="error"
  />
  <TextAreaField
    name="message"
    errorClassName="error"
    validation={{ required: true }}
  />
  <FieldError
    name="message"
    style={{ color: 'red' }}
  />

  <Submit>Save</Submit>
</Form>
```
{% endraw %}

Since the `FieldError` will only show on errors we can just inline styles with {% raw %}`style={{ color: 'red' }}`{% endraw %}.

![23-all-red](https://dev-to-uploads.s3.amazonaws.com/i/vrnrjmvxcsoxowdu54ro.jpg)

Glorious red as far as the eye can see. It would also be nice if we could tell the user a field is required before they hit the submit button. We'll do that by adding `mode: 'onBlur'` and pass that into `validation`.

```jsx
// web/src/pages/ContactPage/ContactPage.js

<Form
  onSubmit={onSubmit}
  validation={
    { mode: 'onBlur' }
  }
>
```

Now when we enter a field and leave without filling it out we are immediately given feedback.

![24-form-validation-mode-onBlur-rendered](https://dev-to-uploads.s3.amazonaws.com/i/l5hwrv68vng8csmvs9k0.jpg)

And for now that's our entire form. Here's a look at all the form code.

{% raw %}
```jsx
// web/src/pages/ContactPage/ContactPage.js

return (
  <BlogLayout>
    <h1>Contact</h1>
    <p>Tell me stuff about my things!</p>

    <Form onSubmit={onSubmit} validation={{ mode: 'onBlur' }}>
      <Label
        name="name"
        errorClassName="error"
      />
      <TextField
        name="name"
        errorClassName="error"
        validation={{ required: true }}
      />
      <FieldError
        name="name"
        style={{ color: 'red' }}
      />

      <Label
        name="email"
        errorClassName="error"
      />
      <TextField
        name="email"
        errorClassName="error"
        validation={{
          required: true, pattern: { value: /[^@]+@[^.]+\..+/, },
        }}
      />
      <FieldError
        name="email"
        style={{ color: 'red' }}
      />

      <Label
        name="message"
        errorClassName="error"
      />
      <TextAreaField
        name="message"
        errorClassName="error"
        validation={{ required: true }}
      />
      <FieldError
        name="message"
        style={{ color: 'red' }}
      />

      <Submit>Save</Submit>
    </Form>
  </BlogLayout>
)
```
{% endraw %}

In the [next part](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-part-6-a25) we'll connect our contact form to our database so we can persistent the data entered into the form.