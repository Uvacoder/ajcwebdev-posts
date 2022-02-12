---
title: semantic github
description: A semantic pull request lets you tell others about changes you've pushed to a branch but in a way that actually means something.
date: 2020-12-12
tags:
  - github
  - git
  - conventional
  - commits
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/ix3i70ri6o47dvuu6xm7.jpg
layout: layouts/post.njk
---

What is a semantic pull request? The first question to ask would be, what is a pull request?
> A pull request lets you tell others about changes you've pushed to a branch in a repository on GitHub.

The second question to ask would be, what makes something semantic?
> Relation to meaning in language or logic.

A semantic pull request lets you tell others about changes you've pushed to a branch but in a way that actually means something. It is also a term for a variety of tools built around GitHub including:

## action-semantic-pull-request

[action-semantic-pull-request](https://github.com/amannn/action-semantic-pull-request) is a [Github Action](https://github.com/features/actions) that ensures that your PR title matches the [Conventional Commits spec](https://www.conventionalcommits.org/).

The Conventional Commits specification is a lightweight convention on top of commit messages. It provides an easy set of rules for creating an explicit commit history; which makes it easier to write automated tools on top of. This convention dovetails with SemVer, by describing the features, fixes, and breaking changes made in commit messages.

## semantic-release

[semantic-release](https://github.com/semantic-release/semantic-release) automates the whole package release workflow including: determining the next version number, generating the release notes and publishing the package.

## semantic-pull-requests

[semantic-pull-requests](https://github.com/zeke/semantic-pull-requests) is a GitHub status check that ensures your pull requests follow the Conventional Commits spec. It is a [Probot](https://probot.github.io/) app on your repos to ensure your pull requests are semantic before you merge them.

Since it can be installed on one or many repositories it's good for use on lots of different repos, or even an entire GitHub organization full of repos. If you want more fine grained control, consider your own custom Actions workflow using a GitHub Action like [action-semantic-pull-request](https://github.com/amannn/action-semantic-pull-request).