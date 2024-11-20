---
layout: page
title: GitHub Workflows
description: GitHub workflows and actions
giscus_comments: true
author: {{site.author}}
date: 18-11-2024
---

Software development involves several steps (installation, development, build, and deploy) that are repeated forever. GitHub workflows are a way to automate the software development cycle. This project is a walk through of  [GitHUb workflows and actions](https://docs.github.com/en/actions) functionality with examples for practice. 

## Workflow setup

We need to create a public organization and two repos under it:

Exercise: workflow setup

- create a public organization named gh-workflows-project and under the organization:
  - create a public repo named call-reusable-workflows and create the following directories:
    - .github
    - .github/workflows
    - .github/actions
  - create a public repo named reusable-workflows and repeat create directories as above.

> [!IMPORTANT]
> All our workflows must be in the directory .github/workflows, without any subdirectories.

Exercise: first workflow

- create our first workflow w1_hello.yml in the repo call-reusable-workflows under .github/workflows
- paste the following content in it and commit:

```yml
name: w1_hello
on:
  workflow_dispatch
jobs:   
  job1:
    steps:
     step1:
     run: echo 'hello'
```

Exercise: run a workflow

- click *Actions* tab of the repo
- on left side menu click w1_hello
- click the dropdown *Run workflow* on the right side
- wait for the run to finish
- refresh your pages site and if everything went well, the site will reload successfully

## Workflows

### variables

Let us learn how to deal with variables in workflows.


