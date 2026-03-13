---
author: Denney Yang
pubDatetime: 2024-01-03T20:40:08Z
title: How to use Git Hooks to set Created and Modified Dates
featured: false
draft: false
tags:
- docs
- FAQ
description: How to use Git Hooks to set your Created and Modified Dates on AstroPaper
---

In this post I will explain how to use the pre-commit Git hook to automate the input of the created (`pubDatetime`) and modified (`modDatetime`) in the AstroPaper blog theme frontmatter

## Table of contents

## Have them Everywhere

[Git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) are great for automating tasks like [adding](https://gist.github.com/SSmale/3b380e5bbed3233159fb7031451726ea) or [checking](https://itnext.io/using-git-hooks-to-enforce-branch-naming-policy-ffd81fa01e5e) the branch name to your commit messages or [stopping you committing plain text secrets](https://gist.github.com/SSmale/367deee757a9b2e119d241e120249000). Their biggest flaw is that client-side hooks are per machine.

You can get around this by having a `hooks` directory and manually copy them to the `.git/hooks` directory or set up a symlink, but this all requires you to remember to set it up, and that is not something I am good at doing.

As this project uses npm, we are able to make use of a package called [Husky](https://typicode.github.io/husky/) (this is already installed in AstroPaper) to automatically install the hooks for us.

> Update! In AstroPaper [v4.3.0](https://github.com/satnaing/astro-paper/releases/tag/v4.3.0), the pre-commit hook has been removed in favor of GitHub Actions. However, you can easily [install Husky](https://typicode.github.io/husky/get-started.html) yourself.

## The Hook

As we want this hook to run as we commit the code to update the dates and then have that as part of our change we are going to use the `pre-commit` hook. This has already been set up by this AstroPaper project, but if it hadn't, you would run `npx husky add .husky/pre-commit 'echo "This is our new pre-commit hook"'`.

Navigating to the `hooks/pre-commit` file, we are going to add one or both of the following snippets.

### Updating the modified date when a file is edited

