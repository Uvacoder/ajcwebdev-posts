---
title: fauna - the data API
description: Fauna is a global serverless database for ubiquitous, low latency access to app data.
date: 2020-06-08
tags:
  - fauna
  - fql
  - graphql
  - database
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/i1rxcx6i54kzayeeu2py.png
layout: layouts/post.njk
---

>*Transactions are hard. Distributed transactions are harder. Distributed transactions over the WAN are final boss hardness.*
>
>***Andy Pavlo*** - *[October 15, 2018](https://twitter.com/andy_pavlo/status/1051974710710407176)*

Fauna is a global serverless database for ubiquitous, low latency access to app data. It prioritizes data correctness and scale. Fauna is heavily integrated with GraphQL making it a natural extension for JAMstack developers wishing to incorporate a database.

The Fauna Query Language provides [many built-in functions](https://docs.fauna.com/fauna/current/api/fql/cheat_sheet) that can be used to query and modify a database. User-defined functions (UDFs) can store and run commonly used Fauna queries.

>*Fauna is a transactional, temporal, geographically distributed, strongly consistent, secure, multi-tenant, QoS-managed operational database. It’s implemented on the JVM for portability, and it’s relational, but not SQL. Instead, it’s queried via type-safe embedded DSLs, like LINQ.*
>
>***Evan Weaver*** - *[Welcome to the Jungle](https://fauna.com/blog/welcome-to-the-jungle) (September 26, 2016)*

## NoSQL

The explosion of data at companies like Google, Facebook, and Amazon in the mid 2000s pushed traditional relational database technology to its breaking limit. To address the difficulties of sharding systems like MySQL and PostgreSQL a new group of databases were created known as NoSQL.

### Chubby

Google’s Bigtable was a distributed storage system implemented on top of the Paxos inspired [Chubby](https://static.googleusercontent.com/media/research.google.com/en//archive/chubby-osdi06.pdf). Before talking about Bigtable it's useful to understand Chubby. The Chubby lock service provided coarse-grained locking as well as reliable (low-volume) storage for a loosely-coupled distributed system.

Its interface is much like a distributed file system with advisory locks with a design emphasis on availability and reliability, instead of high performance.

![Image](https://sedaily-topics.s3.amazonaws.com/topic_images/0_7741808122926126.jpg)

### Bigtable

[Bigtable](https://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf) was a distributed storage system for managing structured data designed to scale to petabytes of data across thousands of commodity servers. [HBase and Hypertable](https://pdfs.semanticscholar.org/38e2/6a2682f14663a54a75c8af9a6cd350d4c1c1.pdf) were created as open source alternatives to Bigtable.

![bigtable-tablet-location-hierarchy](https://dev-to-uploads.s3.amazonaws.com/i/li53eri6fvnivx3bsaoq.jpg)

Bigtable depended on a cluster management system for scheduling jobs, managing resources on shared machines, dealing with machine failures, and monitoring machine status. The Google SSTable file format stored Bigtable data.

![Image](https://sedaily-topics.s3.amazonaws.com/topic_images/0_0070389502551797545.jpg)

An SSTable provided a persistent, ordered immutable map from keys to values, where both keys and values are arbitrary byte strings.

## NoSQL Hangover

In the late 2000s many different experimental systems were created that attempted to meet the scaleability requirements of companies that had jumped ship to NoSQL while retaining the features developers had come to expect from traditional relational database management systems like joins and ACID transactions.

>*The Paxos algorithm, when presented in plain English, is very simple.*
>
>***Leslie Lamport*** - *[Paxos Made Simple (November 1, 2001)](https://www.microsoft.com/en-us/research/uploads/prod/2016/12/paxos-simple-Copy.pdf)*

### MegaStore

In 2011, [Megastore](http://cidrdb.org/cidr2011/Papers/CIDR11_Paper32.pdf) implemented it's own versions of the Paxos algorithm to show how the classic distributed consensus algorithm could be used to achieve consistency at scale.

![megastore](https://dev-to-uploads.s3.amazonaws.com/i/ob6fsbnlfdr9c28j0qxb.jpg)

### Spinnaker

[Spinnaker](https://arxiv.org/pdf/1103.2408.pdf) was designed to run on a large cluster of commodity servers in a single datacenter. Paxos ensured that a data partition in Spinnaker was available for reads and writes as long as a majority of its replicas were alive. It's features included:
* Key-based range partitioning
* 3-way replication
* Transactional get-put API

![spinnaker](https://dev-to-uploads.s3.amazonaws.com/i/c1h8dg6dkvaq8wtde62p.jpg)

## Calvin and Spanner

In 2012, two papers were released that built on the foundational work of Bigtable, Dynamo, Megastore, and Spinnaker by showing how consensus algorithms could be used to build geographically replicated, consistent, ACID compliant, transactional database systems.

[Calvin](http://cs.yale.edu/homes/thomson/publications/calvin-sigmod12.pdf) was published by SIGMOD in May by Daniel Abadi’s Yale team and was followed by Google’s [Spanner](https://static.googleusercontent.com/media/research.google.com/en//archive/spanner-osdi2012.pdf) in October.

![calvin-architecture](https://dev-to-uploads.s3.amazonaws.com/i/ocipwlnklzfws1j6ru7a.jpg)

Spanner ultimately made a bigger splash on the scene at the time but both have proved to be highly influential in the design of databases in the late 2010s including CockroachDB, YugaByte, Vitess, TiDB, and Aurora.

![spanserver-stack](https://dev-to-uploads.s3.amazonaws.com/i/s4wp36mcvu6qs44j605z.jpg)

>*Before [the Google Spanner paper], it was widely believed and also continually marketed by the NoSQL vendors that distributed ACID transactions were literally impossible.*
>
>***Evan Weaver*** - *[Software Daily (March 21, 2019)](https://softwareengineeringdaily.com/wp-content/uploads/2019/03/SED795-FaunaDB.pdf)*

## Raft

Up to this point all of these different systems were using custom implementations of Paxos as their consensus algorithm. Paxos was first described in Leslie Lamport's famous(ly complicated) 1998 paper [The Part-Time Parliament](https://lamport.azurewebsites.net/pubs/lamport-paxos.pdf). It described how a hypothetical parliament made up of voting state machines could use [three phase commits](https://ecommons.cornell.edu/bitstream/handle/1813/6323/82-483.pdf) to achieve global consistency.

If that last sentence doesn’t make any sense to you then you are not alone. 2013 saw the introduction of Raft in a paper titled [In Search of an Understandable Consensus Algorithm](https://raft.github.io/raft.pdf). Its author considered the comprehensibility of the algorithm to be a primary concern.

![raft-replicated-state-machine-architecture](https://dev-to-uploads.s3.amazonaws.com/i/7u44qsni7jf62cnozjnp.jpg)

The consensus algorithm manages a **replicated log** containing **state machine commands** from clients. The state machines process identical sequences of commands from the logs, so they produce the same outputs. It is called Raft because it is a bunch of logs.

Raft aimed to provide the same benefits of Paxos with a simpler implementation that could be more easily adopted by practical systems.

## Fauna Distributed Transaction Protocol

Fauna has taken the insights of Calvin and Raft and combined them with the real world experience of Evan Weaver's time scaling Twitter to create a new base protocol, the [Fauna Distributed Transaction Protocol](https://fauna.com/blog/consistency-without-clocks-faunadb-transaction-protocol). It supports strictly serializable, externally consistent transactions.

In Fauna, data is both partitioned and replicated across machines. Each partition contains multiple records (rows), and each record may have many versions associated with it. Each version is stored separately and is annotated with the transaction identifier that wrote that version.