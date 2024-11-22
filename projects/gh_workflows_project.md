---
layout: page
title: GitHub Workflows
description: GitHub workflows and actions
giscus_comments: true
author: {{site.author}}
date: 18-11-2024
---

Software development involves several steps (installation, development, build, and deploy) that are repeated forever. GitHub workflows are a way to automate the software development cycle. This project is an example focused walk through of  GitHub workflows and actions. We will create and use several workflows, reusable-workflows, and composite actions to understand the GitHub workflow syntax. Visit [GitHub workflows and actions](https://docs.github.com/en/actions) for further help. 
  
## 1. Workflow setup

We need a public organization and two repos under it:

### 1.1 Exercise: Create workflow repos

- create a public organization named gh-workflows-project
- under the organization, create a public repo named call-reusable-workflows and create the following directories:
  - .github
  - .github/workflows
- create a public repo named reusable-workflows and repeat create directories as above. Also, in this repo, create an additional directory named .github/composite-actions

> Workflow files must be directly under .github/workflows, not under any subdirectories.

### 1.2 Exercise: Create a workflow

- create our first workflow w00_hello.yml in the repo call-reusable-workflows under .github/workflows
- paste the following content in it and commit:

{% raw %} 
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
{% endraw %}

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

{% raw %} 
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
{% endraw %}

### Exercise 1.5: Call the reusable workflow

- create a workflow w01_hello.yml in the repo call-reusable-workflows under .github/workflows
- paste the following content in it and commit:

{% raw %} 
```yml
name: w01_hello call a reusable workflow 
on:
workflow_dispatch
jobs:   
job1:
  name: j1 call a reusable workflow
  uses: gh-workflows-project/reusable-workflows/.github/workflows/rw01_hello.yml@main
```
{% endraw %}

Follow the same steps as above to run the workflow and look at the run log. 

### 1.6 Exercise: Create a composite action

- create our first *composite action* file a02_hello/action.yml in the repo reusable-workflows under .github/composite-actions
- paste the following content in it and commit:

{% raw %} 
```yml
name: a02_hello action
description: Just say hello
runs:
using: "composite"
steps:
  - name: say hello
    shell: bash # we have to set shell explicitly in a composite action
    run: echo hello
```
{% endraw %}

> A **composite action** must be in its own directory with the same name as the action name. For example, for an action named my_action it looks like my_action/action.yml.
> An action is called from a **step** of a job and cannot have jobs within it, it may have several steps.

### 1.7 Exercise: Create a workflow to call the composite action

- create a workflow w02_hello.yml in the repo call-reusable-workflows
- paste the following content in it and commit:

{% raw %} 
```yml
name: w02_hello call a composite action
on:
workflow_dispatch
jobs:   
job1:
  runs-on: ubuntu-latest
  name: j1 call a composite action
  steps:
    - name: call the action
      uses: gh-workflows-project/reusable-workflows/.github/composite-actions/a02_hello@main
```
{% endraw %}

Follow the same steps as above to run the workflow and look at the run log. 

## 2. *env* context

> The **env** context can be defined at all levels (root, job, or step).
> The **env** context cannot be used under the **id** and **uses** keys, elsewhere it is available.

Exercise:
- create a workflow w03_env.yml in the repo call-reusable-workflows
- paste the following content in it and commit:

{% raw %} 
```yaml
name: w03_env demo using env context
on:
  workflow_dispatch
env:  
  globalvar: globalvar_value
jobs:
  job1:
    name: j1 use env context in a job
    runs-on: ubuntu-latest
    env: 
      jobvar: jobvar_value
    steps:
      - name: update env variables
        env: 
          stepvar1: stepvar1_value
          stepvar2: stepvar2_value
        run: |
          echo 'stepvar1, stepvar2, jobvar, and globalvar all are available in this step'
          echo 'we can access env vars with or without env context prefix:'
          echo $stepvar1 $jobvar $globalvar 
          echo ${{ env.stepvar1 }} ${{ env.jobvar }} ${{ env.globalvar }}
          echo let us update values
          echo pushing the var to the env file GITHUB_ENV changes the scope from step to job level
          echo stepvar2=stepvar2_new_value >> $GITHUB_ENV
          echo jobvar=jobvar_new_value >> $GITHUB_ENV
          echo globalvar=globalvar_new_value >> $GITHUB_ENV 
          echo The updated values will be available in the next step, not this step.
          echo see the updated values are not available in this step:
          echo ${{ env.stepvar1 }} 
          echo ${{ env.stepvar2 }} 
          echo ${{ env.jobvar }} 
          echo ${{ env.globalvar }}            
      - name: test updated env variables 
        run: |
          echo all updated values are available and stepvar1 is not available here
          echo ${{ env.stepvar1 }} 
          echo ${{ env.stepvar2 }} 
          echo ${{ env.jobvar }} 
          echo ${{ env.globalvar }}  
  job2:
    name: j2 show env vars behaviour across jobs
    runs-on: ubuntu-latest
    # To check if the previous job has any effect on env context, we want job1 to finish before we start this job
    needs: job1
    steps:
      - name: test env variables
        run: |
          echo stepvar1, stepvar2, and jobvar are not available acrosss jobs because each job executes with its own env context
          echo global variable is available but the update done in another job will not affect it.
          echo ${{ env.stepvar1 }} 
          echo ${{ env.stepvar2 }} 
          echo ${{ env.jobvar }} 
          echo ${{ env.globalvar }} 
      - name: GITHUB_ENV and github.env are the same
        run: |
          echo GITHUB_ENV and github.env give path of the file that stores env variables, this path is unique for each step
          echo $GITHUB_ENV 
          echo ${{github.env}} 
          echo done with workflow 
```
{% endraw %}

Follow the same steps as above to run the workflow and look at the run log. The usage of env context in an *action* is not different and we do not need a separate example.

## 3. *inputs* and *outputs* contexts

While *env* context is very useful *within* a workflow or action, how do we pass information *between* workflows? We will learn it below.

> The **iputs** context is used to pass user defined variables **from the caller** to the callee workflows and actions.
> The **outputs** context is used to pass information **from the callee** workflow or action to the caller workflow.
> To work with outputs, we have to use **GITHUB_OUPUTS**, a github default variable for the step dependent path to the file that saves outputs.

In the following subsections we learn how to use the inputs and ouputs contexts in a reusable workflow and a composite action.

### 3.1 Reusable workflow

Exercise: Create a reusable workflow:
- create a reusasble workflow rw04_inputs_outputs.yml in the repo reusable-workflows
- paste the following content in it and commit:

{% raw %} 
```yaml
name: rw04_inputs_outputs a reusable workflow with inputs and outputs 
on:
  workflow_call:
    inputs:
      input1:
        required: true  # if 'required' is 'false', set a default value using default: somevalue
        type: string 
      input2:
        required: true  # if 'required' is 'false', set a default value using default: somevalue
        type: string
    outputs:
      an_output:
        description: "An output"
        value: ${{ jobs.set_outputs.outputs.output1 }}
      another_output:
        description: "Another output"
        value: ${{ jobs.set_outputs.outputs.output2 }}
jobs:
  set_outputs:
    name: Generate output
    runs-on: ubuntu-latest
    outputs:
      output1: ${{ steps.step1.outputs.output1 }}
      output2: ${{ steps.step2.outputs.output2 }}
    steps:
      - id: step1
        run: echo "output1=I am an output" >> $GITHUB_OUTPUT
      - id: step2
        run: echo "output2=I am another output" >> $GITHUB_OUTPUT
  print_to_console: # just verify inputs and outputs
   name: print inputs and ouputs
   runs-on: ubuntu-latest
   needs: [set_outputs]
   steps:
    - run: echo '${{ toJSON(inputs) }}'
    - run: echo '${{ toJSON(needs.set_outputs.outputs) }}'
    - run: echo done with reusable workflow
```
{% endraw %}

Now we need to call the above reusable workflow:

Exercise: Create a workflow to pass inputs to the reusable workflow and use outputs from the reusable workflow.
- create a workflow w04_inputs_outputs.yml in the repo call-reuasble-workflows.
- paste the following content in it and commit:

{% raw %} 
```yaml
name: w04_inputs_outputs call a reusable workflow with inputs and outputs 
on:
  workflow_dispatch
jobs:   
  job1: 
    name: j1 call reusable workflow
    uses: gh-workflows-project/reusable-workflows/.github/workflows/rw04_inputs_outputs.yml@main
    with:
     input1: an_input
     input2: another_input
  job2:
    name: j2 use output from j1
    runs-on: ubuntu-latest
    needs: job1 # start only after job1 is done
    steps:
      - name: use output from a completed job
        run: |
          echo ${{ needs.job1.outputs.an_output }} 
          echo ${{ needs.job1.outputs.another_output }}  
  job3:
    name: j3 pass output between steps of a job
    runs-on: ubuntu-latest
    steps:
      - name: step1 create output
        id: step1
        run: echo "output=myoutput" >> $GITHUB_OUTPUT 
      - name: use output from previous step
        run: |
         echo ${{steps.step1.outputs.output}}
         echo workflow done
```
{% endraw %}

Run the workflow and look at the run log. 

### 3.2 Composite action

Exercise: How do we pass information between a caller workflow and an action? Can you create a demo?

Here is a demo:

- create a composite action a05_inputs_outputs in the repo reuasble-workflows
- paste the following content in it and commit:
  
{% raw %} 
```yaml
name:  a05_inputs_outputs using inputs and outputs in a composite action
description: 'use input, give output'
inputs:
  input1:
    required: true  # if 'required' is 'false', set a default value using default: somevalue
    type: string
outputs:
  an_output:
    description: "an output"
    value: ${{ steps.step1.outputs.output1 }}
  another_output:
    description: "another output"
    value: ${{ steps.step1.outputs.output2 }}
runs:
  using: "composite"
  steps:
    - name: set outputs
      shell: bash
      id: step1
      run: |
        echo "output1=i am an out" >> $GITHUB_OUTPUT
        echo "output2=i am another output" >> $GITHUB_OUTPUT
      
    - name: use output from previous step 
      shell: bash
      id: step2
      run: |
        echo ${{ inputs.input1 }}
        echo ${{steps.step1.outputs.output1}}
        echo done with action
```
{% endraw %} 

Exercise: Create a workflow to call the above action.

- create a workflow w05_inputs_outputs.yml in the repo call-reuasble-workflows.
- paste the following content in it and commit:

{% raw %} 
```yaml
name: w05_inputs_outputs call a composite action with inputs and outputs 
on:
  workflow_dispatch
jobs:    
  job1:
    name: j1 pass output between steps of a job using a composite action
    runs-on: ubuntu-latest
    steps:
      - name: step1 call action and create output
        id: step1
        uses: gh-workflows-project/reusable-workflows/.github/composite-actions/a05_inputs_outputs@main
        with:
          input1: an_input
      - name: use output from previous step
        run: |
         echo ${{steps.step1.outputs.an_output}}
         echo ${{steps.step1.outputs.another_output}}
         echo workflow done
```
{% endraw %}

Run the workflow and look at the run log. 

## 4. *secrets* and *vars* contexts

tbc



