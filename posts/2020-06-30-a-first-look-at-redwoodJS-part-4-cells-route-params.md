---
title: a first look at redwoodJS part 4 - cells, route params
description: Set up our frontend to render a list of our blog posts by querying data from our backend
date: 2020-06-30
tags:
  - redwoodjs
  - graphql
  - react
  - cells
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/7x38o4nssnlf0d9hdo5h.png
layout: layouts/post.njk
---

>*What I’ve experienced, and what I know many people have experienced learning React and getting into this is... that path right now is very, very, very, very, very long. And hard. And horrible.*
>
>***Tom Preston-Werner*** - *[Full Stack Radio](https://www.fullstackradio.com/episodes/138)*

# Part 4 - Cells, Route Params

* In [part 1](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-1017) we installed and created our first RedwoodJS application
* In [part 2](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-part-2-44ph) we created links to our different page routes and a reusable layout for our site
* In [part 3](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-part-3-5ao5) we got our database up and running and learned to create, retrieve, update, and destroy blog posts.

In this part we'll set up our frontend to query data from our backend to render a list of our blog posts to the front page. If you've never worked with [GraphQL](https://www.howtographql.com/) or serverless functions like [Lambda](https://www.serverless.com/aws-lambda/) then some of the concepts in this part may be new.

## `redwood generate cell`

Now we are going to create a cell that will render the most recent blog posts to the front page.

```bash
yarn rw g cell BlogPosts
```

```
✔ Generating cell files...
  ✔ Successfully wrote file `./web/src/components/BlogPostsCell/BlogPostsCell.mock.js`
  ✔ Successfully wrote file `./web/src/components/BlogPostsCell/BlogPostsCell.test.js`
  ✔ Successfully wrote file `./web/src/components/BlogPostsCell/BlogPostsCell.stories.js`
  ✔ Successfully wrote file `./web/src/components/BlogPostsCell/BlogPostsCell.js`
```

In `BlogPostsCell.js` there will be a `QUERY` that uses `JSON.stringify` to render the results of the query. But there is one thing we need to change.

{% raw %}
```jsx
// web/src/components/BlogPostsCell/BlogPostsCell.js

export const QUERY = gql`
  query BlogPostsQuery {
    blogPosts {
      id
    }
  }
`

export const Loading = () => <div>Loading...</div>
export const Empty = () => <div>No posts yet!</div>
export const Failure = ({ error }) => (
  <div style={{ color: 'red' }}>Error: {error.message}</div>
)

export const Success = ({ blogPosts }) => {
  return (
    <ul>
      {blogPosts.map((item) => {
        return <li key={item.id}>{JSON.stringify(item)}</li>
      })}
    </ul>
  )
}
```
{% endraw %}

We need to make a slight adjustment to get our QUERY to match up with the schema that we have already created. Change each instance of `blogPosts` to just `posts`.

{% raw %}
```jsx
// web/src/components/BlogPostsCell/BlogPostsCell.js

export const QUERY = gql`
  query BlogPostsQuery {
    posts {
      id
    }
  }
`

export const Loading = () => <div>Almost there...</div>
export const Empty = () => <div>I NEED A POST</div>
export const Failure = ({ error }) => (
  <div style={{ color: 'red' }}>Error: {error.message}</div>
)

export const Success = ({ posts }) => {
  return (
    <ul>
      {posts.map((item) => {
        return <li key={item.id}>{JSON.stringify(item)}</li>
      })}
    </ul>
  )
}
```
{% endraw %}

Now we can take the `BlogPostsCell` component and insert it into our `HomePage` component. We need to first import it, and then place the tag inside of the `BlogLayout` tags.

```jsx
// web/src/pages/HomePage/HomePage.js

import BlogLayout from 'src/layouts/BlogLayout'
import BlogPostsCell from 'src/components/BlogPostsCell'

const HomePage = () => {
  return (
    <>
      <BlogLayout>
        <p>This page is the home!</p>
        <BlogPostsCell />
      </BlogLayout>
    </>
  )
}

export default HomePage
```

This gives us just the `id` and the `typename` which is `Post`.

![01-BlogPostsCell-render-HomePage](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hta2huh6eowfkm70bvey.png)

Lets go back to our `QUERY` and add in `title`, `body`, and `createdAt`.

```jsx
// web/src/components/BlogPostsCell/BlogPostsCell.js

export const QUERY = gql`
  query BlogPostsQuery {
    posts {
      id
      title
      body
      createdAt
    }
  }
`

// Loading, Empty, Failure...

export const Success = ({ posts }) => {
  return json.stringify(posts)
}
```

Now we should get all the info we need on the home page.

![02-BlogPostsCell-render-id-title-body-createdAt](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0ipdq8exvl2hvtpvpzyb.png)

This doesn't look very good though, I don't think anyone would want to read this blog. In the `BlogPostsCell` file inside `Success` we can create a component for our post titles.

```jsx
export const Success = ({ posts }) => {
  return posts.map(post => (
    <article key={post.id}>
      <header>
        <h2>
          {post.title}
        </h2>
      </header>
    </article>
  ))
}
```

![03-post-titles](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lkkk9k1ln5mqd41hhiwh.png)

```jsx
export const Success = ({ posts }) => {
  return posts.map(post => (
    <article key={post.id}>
      <header>
        <h2>
          {post.title}
        </h2>
      </header>

      <p>
        {post.body}
      </p>

      <time>
        {post.createdAt}
      </time>
    </article>
  ))
}
```

We'll do some more styling later on but for now we have our posts rendered to the front page.

![04-BlogPostsCell-render-map-over-posts](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kwtxxgio54xif0efz3ph.png)

## Routing Params

Now that we have our homepage listing all the posts, let's build the "detail" page with a canonical URL that displays a single post. 

```bash
yarn rw g page BlogPost
```

Link the title of the post on the homepage to the detail page and import `Link` and `routes`.

```jsx
// web/src/components/BlogPostsCell/BlogPostsCell.js

import { Link, routes } from '@redwoodjs/router'

// QUERY, Loading, Empty and Failure definitions...

export const Success = ({ posts }) => {
  return posts.map((post) => (
    <article key={post.id}>
      <header>
        <h2>
          <Link to={routes.blogPost()}>
            {post.title}
          </Link>
        </h2>
      </header>

      <p>
        {post.body}
      </p>

      <time>
        {post.createdAt}
      </time>
    </article>
  ))
}
```

If you click the link on the title of the blog post you should see the boilerplate text on `BlogPostPage`.

![05-](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/d66y8b8vkls70qhh4liz.png)

Since we need to specify which post we want to view on this page, it would be nice to be able to specify the ID of the post in the URL with something like `/blog-post/1`. Tell the `<Route>` to expect another part of the URL and give that part a name that we can reference later.

```jsx
// web/src/Routes.js

<Route
  path="/blog-post/{id}"
  page={BlogPostPage}
  name="blogPost"
/>
```

`{id}` is a **route parameter**. Whatever value is in this position in the path is referenced by the name inside the curly braces. Now we need to construct a link that has the ID of a post in it.

```jsx
// web/src/components/BlogPostsCell/BlogPostsCell.js

<Link to={routes.blogPost(
  { id: post.id }
)}>
  {post.title}
</Link>
```

For routes with route parameters, the named route function expects an object where you specify a value for each parameter. Now clicking the link takes you to `/blog-post/1`.

![06-](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ca8slqhxkcxozwzlqmd8.png)

### Using the Parameter

To display a specific post we'll need to retrieve data from the database. This means we want a cell.

```bash
yarn rw g cell BlogPost
```

We'll import that cell into `BlogPostPage` and wrap the page with `BlogLayout`.

```jsx
// web/src/pages/BlogPostPage/BlogPostPage.js

import BlogLayout from 'src/layouts/BlogLayout'
import BlogPostCell from 'src/components/BlogPostCell'

const BlogPostPage = () => {
  return (
    <BlogLayout>
      <BlogPostCell />
    </BlogLayout>
  )
}

export default BlogPostPage
```

We need access to the `{id}` route param so we can look up the ID of the post in the database. Update the query to accept a variable and change the query name from `blogPost` to just `post`.

```jsx
// web/src/components/BlogPostCell/BlogPostCell.js

export const QUERY = gql`
  query BlogPostQuery($id: Int!) {
    post(id: $id) {
      id
      title
      body
      createdAt
    }
  }
`

export const Loading = () => <div>Just a few more seconds I promise...</div>
export const Empty = () => <div>NO POST HERE SORRY BUD</div>
export const Failure = ({ error }) => <div>{error.message}</div>

export const Success = ({ post }) => {
  return JSON.stringify(post)
}
```

Where will that `$id` come from? Whenever you put a route param in a route, that param is automatically made available to the page that route renders.

```jsx
// web/src/pages/BlogPostPage/BlogPostPage.js

const BlogPostPage = ({ id }) => {
  return (
    <BlogLayout>
      <BlogPostCell id={id} />
    </BlogLayout>
  )
}
```

`id` already exists since we named our route param `{id}`. The `id` ends up as the `$id` because any props you give to a cell will automatically be turned into variables and given to the query.

![07-](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m94kzkjiscrmtn0nney1.png)

If you take a look in the web inspector console you can see the actual error coming from GraphQL:

```
[GraphQL error]: Message: Variable "$id" got invalid value "1";
Expected type Int. Int cannot represent non-integer value: "1",
Location: [object Object], Path: undefined
```

Route params are extracted as strings from the URL, but GraphQL wants an integer for the ID.

### Route Param Types

**Route param types** make it possible to request the conversion right in the route's path by adding `:Int` to the existing route param.

```jsx
// web/src/Routes.js

<Route
  path="/blog-post/{id:Int}"
  page={BlogPostPage}
  name="blogPost"
/>
```

This converts the `id` param to a number before passing it to your Page and prevents the route from matching unless the `id` path segment consists entirely of digits.

![08-](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kmjhcvyuo6qjnh9wszlj.png)

If any non-digits are found, the router will keep trying other routes, eventually showing the `NotFoundPage` if no routes match.

### Displaying a Blog Post

To display the actual post instead of just dumping the query result we'll use a good old fashioned component.

```bash
yarn rw g component BlogPost
```

This creates `web/src/components/BlogPost/BlogPost.js`.

```jsx
// web/src/components/BlogPost/BlogPost.js

const BlogPost = () => {
  return (
    <div>
      <h2>{'BlogPost'}</h2>
      <p>{'Find me in ./web/src/components/BlogPost/BlogPost.js'}</p>
    </div>
  )
}

export default BlogPost
```

Take the post display code out of `BlogPostsCell` and put it here instead, taking the `post` in as a prop:

```jsx
// web/src/components/BlogPost/BlogPost.js

import { Link, routes } from '@redwoodjs/router'

const BlogPost = ({ post }) => {
  return (
    <article>
      <header>
        <h2>
          <Link to={routes.blogPost({ id: post.id })}>
            {post.title}
          </Link>
        </h2>
      </header>

      <div>
        {post.body}
      </div>
    </article>
  )
}

export default BlogPost
```

Update `BlogPostsCell` and `BlogPostCell` to use this new component instead:

```jsx
// web/src/components/BlogPostsCell/BlogPostsCell.js

import BlogPost from 'src/components/BlogPost'

// Loading, Empty, Failure...

export const Success = ({ posts }) => {
  return posts.map(
    (post) => <BlogPost key={post.id} post={post} />
  )
}
```

```jsx
// web/src/components/BlogPostCell/BlogPostCell.js

import BlogPost from 'src/components/BlogPost'

// Loading, Empty, Failure...

export const Success = ({ post }) => {
  return <BlogPost post={post} />
}
```

We can now move back and forth between the homepage and the detail page.

![09-](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/29jy14okxgyfrk9i2l7r.png)

In the next part we will be generating a contact page and taking input from the user with React Hook Form.