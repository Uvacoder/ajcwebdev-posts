---
title: a first look at redwoodJS part 7 - authentication, netlify identity
description: Implement authentication with Netlify Identity
date: 2020-07-30
tags:
  - redwoodjs
  - react
  - netlify
  - jamstack
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/7x38o4nssnlf0d9hdo5h.png
layout: layouts/post.njk
---

Unfortunately I do not have any epic Tom quotes about authentication, so I'll leave you with this:

>*I have plenty of ideas.*
>  
>***Tom Preston-Werner - [RedwoodJS with Tom Preston-Werner](https://www.softwaredaily.com/post/5ec7997912b353000c6381d8/RedwoodJS-with-Tom-PrestonWerner)***

# Part 7 - Authentication

In this penultimate part we'll add authentication to our application, something that has never confused any developer ever. We're going to implement a login form so no one can edit our blog willy-nilly. We will accomplish this with:
* [@redwoodjs/auth](https://www.npmjs.com/package/@redwoodjs/auth) - lightweight wrapper around popular SPA auth libraries
* [Netlify Identity Widget](https://github.com/netlify/netlify-identity-widget) - component for authenticating with Netlify's Identity service

However, this is not the only way to implement authentication with Redwood. Redwood currently supports a wide range of authentication providers including:

- [Auth0](https://github.com/auth0/auth0-spa-js)
- [Azure Active Directory](https://github.com/AzureAD/microsoft-authentication-library-for-js)
- [Netlify GoTrue-JS](https://github.com/netlify/gotrue-js)
- [Magic Links - Magic.js](https://github.com/MagicHQ/magic-js)
- [Ethereum](https://github.com/oneclickdapp/ethereum-auth)
- [Supabase](https://supabase.io/docs/guides/auth)
- [Nhost](https://docs.nhost.io/auth)
- [Firebase's GoogleAuthProvider](https://firebase.google.com/docs/reference/js/firebase.auth.GoogleAuthProvider)

Check out the [Auth Playground](https://github.com/redwoodjs/playground-auth) to explore the other providers.

## 7.1 Administration

The first step we will take on our journey to securing a Redwood applications includes updating the `/posts` routes to include admin screens at `/admin`. The four routes starting with `/posts` will now start with `/admin/posts` instead:

```html
// web/src/Routes.js

<Route
  path="/admin/posts/new"
  page={NewPostPage}
  name="newPost"
/>

<Route
  path="/admin/posts/{id:Int}/edit"
  page={EditPostPage}
  name="editPost"
/>

<Route
  path="/admin/posts/{id:Int}"
  page={PostPage}
  name="post"
/>

<Route
  path="/admin/posts"
  page={PostsPage}
  name="posts"
/>
```

Open `http://localhost:8910/admin/posts` to see the generated scaffold page. We don't have to update any of the `<Link>`s that were generated by the scaffolds because the named route functions didn't change. We have now moved the route to a different path, but it is still an unsecured route.

## 7.2 `rw generate auth`

A subset of the previously mentioned auth providers can also be easily configured by using the `rw generate auth` command.

```bash
yarn rw setup auth <provider>
```

Choose the `provider` option based on your own authentication provider. Supported options include:
* `auth0`
* `firebase`
* `goTrue`
* `magicLink`
* `netlify`

Since we are using the Netlify Identity Widget we will use the `netlify` option.

```bash
yarn rw setup auth netlify
```

This will create an `auth.js` file in `api/src/lib` and automatically modify our `index.js` file in `web/src` and our `graphql.js` file in `api/src/functions`.

```
✔ Generating auth lib...
  ✔ Successfully wrote file `./api/src/lib/auth.js`
✔ Adding auth config to web...
✔ Adding auth config to GraphQL API...
✔ Adding required web packages...
✔ Installing packages...
✔ One more thing...

You will need to enable Identity on your Netlify site and configure the API endpoint.

See: https://github.com/netlify/netlify-identity-widget#localhost
```

If we check our `package.json` file in our `web` folder we'll see two new dependencies:
* `@redwoodjs/auth`
* `netlify-identity-widget`

### `AuthProvider`

If we check our `App.js` file in `web/src` we will see the code modifications that were performed by the `yarn rw setup auth` command.

```jsx
// web/src/App.js

import { AuthProvider } from '@redwoodjs/auth'
import netlifyIdentity from 'netlify-identity-widget'
import { isBrowser } from '@redwoodjs/prerender/browserUtils'

isBrowser && netlifyIdentity.init()

const App = () => (
  <FatalErrorBoundary page={FatalErrorPage}>
    <AuthProvider
      client={netlifyIdentity}
      type="netlify"
    >
      <RedwoodApolloProvider>
        <Routes />
      </RedwoodApolloProvider>
    </AuthProvider>
  </FatalErrorBoundary>
)

export default App
```

`AuthProvider` wraps the Router and takes in a `client` and `type` which are `netlifyIdentity` and `netlify` respectively.

### `getCurrentUser`

Our `graphql.js` file in the `api/src/functions` directory is importing `getCurrentUser` and then passing it to the `handler` that sets up our GraphQL API so we don't have to worry about it.

```javascript
// api/src/functions/graphql.js

import { getCurrentUser } from 'src/lib/auth'

export const handler = createGraphQLHandler({
  getCurrentUser,
  schema: makeMergedSchema({
    schemas,
    services: makeServices({ services }),
  }),
})
```

### `auth.js`

The `auth.js` file was generated in our `api/src` folder.

![11-lib-auth.js](https://dev-to-uploads.s3.amazonaws.com/i/y2ho6mvb8ijdwh6l6irp.jpg)

Here is the generated file with comments removed.

```javascript
// api/src/lib/auth.js

import {
  AuthenticationError,
  ForbiddenError,
  parseJWT
} from '@redwoodjs/api'

export const getCurrentUser = async (decoded, { _token, _type }, { _event, _context }) => {
  return { ...decoded, roles: parseJWT({ decoded }).roles }
}

export const requireAuth = ({ role } = {}) => {
  if (!context.currentUser) {
    throw new AuthenticationError("You don't have permission to do that.")
  }

  if (
    typeof role !== 'undefined' &&
    typeof role === 'string' &&
    !context.currentUser.roles?.includes(role)
  ) {
    throw new ForbiddenError("You don't have access to do that.")
  }

  if (
    typeof role !== 'undefined' &&
    Array.isArray(role) &&
    !context.currentUser.roles?.some((r) => role.includes(r))
  ) {
    throw new ForbiddenError("You don't have access to do that.")
  }
}
```

## 7.3 API Authentication

First let's lock down the API so we can be sure that only authorized users can create, update and delete a Post.

### `requireAuth`

We'll import `requireAuth` and use it to restrict access to our endpoints. It is a helper method used in our services. If someone is not authenticated it will throw an error.

```javascript
// api/src/services/posts/posts.js

import { db } from 'src/lib/db'
import { requireAuth } from 'src/lib/auth'

export const posts = () => {
  return db.post.findMany()
}

export const post = ({ id }) => {
  return db.post.findUnique({
    where: { id },
  })
}

export const createPost = ({ input }) => {
  requireAuth()
  return db.post.create({
    date: input,
  })
}

export const updatePost = ({ id, input }) => {
  requireAuth()
  return db.post.update({
    data: input,
    where: { id },
  })
}

export const deletePost = ({ id }) => {
  requireAuth()
  return db.post.delete({
    where: { id },
  })
}
```

We want to restrict access to the sensitive endpoints including `createPost`, `updatePost`, and `deletePost`.

![14-test-new-post](https://dev-to-uploads.s3.amazonaws.com/i/j5jayhvnq8ksx7oir0ps.jpg)

Try to make a new post.

![15-don't-have-permission-to-make-new-post](https://dev-to-uploads.s3.amazonaws.com/i/0odsc2xc7ijj8xfumf3f.jpg)

## 7.4 Web Authentication

Now we'll restrict access to the admin pages completely unless you're logged in. The first step will be to denote which routes will require that you be logged in.

### `Private`

We were told that we don't have permission to create a post. But we want to make it so an unauthenticated user can't even get this far. Let's create private routes for all the `/posts` routes. We'll import `Private` from `@redwoodjs/router` and use it to wrap our `/posts` routes.

```jsx
// web/src/Routes.js

import { Router, Route, Set, Private } from '@redwoodjs/router'
import BlogPostLayout from 'src/layouts/BlogPostLayout'

const Routes = () => {
  return (
    <Router>
      <Set wrap={BlogPostLayout}>
        <Route path="/blog-post/{id:Int}" page={BlogPostPage} name="blogPost" />
        <Route path="/contact" page={ContactPage} name="contact" />
        <Route path="/about" page={AboutPage} name="about" />
        <Route path="/" page={HomePage} name="home" />
      </Set>
      <Private>
        <Route path="/admin/posts/new" page={NewPostPage} name="newPost" />
        <Route path="/admin/posts/{id:Int}/edit" page={EditPostPage} name="editPost" />
        <Route path="/admin/posts/{id:Int}" page={PostPage} name="post" />
        <Route path="/admin/posts" page={PostsPage} name="posts" />
      </Private>
      <Route notfound page={NotFoundPage} />
    </Router>
  )
}

export default Routes
```

If we go back to the `new` route we'll now get an error message.

![16-something-went-wrong](https://dev-to-uploads.s3.amazonaws.com/i/5pcsziq7mn2875k4qam5.jpg)

Instead of just showing an error message, we can redirect the user back to the home page by adding `unauthenticated="home"`.

```jsx
<Private unauthenticated="home">
  <Route
    path="/posts/new"
    page={NewPostPage}
    name="newPost"
  />
  <Route
    path="/posts/{id:Int}/edit"
    page={EditPostPage}
    name="editPost"
  />
  <Route
    path="/posts/{id:Int}"
    page={PostPage}
    name="post"
  />
  <Route
    path="/posts"
    page={PostsPage}
    name="posts"
  />
</Private>
```

Now we'll see a redirect in the URL and we'll be taken back to the home

![17-redirect-to-home-page](https://dev-to-uploads.s3.amazonaws.com/i/1lkg2p4flr14rj7odq7p.jpg)

### `useAuth`

To implement login we'll import `useAuth` from the Redwood `auth` package and pull out the `logIn` object with object destructuring. We'll then add a link to Login and pass the `logIn` object to the `onClick` event handler.

```jsx
import { Link, routes } from '@redwoodjs/router'
import { useAuth } from '@redwoodjs/auth'

const BlogLayout = ({ children }) => {
  const { logIn } = useAuth()

  return (
    <>
      <header>
        <h1>
          <Link to={routes.home()}>ajcwebdev</Link>
        </h1>

        <nav>
          <ul>
            <li>
              <Link to={routes.about()}>
                About
              </Link>
            </li>
            <li>
              <Link to={routes.contact()}>
                Contact
              </Link>
            </li>
            <li>
              <a href="#" onClick={logIn}>
                Login
              </a>
            </li>
          </ul>
        </nav>
      </header>

      <main>{children}</main>
    </>
  )
}

export default BlogLayout
```

If we return to our browser we'll now see a link to log in.

![18-home-page-with-log-in-link](https://dev-to-uploads.s3.amazonaws.com/i/t9bnxguhqw4fynl3bioh.jpg)

## 7.5 Netlify Identity

`@redwoodjs/auth` is a lightweight wrapper around popular SPA authentication libraries. I'm going to use the [Netlify Identity Widget](https://github.com/netlify/netlify-identity-widget), however Redwood can support a wide range of authentication providers including:

- [Auth0](https://github.com/auth0/auth0-spa-js)
- [Azure Active Directory](https://github.com/AzureAD/microsoft-authentication-library-for-js)
- [Netlify GoTrue-JS](https://github.com/netlify/gotrue-js)
- [Magic Links - Magic.js](https://github.com/MagicHQ/magic-js)
- [Ethereum](https://github.com/oneclickdapp/ethereum-auth)
- [Supabase](https://supabase.io/docs/guides/auth)
- [Nhost](https://docs.nhost.io/auth)
- [Firebase's GoogleAuthProvider](https://firebase.google.com/docs/reference/js/firebase.auth.GoogleAuthProvider)

Check out the [Auth Playground](https://github.com/redwoodjs/playground-auth) for working examples of different authentication providers.

### Go to the Identity tab

![06-netlify-identity](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/khqg41evhb9cp80ets1e.png)

### Click Enable Identity to enable identity

![07-netlify-identity-level-0](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/enhjjte4wv16vu0em7ti.png)

Here we can invite users to give them permissions. We are going to lock down our site and only give ourselves permission to see or edit anything.

### Click `Invite user` to invite a user

![08-invite-users](https://dev-to-uploads.s3.amazonaws.com/i/md0mzxm5jw9zrrtak2nu.jpg)

If we click the link we'll get this fancy looking message from Netlify asking for our site's url.

![19-url-of-your-netlify-site](https://dev-to-uploads.s3.amazonaws.com/i/24kk3h89pbsjd4sgc4uf.jpg)

Enter the domain that we created at the beginning of the article.

![20-set-site's-url](https://dev-to-uploads.s3.amazonaws.com/i/3l2vn0vg40x4402vo6lm.jpg)

Now we have our log in forum.

![21-log-in-form](https://dev-to-uploads.s3.amazonaws.com/i/6inq8s5leaez9azgz3fa.jpg)

You should have received an email to accept the invitation from Netlify.

![22-you've-been-invited-to-join](https://dev-to-uploads.s3.amazonaws.com/i/7dee7ylzvx3x8h5lygrl.jpg)

If we follow the link to accept the invite we'll be taken to our live website and there will be an invite token in the URL.

![23-deployed-site-invite-token](https://dev-to-uploads.s3.amazonaws.com/i/4ml9xjya2j25wew3qbkg.jpg)

Grab the `invite_token` starting with the `#` and copy-paste it over to your localhost.

![24-localhost-invite-token](https://dev-to-uploads.s3.amazonaws.com/i/41pihd1ctwrmfspl763w.jpg)

You should now get a form to enter your password and complete your signup.

![25-complete-your-signup](https://dev-to-uploads.s3.amazonaws.com/i/1m84bslmfeyi3wm5tsgm.jpg)

If you did everything correctly then you should see your blog posts again.

![26-protected-posts](https://dev-to-uploads.s3.amazonaws.com/i/cndoe6k1jv2m2s1kfad4.jpg)

Now we want to add a link to our home page that we can use to log in and log out. We'll destructure two addition objects, `isAuthenticated` and `logOut`.

```jsx
const BlogLayout = ({ children }) => {
  const {
    logIn,
    isAuthenticated,
    logOut
  } = useAuth()
```

We'll add another list them that uses a ternary operator to check whether we are authenticated and to display either Log Out or Log In depending on whether we are logged in or not.

```jsx
<li>
  <a href="#" onClick={logIn} >
    { isAuthenticated ? 'Log Out' : 'Log In'}
  </a>
</li>
```

Go back to your browser and since we are logged in you should see a link for Log Out.

![27-home-page-log-out](https://dev-to-uploads.s3.amazonaws.com/i/60ti579d6q9g04n8hj71.jpg)

Now lets also add a ternary operator to the link itself so it knows to log out with we are currently logged in, and log in if we aren't currently logged in.

```jsx
<li>
  <a
    href="#"
    onClick={isAuthenticated ? logOut : logIn}
  >
    { isAuthenticated ? 'Log Out' : 'Log In'}
  </a>
</li>
```

If we click Log Out the page will refresh and the link will change to Log In.

![28-home-page-log-in-link](https://dev-to-uploads.s3.amazonaws.com/i/f6ye0seq296fp5zj0sfr.jpg)

If we click Log In then we see our log in form again.

![29-log-in-form](https://dev-to-uploads.s3.amazonaws.com/i/cew9gsrpyjtoe0q4hamc.jpg)

If we log in then the link will change back to Log Out.

![30-home-page-log-out-link](https://dev-to-uploads.s3.amazonaws.com/i/1uev1vu8zzpvybqdjxds.jpg)

We'll destructure one more object called `currentUser`.

```jsx
const BlogLayout = ({ children }) => {
  const {
    logIn,
    isAuthenticated,
    logOut,
    currentUser
  } = useAuth()
```

We'll check to make sure we're authenticated and if so we'll show the current user's email with `currentUser.email`.

```jsx
{isAuthenticated && <li>Logged in as {currentUser.email}</li>}
```

If we now look back in our browser you should see a message saying you are logged in.

![31-logged-in-as](https://dev-to-uploads.s3.amazonaws.com/i/0ydtnd7zzque5wvn2ryy.jpg)

In the [next part](https://dev.to/ajcwebdev/a-first-look-at-redwood-js-part-8-3pjc) we'll finally deploy our project to the internet with the universal deployment machi.... I mean, Netlify.