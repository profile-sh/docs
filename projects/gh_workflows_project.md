---
layout: page
title: GitHub Workflows
description: GitHub workflows and actions
giscus_comments: true
author: {{site.author}}
date: 18-11-2024
---

Software development involves several steps (installation, development, build, and deploy) that are repeated forever. GitHub workflows are a way to automate the software development cycle. This project is an example focused walk through of  GitHub workflows and actions. Visit [GitHub workflows and actions](https://docs.github.com/en/actions) for further help.

<details open>
## 1. Workflow setup

We need to create a public organization and two repos under it:

### 1.1 Exercise: Create workflow repos

- create a public organization named gh-workflows-project and under the organization:
  - create a public repo named call-reusable-workflows and create the following directories:
    - .github
    - .github/workflows
  - create a public repo named reusable-workflows and repeat create directories as above. Also, in this repo, create an additional directory named .github/composite-actions

> [!IMPORTANT]
> All our workflows must be in the directory .github/workflows, without any subdirectories.

### 1.2 Exercise: Create a workflow

- create our first workflow w00_hello.yml in the repo call-reusable-workflows under .github/workflows
- paste the following content in it and commit:

```yml
name: w00_hello
on:
  workflow_dispatch
jobs:   
  job1:
    name: test job
    runs-on: ubuntu-latest
    steps:
      - name: say hello
        run: echo 'hello'
```

### 1.3 Exercise: Run the workflow and check run log

- click *Actions* tab of the repo
- on left side menu click w1_hello
- click the dropdown *Run workflow* on the right side
- wait for the run to finish
- click on w1_hello (it is next to green cricle with a tick mark) run log
- click the job name to see the run log

If you see a red circle after the workflow run finishes, that means you have errors, clicking on the run log will show the errors.

### 1.4 Exercise: Create a reusable workflow

- create our first reusable workflow rw01_hello.yml in the repo reusable-workflows under .github/workflows
- paste the following content in it and commit:

```yml
name: rw01_hello
on:
  workflow_call
jobs:   
  job1:
    name: test job
    runs-on: ubuntu-latest
    steps:
      - name: say hello
        run: echo 'hello'
```

### Exercise 1.5: Call the reusable workflow

- create a workflow w01_hello.yml in the repo call-reusable-workflows under .github/workflows
- paste the following content in it and commit:

```yml
name: w01 call a reusable workflow 
on:
  workflow_dispatch
jobs:   
  job1:
    name: j1 call a reusable workflow
    uses: gh-workflows-project/reusable-workflows/.github/workflows/rw01_hello.yml@main
```

Follow the same steps as above to run the workflow and look at the run log. 

### 1.6 Exercise: Create a composite action

- create our first *composite action* file a01_hello/action.yml in the repo reusable-workflows under .github/composite-actions
- paste the following content in it and commit:

```yml
name: a01_hello action
description: Just say hello
runs:
  using: "composite"
  steps:
    - name: say hello
      shell: bash # we have to set shell explicitly in a composite action
      run: echo hello
```

> [!IMPORTANT]
> A *composite action* must be in its own directory with the same name as the *action* name. For example, for an action named my_action it looks like my_action/action.yml.

### 1.7 Exercise: Create a workflow to call the composite action

- create a workflow w02_hello.yml in the repo call-reusable-workflows
- paste the following content in it and commit:

```yml
name: w02 call a composite action
on:
  workflow_dispatch
jobs:   
  job1:
    runs-on: ubuntu-latest
    name: j1 call a composite action
    steps:
      - name: call the action
        uses: gh-workflows-project/reusable-workflows/.github/composite-actions/a01_hello@main
```

Follow the same steps as above to run the workflow and look at the run log. 

## Workflows

### variables

Let us learn how to deal with variables in workflows.

</details>


