---
title: a first look at redwoodJS part 6 - sdl, useMutation
description: Connect our contact form to the database to persist data entered into the form
date: 2020-07-26
tags:
  - redwoodjs
  - graphql
  - prisma
  - form
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/7x38o4nssnlf0d9hdo5h.png
layout: layouts/post.njk
---

>*GraphQL becomes the one and only mediator between your frontend and your backend.*  
>
>***Tom Preston-Werner - [RedwoodJS with Tom Preston-Werner](https://www.softwaredaily.com/post/5ec7997912b353000c6381d8/RedwoodJS-with-Tom-PrestonWerner)***

# Part 6 - SDL, useMutation

In [part 5](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-part-5-2ol4) we generated a contact page and took input from the user. In this part we'll connect our contact form to our database so we can persist the data entered into the form.

# 6.1 `Contact`

To modify our database we'll go to our `schema.prisma` file and add a `Contact` model.

```
// api/db/schema.prisma

model Contact {
  id        Int      @id @default(autoincrement())
  name      String
  email     String
  message   String
  createdAt DateTime @default(now())
}
```

The [model](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-schema/data-model) takes the `name`, `email`, and `message` strings from the form inputs. It creates an `id` with `autoincrement()` and `createdAt` time with [`now()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/now).

Enter `yarn rw prisma migrate dev` to create a migration.

```bash
yarn rw prisma migrate dev --name contact-it
```

```
Running Prisma CLI:

Prisma schema loaded from db/schema.prisma
Datasource "DS": PostgreSQL database "railway", schema "public" at "containers-us-west-1.railway.app:5884"

The following migration(s) have been created and applied from new schema changes:

migrations/
  └─ 20210307135440_contact_it/
    └─ migration.sql

✔ Generated Prisma Client (2.16.1) to ./../node_modules/@prisma/client in 66ms

Everything is now in sync.
```

```sql
CREATE TABLE "Contact" (
    "id" SERIAL NOT NULL,
    "name" TEXT NOT NULL,
    "email" TEXT NOT NULL,
    "message" TEXT NOT NULL,
    "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY ("id")
)
```

# 6.2 `redwood generate sdl`

Enter `yarn redwood generate sdl contact` to create the schema definition language for the contact form.

```bash
yarn rw g sdl contact
```

```
✔ Generating SDL files...
  ✔ Successfully wrote file `./api/src/graphql/contacts.sdl.js`
  ✔ Successfully wrote file `./api/src/services/contacts/contacts.scenarios.js`
  ✔ Successfully wrote file `./api/src/services/contacts/contacts.test.js`
  ✔ Successfully wrote file `./api/src/services/contacts/contacts.js`
```

This will create an sdl in your `graphql` folder and a resolver in your `services` folder.

![06-api-src-folder](https://dev-to-uploads.s3.amazonaws.com/i/mtos8csbkgarchjy4nox.jpg)

# 6.3 `contacts.sdl.js`

We'll look at our schema definition language first.

```javascript
// api/src/graphql/contacts.sdl.js

export const schema = gql`
  type Contact {
    id: Int!
    name: String!
    email: String!
    message: String!
    createdAt: DateTime!
  }

  type Query {
    contacts: [Contact!]!
  }

  input CreateContactInput {
    name: String!
    email: String!
    message: String!
  }

  input UpdateContactInput {
    name: String
    email: String
    message: String
  }
`
```

The sdl has two types:
* `Contact`
* `Query`

And two inputs:
* `CreateContactInput`
* `UpdateContactInput`

# 6.4 `createContact`

We need to create our own mutation. We don't get this by default because the sdl generator doesn't know what we plan to do with the schema.

```javascript
// api/src/graphql/contacts.sdl.js

export const schema = gql`
  type Contact {
    id: Int!
    name: String!
    email: String!
    message: String!
    createdAt: DateTime!
  }

  type Query {
    contacts: [Contact!]!
  }

  input CreateContactInput {
    name: String!
    email: String!
    message: String!
  }

  input UpdateContactInput {
    name: String
    email: String
    message: String
  }

  type Mutation {
    createContact(
      input: CreateContactInput!
    ): Contact
  }
`
```

Our `Mutation` type will be called `createContact`. The input is `CreateContactInput!` and it returns `Contact`.

# 6.5 `contacts.js`

Now we'll look at our `contacts` service. It has a resolver for the `Query` type from our sdl.

```javascript
// api/src/services/contacts/contacts.js

import { db } from 'src/lib/db'

export const contacts = () => {
  return db.contact.findMany()
}
```

We'll create a resolver for `createContact`.

```javascript
// api/src/services/contacts/contacts.js

import { db } from 'src/lib/db'

export const contacts = () => {
  return db.contact.findMany()
}

export const createContact = ({ input }) => {
  return db.contact.create(
    { data: input }
  )
}
```

This takes `input` from the browser and adds it to the database.

# 6.6 `CREATE_CONTACT`

In our `ContactPage` file we'll add the mutation and assign it to `CREATE_CONTACT`. It is a common convention to use all caps for our mutations.

```jsx
// web/src/pages/ContactPage/ContactPage.js

const CREATE_CONTACT = gql`
  mutation CreateContactMutation($input: CreateContactInput!) {
    createContact(input: $input) {
      id
    }
  }
`
```

We will just return the `id` since the user will not see this. Make sure to import `useMutation` at the top of the file.

```jsx
// web/src/pages/ContactPage/ContactPage.js

import { useMutation } from '@redwoodjs/web'
```

Inside the `ContactPage` component we'll call `useMutation()`. We pass an object `create` with variables that are defined in `CREATE_CONTACT`.

```jsx
// web/src/pages/ContactPage/ContactPage.js

const ContactPage = () => {
  const [create] = useMutation(CREATE_CONTACT)

  const onSubmit = (data) => {
    create({
      variables: {
        input: data
      }
    })
    console.log(data)
  }

  return (...)
}
```

The input is the actual object containing the `data` which is assigned to `variables` and passed to the `create()` function inside `onSubmit`. We are now ready to enter our data. Enter a name, email, and message and click the Save button.

![07-form-input](https://dev-to-uploads.s3.amazonaws.com/i/kjzs5a7m4e086ullmreo.jpg)

Our form gives us the following object which we saw in the previous part.

![08-form-input-console-log](https://dev-to-uploads.s3.amazonaws.com/i/4z7jry0ch4sgdy5ibzn6.jpg)

# 6.7 `graphiQL`

We can now check to make sure it was saved. Remember all the way back in part 1 when we said there was something else running on `localhost:8911`?

![09-localhost-8911](https://dev-to-uploads.s3.amazonaws.com/i/1zhyxon1bkt73nyp26zr.jpg)

This is how we access the graphiQL interface. Click the graphql link.

![10-graphiQL](https://dev-to-uploads.s3.amazonaws.com/i/512op701y527bb8offur.jpg)

GraphiQL is like an IDE for your GraphQL queries. On the left side you'll enter a query, and after clicking the middle play button the response will appear on the right side.

We'll create a `contact` query that will return all the `contacts` in the database.

```graphql
query ContactQuery {
  contacts {
    id
    name
    email
    message
  }
}
```

We'll ask for the `id`, `name`, `email`, and `message`.

Click the play button and on the right side you'll receive a `data` object containing whatever you entered into your form.

![12-contact-data](https://dev-to-uploads.s3.amazonaws.com/i/pp55ohfrk4pdeir1aamf.jpg)

The `data` object contains a `contacts` array with objects containing the form input. There's a few more things we can add to our form to make the user experience a little better.
1. We want to give the user some immediate feedback after they click the save button.
2. We also want to put in safeguards in case the user feels like clicking the save button many times in fast succession.

# 6.8 `loading`

To do this we'll add two objects after `create`:
* `loading`
* `error`

```jsx
// web/src/pages/ContactPage/ContactPage.js

const ContactPage = () => {
  const [
    create,
    { loading, error }
  ] = useMutation(CREATE_CONTACT)

  const onSubmit = (data) => {...}
  return (...)
}
```

`loading` will be true while the mutation is being performed, and will change to false when the mutation is complete. Add `disabled` to the `Submit` button and pass it the `loading` object.

```jsx
// web/src/pages/ContactPage/ContactPage.js

return (
  // ...
  <Submit disabled={loading}>
    Save
  </Submit>
  // ...
)
```

While the mutation is in progress disabled is true, and once it is complete it will change to false. To test this out go to your Network tab in your browsers devtools. Here we can simulate slow 3G.

![13-slow-3G](https://dev-to-uploads.s3.amazonaws.com/i/nj3ubs2v4ptbp6okikta.jpg)

Enter some more data and click the Save button.

![14-save-button-greyed-out](https://dev-to-uploads.s3.amazonaws.com/i/xm2ykilh8nvwlcmlsqjr.jpg)

Now when you click the Save button it will be greyed out for a moment while the input is saved to the database.

# 6.9 `useForm`

Another problem with this form is the input sticks around on the page even after it is submitted. It would be better if we cleared the fields after submitting. We can do this by importing `useForm` from `react-hook-form`.

```jsx
// web/src/pages/ContactPage/ContactPage.js

import { useForm } from 'react-hook-form'
```

Inside our `ContactPage` component we'll create a `formMethods` variable and assign it the `useForm()` function that we just imported.

```jsx
// web/src/pages/ContactPage/ContactPage.js

const ContactPage = () => {
  const formMethods = useForm()
  //...
```

Inside `<Form>` we'll add `formMethods` and pass it `formMethods` as a prop.

{% raw %}
```jsx
// web/src/pages/ContactPage/ContactPage.js

<Form
  onSubmit={
    onSubmit
  }
  validation={{
    mode: 'onBlur'
  }}
  formMethods={formMethods}
>
```
{% endraw %}

This gives us access to the reset function. In `useMutation` we'll add a second parameter, which is a callback called `onCompleted`.

```jsx
// web/src/pages/ContactPage/ContactPage.js

const ContactPage = () => {
  const formMethods = useForm()
  const [create, { loading } ] = useMutation(CREATE_CONTACT, {
    onCompleted: () => {
      formMethods.reset()
      alert('Thanks for telling me stuff about my things!')
    }
  })
```

When the mutation is complete we'll call `formMethods.reset()` and display an `alert()` saying `Thanks for telling me stuff about my things!` To test this in the browser, enter some more data and click the Save button.

![15-alert-thanks-for-the-message](https://dev-to-uploads.s3.amazonaws.com/i/ttuy2eu9b30735ggvyt7.jpg)

First the alert message appears on the screen. After clicking OK the form will be cleared.

![16-form-cleared](https://dev-to-uploads.s3.amazonaws.com/i/hl362fprnvvy4vp8qfco.jpg)

Another thing we can improve is the label. Right now it is just defaulting to whatever is entered as the `name` attribute in `<Label>`. We can add a closing `</Label>` tag and enter something between the tags to change the label. We'll capitalize the labels.

```javascript
<Label
  name="name"
  errorClassName="error"
>
  Name
</Label>

<Label
  name="email"
  errorClassName="error"
>
  Email
</Label>

<Label
  name="message"
  errorClassName="error"
>
  Message
</Label>
```

Check your form for the changes.

![17-updated-form-labels](https://dev-to-uploads.s3.amazonaws.com/i/gdahhgdkodvp3ct0oodl.jpg)

In the [next part](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-part-7-22b0) we'll add authentication to our project so the haxz0rs can't steal your Dogecoin.