---
title: a first look at postgraphile with railway
description: PostGraphile builds a GraphQL API from a PostgreSQL schema that automatically detects tables, columns, indexes, relationships, views, types, functions, and comments.
date: 2021-07-17
tags:
  - postgraphile
  - postgres
  - graphql
  - railway
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n4pxof36x7fdjvq0fwhd.jpeg
layout: layouts/post.njk
---

PostGraphile builds a GraphQL API from a PostgreSQL schema that automatically detects tables, columns, indexes, relationships, views, types, functions, and comments. It combines PostgreSQL's [role-based grant system](https://www.postgresql.org/docs/current/static/user-manag.html) and [row-level security policies](https://www.postgresql.org/docs/current/static/ddl-rowsecurity.html) with Graphile Engine's [GraphQL look-ahead](https://www.graphile.org/graphile-build/look-ahead/) and [plugin expansion](https://www.graphile.org/graphile-build/plugins/) technologies.

## Outline

* [Provision a PostgreSQL database with Railway](#provision-a-postgresql-database-with-railway)
  * [Railway Dashboard](#railway-dashboard)
  * [Railway CLI](#railway-cli)
  * [Check Railway CLI version](#check-railway-cli-version)
  * [Login with railway login](#login-with-railway-login)
  * [Initialize project with railway init](#initialize-project-with-railway-init)
  * [Provision PostgreSQL with railway add](#provision-postgresql-with-railway-add)
  * [Connect to database](#connect-to-database)
  * [Seed database](#seed-database)
  * [List tables in database](#list-tables-in-database)
  * [Describe table](#describe-table)
  * [Quit psql](#quit-psql)
  * [Copy database connection string to clipboard](#copy-database-connection-string-to-clipboard)
* [Introspect Database with PostGraphile](#introspect-database-with-postgraphile)
  * [Introspect Railway Database](#introspect-railway-database)
  * [Test the endpoint](#test-the-endpoint)
  * [Connect to endpoint with ngrok](#connect-to-endpoint-with-ngrok)

## Provision a PostgreSQL database with Railway

There are two ways to setup a PostgreSQL database with Railway, through the dashboard or through the CLI.

### Railway Dashboard

Click [dev.new](https://dev.new) and choose "Provision PostgreSQL" After the database is setup click "PostgreSQL" on the left and then choose "Connect". Copy and paste the PostgreSQL client command.

### Railway CLI

First you need to install the [Railway CLI](https://docs.railway.app/cli/installation).

### Check Railway CLI version

```bash
railway version
```

```
railway version 0.2.40
```

### Login with railway login

If you do not have a Railway account you will be prompted to create one.

```bash
railway login
```

### Initialize project with railway init

Run the following command, select “Empty Project,” and give your project a name.

```bash
railway init
```

### Provision PostgreSQL with railway add

Run the following command and select PostgreSQL to add a plugin to your Railway project.

```bash
railway add
```

### Connect to database

```bash
railway connect postgresql
```

```
psql (13.3, server 13.2)
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

railway=# 
```

### Seed database

Run the following SQL commands to create a test table with seed data.

```sql
CREATE TABLE Post (title text, body text);
INSERT INTO Post VALUES ('This is a blog post', 'Wooooooo');
INSERT INTO Post VALUES ('Another blog post', 'Even better than the other!');
```

![01-railway-seed-data](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/93pzypyfryw1oqx5rjt7.png)

### List tables in database

```bash
\d
```

```
        List of relations
 Schema | Name | Type  |  Owner   
--------+------+-------+----------
 public | post | table | postgres
(1 row)
```

### Describe table

```bash
\d post
```

```
              Table "public.post"
 Column | Type | Collation | Nullable | Default 
--------+------+-----------+----------+---------
 title  | text |           |          | 
 body   | text |           |          | 
```

### Quit psql

```bash
\q
```

### Copy database connection string to clipboard

```bash
echo `railway variables get DATABASE_URL` | pbcopy
```

## Introspect Database with PostGraphile

It is easy to install PostGraphile with [npm](https://docs.npmjs.com/getting-started/installing-node), although the PostGraphile documentation does not recommend installing PostGraphile globally if you want to use plugins.

```bash
npm install -g postgraphile
```

If you do not globally install you will need to add `npx` the beginning of all `postgraphile` commands in this tutorial.

### Introspect Railway Database

```bash
railway run postgraphile --watch --enhance-graphiql --dynamic-json --port 5001
```

Open `localhost:5001/graphiql` and send the following query.

![02-postgraphile-graphiql](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/k67m0nstevn58dh85liy.png)

### Test the endpoint

```bash
curl \
  --request POST \
    --url "http://localhost:5001/graphql" \
    --header "Content-Type: application/json" \
    --data '{"query":"{ query { allPosts { totalCount nodes { body title } } } }"}'
```

```json
{
  "data":{
    "query":{
      "allPosts":{
        "totalCount":2,
        "nodes":[
          {
            "body":"Wooooooo",
            "title":"This is a blog post"
          },
          {
            "body":"Even better than the other!",
            "title":"Another blog post"
          }
        ]
      }
    }
  }
}
```

### Connect to endpoint with ngrok

[ngrok](https://ngrok.com/) provides an instant, secure URL to your localhost server through any NAT or firewall where you can introspect all HTTP traffic running over your tunnels.

```bash
./ngrok http 5001
```

```
Session Status                online
Account                       Anthony Campolo (Plan: Free)
Version                       2.3.40
Region                        United States (us)
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://363ef1ef5cf3.ngrok.io -> http://localhost:5001
Forwarding                    https://363ef1ef5cf3.ngrok.io -> http://localhost:5001

Connections                   ttl     opn     rt1     rt5     p50     p90
                              2       0       0.00    0.00    5.11    5.21
```

Send the same query with your API tool of choice.

![03-all-posts-query](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xmdadlx9uykt31ie8k4x.png)